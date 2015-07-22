# 如何在过滤器的前后设置事件分发器

在 web 开发中很常见，在您的控制器动作之前或之后，需要执行一些逻辑作为过滤器或挂钩。

在 symfony1 中，由 preExecute 和 postExecute 方法达到。大部分主要的框架有相似的方法，但是在 Symfony 缺没有这样的事。好消息是有一个更好的方法来妨碍在 Request -> Response 进程中使用 [EventDispatcher 组件](http://symfony.com/doc/current/components/event_dispatcher/introduction.html)。

## 标识验证示例

想象您需要开发一个 API，其中一些控制器是公开的但是其他的一些别限制在一个或一些客户上。对于这些私人的功能，您可以为客户提供一个标识来进行自我验证。

所以，在执行您的控制器动作之前，您需要检查动作是否限制。如果是限制的话，您需要验证所提供的标识。

请记住本教程为了简洁，标识将被在配置中定义，并且不管是数据库设置还是认证，没有通过安全组件的都不会被使用。

## kernel.controller 事件隔离器之前

首先，使用 **config.yml** 存储一些基本的标识配置，还有参数秘钥：

YAML

```
# app/config/config.yml
parameters:
    tokens:
        client1: pass1
        client2: pass2
```

XML

```
<!-- app/config/config.xml -->
<parameters>
    <parameter key="tokens" type="collection">
        <parameter key="client1">pass1</parameter>
        <parameter key="client2">pass2</parameter>
    </parameter>
</parameters>
```

PHP

```
// app/config/config.php
$container->setParameter('tokens', array(
    'client1' => 'pass1',
    'client2' => 'pass2',
));
```

## 标记要检查的控制器

**kernel.controller** 监听器会收到*每个*请求的通知，恰好在控制器执行之前。所以首先，您需要一些办法来验证匹配请求的控制器是否需要标识验证。

一个简洁容易的方法是创建一个空的接口，使控制器实现它：

```
namespace AppBundle\Controller;

interface TokenAuthenticatedController
{
    // ...
}
```

实现此接口的控制器看起来像这样：

```
namespace AppBundle\Controller;

use AppBundle\Controller\TokenAuthenticatedController;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class FooController extends Controller implements TokenAuthenticatedController
{
    // An action that needs authentication
    public function barAction()
    {
        // ...
    }
}
```

## 创建一个事件监听器

接下来，您需要创建一个事件监听器，保存在您控制器之前执行的逻辑。如果您对事件监听器不熟悉的话，您可以在[如何创建一个事件监听器](http://symfony.com/doc/current/cookbook/service_container/event_listener.html)学到更多。

```
// src/AppBundle/EventListener/TokenListener.php
namespace AppBundle\EventListener;

use AppBundle\Controller\TokenAuthenticatedController;
use Symfony\Component\HttpKernel\Exception\AccessDeniedHttpException;
use Symfony\Component\HttpKernel\Event\FilterControllerEvent;

class TokenListener
{
    private $tokens;

    public function __construct($tokens)
    {
        $this->tokens = $tokens;
    }

    public function onKernelController(FilterControllerEvent $event)
    {
        $controller = $event->getController();

        /*
         * $controller passed can be either a class or a Closure.
         * This is not usual in Symfony but it may happen.
         * If it is a class, it comes in array format
         */
        if (!is_array($controller)) {
            return;
        }

        if ($controller[0] instanceof TokenAuthenticatedController) {
            $token = $event->getRequest()->query->get('token');
            if (!in_array($token, $this->tokens)) {
                throw new AccessDeniedHttpException('This action needs a valid token!');
            }
        }
    }
}
```

## 注册监听器

最后，作为服务器注册您的监听器，并且标记为事件监听器。通过在 **kernel.controller** 上监听，您就可以告知 Symfony 您想要在任何控制器执行前调用监听器。

YAML

```
# app/config/services.yml
services:
    app.tokens.action_listener:
        class: AppBundle\EventListener\TokenListener
        arguments: ["%tokens%"]
        tags:
            - { name: kernel.event_listener, event: kernel.controller, method: onKernelController }
```

XML

```
<!-- app/config/services.xml -->
<service id="app.tokens.action_listener" class="AppBundle\EventListener\TokenListener">
    <argument>%tokens%</argument>
    <tag name="kernel.event_listener" event="kernel.controller" method="onKernelController" />
</service>
```

PHP

```
// app/config/services.php
use Symfony\Component\DependencyInjection\Definition;

$listener = new Definition('AppBundle\EventListener\TokenListener', array('%tokens%'));
$listener->addTag('kernel.event_listener', array(
    'event'  => 'kernel.controller',
    'method' => 'onKernelController'
));
$container->setDefinition('app.tokens.action_listener', $listener);
```

有了这个配置，您的 **TokenListener onKernelController** 方法将会在每一个请求都执行。如果将要执行的控制器实现了 **TokenAuthenticatedController**，应用过标识验证。这让您在想要的任何控制器上有一个“前”过滤器。

## kernel.response 事件隔离器之后

除了在您的控制器*之前*有一个执行的“挂钩”，您也可以添加一个挂钩在您的控制器*之后*执行。这个例子，想象您想添加一个 sha1 散列（使用那个标识的盐）到所有已通此标识验证的响应。

另一个核心 Symfony 事件—叫做 **kernel.response**— 在每一项请求都提醒，但是实在控制器返回到响应对象之后。创建一个“后”监听器如同创建一个监听器类一样简单，并且在此事件上注册。

例如，从之前的例子中拿出 **TokenListener**，首先在请求属性中记录验证标识。这将作为一个基本标志，即此请求进行了标识验证。

```
public function onKernelController(FilterControllerEvent $event)
{
    // ...

    if ($controller[0] instanceof TokenAuthenticatedController) {
        $token = $event->getRequest()->query->get('token');
        if (!in_array($token, $this->tokens)) {
            throw new AccessDeniedHttpException('This action needs a valid token!');
        }

        // mark the request as having passed token authentication
        $event->getRequest()->attributes->set('auth_token', $token);
    }
}
```

现在，在此类添加另一个方法—**onKernelResponse**—寻找在此请求对象上的标志并如果找到的话，就在响应上设置一个自定义标题：

```
// add the new use statement at the top of your file
use Symfony\Component\HttpKernel\Event\FilterResponseEvent;

public function onKernelResponse(FilterResponseEvent $event)
{
    // check to see if onKernelController marked this as a token "auth'ed" request
    if (!$token = $event->getRequest()->attributes->get('auth_token')) {
        return;
    }

    $response = $event->getResponse();

    // create a hash and set it as a response header
    $hash = sha1($response->getContent().$token);
    $response->headers->set('X-CONTENT-HASH', $hash);
}
```

最终，第二个“标签”也需要在服务定义中来通知 Symfony **onKernelResponse** 事件应该为 **kernel.response** 通知：

YAML

```
# app/config/services.yml
services:
    app.tokens.action_listener:
        class: AppBundle\EventListener\TokenListener
        arguments: ["%tokens%"]
        tags:
            - { name: kernel.event_listener, event: kernel.controller, method: onKernelController }
            - { name: kernel.event_listener, event: kernel.response, method: onKernelResponse }
```

XML

```
<!-- app/config/services.xml -->
<service id="app.tokens.action_listener" class="AppBundle\EventListener\TokenListener">
    <argument>%tokens%</argument>
    <tag name="kernel.event_listener" event="kernel.controller" method="onKernelController" />
    <tag name="kernel.event_listener" event="kernel.response" method="onKernelResponse" />
</service>
```

PHP

```
// app/config/services.php
use Symfony\Component\DependencyInjection\Definition;

$listener = new Definition('AppBundle\EventListener\TokenListener', array('%tokens%'));
$listener->addTag('kernel.event_listener', array(
    'event'  => 'kernel.controller',
    'method' => 'onKernelController'
));
$listener->addTag('kernel.event_listener', array(
    'event'  => 'kernel.response',
    'method' => 'onKernelResponse'
));
$container->setDefinition('app.tokens.action_listener', $listener);
```

就这样！**TokenListener** 现在在每一个控件执行前都被通知（**onKernelController**）并且在每个控件之后返回一个响应（**onKernelResponse**）。通过让具体的控件实现 **TokenAuthenticatedController** 接口，您的监听器知道哪个控件是该采取行动的。并且通过在请求“属性”包里存储一个值，**onKernelResponse** 方法知道添加额外的标题。

玩得开心！





