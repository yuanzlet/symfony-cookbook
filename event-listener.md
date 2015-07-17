# 如何创建事件监听器

Symfony 具有很多能触发您应用中的自定义行为的事件和钩子（hooks）。这些事件可以在 [KernelEvents ](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/KernelEvents.html "KernelEvents") 类中查看并且是由 HttpKernel 组件引发。

如果您想要挂钩到事件并添加您自己的自定义逻辑，您必须创建一种在这个事件中作为事件监听者的服务。在此条目中，您将创建一个作为异常监听器的服务，允许您修改您的应用程序异常的显示方式。**KernelEvents::EXCEPTION** 事件是核心内核事件之一:

```
// src/AppBundle/EventListener/AcmeExceptionListener.php
namespace AppBundle\EventListener;

use Symfony\Component\HttpKernel\Event\GetResponseForExceptionEvent;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpKernel\Exception\HttpExceptionInterface;

class AcmeExceptionListener
{
    public function onKernelException(GetResponseForExceptionEvent $event)
    {
        // You get the exception object from the received event
        $exception = $event->getException();
        $message = sprintf(
            'My Error says: %s with code: %s',
            $exception->getMessage(),
            $exception->getCode()
        );

        // Customize your response object to display the exception details
        $response = new Response();
        $response->setContent($message);

        // HttpExceptionInterface is a special type of exception that
        // holds status code and header details
        if ($exception instanceof HttpExceptionInterface) {
            $response->setStatusCode($exception->getStatusCode());
            $response->headers->replace($exception->getHeaders());
        } else {
            $response->setStatusCode(Response::HTTP_INTERNAL_SERVER_ERROR);
        }

        // Send the modified response object to the event
        $event->setResponse($response);
    }
} 
```

> 每个事件接受的 **$event** 对象略有不同。对 **kernel.exception** 事件来说，它是[GetResponseForExceptionEvent ](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/Event/GetResponseForExceptionEvent.html"GetResponseForExceptionEvent")。如果想了解每个事件监听接受的对象是什么类型的，请参阅：[KernelEvents ](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/KernelEvents.html "KernelEvents")。

> 当给 **kernel.request**, **kernel.view** 或者 **kernel.exception** 事件设置响应的时候。事件的传播是停止的，所以优先级较低的监听器并没有被调用。

现在您已经创建了一个类，您现在只需要把它注册为一个服务，并且使用一个特殊的标签告诉 Symfony 这个类是一个 **kernel.exception event** 事件上的监听器：

YAML:

```
# app/config/services.yml
services:
    kernel.listener.your_listener_name:
        class: AppBundle\EventListener\AcmeExceptionListener
        tags:
            - { name: kernel.event_listener, event: kernel.exception, method: onKernelException }
```

XML:

```
<!-- app/config/services.xml -->
<service id="kernel.listener.your_listener_name" class="AppBundle\EventListener\AcmeExceptionListener">
    <tag name="kernel.event_listener" event="kernel.exception" method="onKernelException" />
</service>
```

PHP:

```
// app/config/services.php
$container
    ->register('kernel.listener.your_listener_name', 'AppBundle\EventListener\AcmeExceptionListener')
    ->addTag('kernel.event_listener', array('event' => 'kernel.exception', 'method' => 'onKernelException'))
;
```

> 这里有一个默认值为 0 并且具有可选性的附加优先标签选项。所有监听器都会按照它们的优先级顺序进行执行（从高到低）。当您需要确保您的监听器是按照顺序执行的时候，使用这个标签会非常有用。

## 请求事件，检查类型

一个单页面可以发送很多请求（一个主请求和很多子请求），这也是为什么使用 **KernelEvents::REQUEST** 事件时，您可能需要检查请求的类型。这可以按照如下步骤完成:

```
// src/AppBundle/EventListener/AcmeRequestListener.php
namespace AppBundle\EventListener;

use Symfony\Component\HttpKernel\Event\GetResponseEvent;
use Symfony\Component\HttpKernel\HttpKernel;

class AcmeRequestListener
{
    public function onKernelRequest(GetResponseEvent $event)
    {
        if (!$event->isMasterRequest()) {
            // don't do anything if it's not the master request
            return;
        }

        // ...
    }
}
```

> 两种可用的 [HttpKernelInterface](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/HttpKernelInterface.html "HttpKernelInterface") 接口里的请求分别是：**HttpKernelInterface::MASTER_REQUEST** 和 **HttpKernelInterface::SUB_REQUEST**。

## 调试事件监听器

> **2\.**在 Symfony 2.6 节 中我们讲过 debug:event-dispatcher 命令。

您可以得知哪些监听器通过使用控制台注册到了事件中。如果想要显示所有事件和它们的监听器，您可以通过运行下面的代码来实现：

```
$ php app/console debug:event-dispatcher
```

您可以通过制定某个监听器的名称来获得某个已注册到特定事件的监听器：

```
$ php app/console debug:event-dispatcher kernel.exception
```

