# 如何把 Controller 定义为服务

在本书中，你已经学习了当扩展基本的 [Controller](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Controller/Controller.html) 类时如何轻松使用 controller 了。除此之外，controllers 也可以被指定为服务。  

>将 controller 指定为服务将会花费一番功夫。最基本的优点就是整个 controller 或其他传递到 controller 的服务可以通过服务容器配置修正。当开发开放的 bundle 或者在不同的工程中使用的 bundle 时将会很有用。  

>第二个优点就是你的 controller 更加“沙箱化”。通过观察构造器变元，很容易看到 controller 可以做或者不可以做什么。并且因为每个依赖性需要手动注入，很显然（例如你有很多的构造器变元）当你的 controller 变得太大时。[最佳的实践案例](http://symfony.com/doc/current/best_practices/controllers.html)也推荐将 controller 定义为服务：避免将你的商业逻辑放到 
controller 中。相反，注入服务是工作的主要内容。  

>因此，即使你不将你的 controllers 指定为服务，你将会看到这个会在一些开源的 Symfony 中完成。理解两种方法的利弊也很重要。  

## 把 Controller 定义为服务 ##

controller 可以像其他类一样被定义为服务。举例来说，如果你有如下的简单的 controller：  

```
// src/AppBundle/Controller/HelloController.php
namespace AppBundle\Controller;

use Symfony\Component\HttpFoundation\Response;

class HelloController
{
    public function indexAction($name)
    {
        return new Response('<html><body>Hello '.$name.'!</body></html>');
    }
}
```  

接下来你就可以按下列步骤将它定义为服务：  

```YAML
# app/config/services.yml
services:
    app.hello_controller:
        class: AppBundle\Controller\HelloController
```  

```XML
<!-- app/config/services.xml -->
<services>
    <service id="app.hello_controller" class="AppBundle\Controller\HelloController" />
</services>
```  

```PHP
// app/config/services.php
use Symfony\Component\DependencyInjection\Definition;

$container->setDefinition('app.hello_controller', new Definition(
    'AppBundle\Controller\HelloController'
));
```  

## 参照服务 ##

为了提及定义为服务的 controller，可以使用冒号（：）。举例来说，将上述定义的服务的 **indexAction()** 方法使用 **app.hello_controller** id 转发：  

```
$this->forward('app.hello_controller:indexAction', array('name' => $name));
```  

>当使用这个语法是你不能丢弃方法名称的 **Action** 部分。  

当定义路由 **_controller** 值的时候，你也可以通过使用相同的符号将服务路由：  

```YAML
# app/config/routing.yml
hello:
    path:     /hello
    defaults: { _controller: app.hello_controller:indexAction }
```  

```XML
<!-- app/config/routing.xml -->
<route id="hello" path="/hello">
    <default key="_controller">app.hello_controller:indexAction</default>
</route>
```  

```PHP
// app/config/routing.php
$collection->add('hello', new Route('/hello', array(
    '_controller' => 'app.hello_controller:indexAction',
)));
```  

>你也可以使用定义为服务的 controller 符号来配置路由。细节详见 [FrameworkExtraBundle 文档](https://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/annotations/routing.html)。  

>如果 controller 服务调用 **__invoke** 方法，你可以简单的提及服务 id（**app.hello_controller**）。  

## 基本 Controller 方法的替代品 ##

当使用定义为服务的 Controller 时，大多会扩展基本的 **Controller** 类。代替依赖于它的快捷的方法，你将会直接交互你需要的服务。幸运的是，这个通常很简单并且基础的 [Controller 类的源代码](https://github.com/symfony/symfony/blob/master/src/Symfony/Bundle/FrameworkBundle/Controller/Controller.php)是如何执行普通任务的很好的资源。  

举例来说，如果你想要渲染一个模板而不是直接创建 **Response** 对象，之后如果你扩展了 Symfony 的基本 controller 你的代码就会看起来像这样：  

```
// src/AppBundle/Controller/HelloController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class HelloController extends Controller
{
    public function indexAction($name)
    {
        return $this->render(
            'AppBundle:Hello:index.html.twig',
            array('name' => $name)
        );
    }
}

```  

如果你看过 Symfony 的[基本 Controller 类](https://github.com/symfony/symfony/blob/master/src/Symfony/Bundle/FrameworkBundle/Controller/Controller.php)源代码的 **render** 功能，你将会看到这个方法实际上使用了 **templating** 服务：  

```
public function render($view, array $parameters = array(), Response $response = null)
{
    return $this->container->get('templating')->renderResponse($view, $parameters, $response);
}
```  

在定义为服务的 controller 中，你可以注入 **templating** 服务并且直接使用它：  

```
// src/AppBundle/Controller/HelloController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Templating\EngineInterface;
use Symfony\Component\HttpFoundation\Response;

class HelloController
{
    private $templating;

    public function __construct(EngineInterface $templating)
    {
        $this->templating = $templating;
    }

    public function indexAction($name)
    {
        return $this->templating->renderResponse(
            'AppBundle:Hello:index.html.twig',
            array('name' => $name)
        );
    }
}
```  

这个服务定义也需要修正从而区分构造器变元：  

```YAML
# app/config/services.yml
services:
    app.hello_controller:
        class:     AppBundle\Controller\HelloController
        arguments: ["@templating"]
```  

```XML
<!-- app/config/services.xml -->
<services>
    <service id="app.hello_controller" class="AppBundle\Controller\HelloController">
        <argument type="service" id="templating"/>
    </service>
</services>
```  

```PHP
// app/config/services.php
use Symfony\Component\DependencyInjection\Definition;
use Symfony\Component\DependencyInjection\Reference;

$container->setDefinition('app.hello_controller', new Definition(
    'AppBundle\Controller\HelloController',
    array(new Reference('templating'))
));
```  

代替从容器中抓取 **templating** 服务，你可以只向 controller 中直接注入你需要的特定的服务。  

>这并不意味着你不能从你自己的基础 controller扩展这些 controller。从标准的基础 controller 移出去是因为它的帮助方法依靠可用的容器，这个容器不是被定义为服务的 controller 的情形。将注入服务的普通代码提取出来而不是将这个代码放到你扩展的 controller 之中是一个好主意。这两种方法都可行，你想如何组织你的可重复利用代码完全取决于你。  

### 基础 Controller 方法以及他们的服务的替代 ###

这个列表解释了基础 Controller 的方便的方法：  

[createForm()](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Controller/Controller.html#createForm())（服务：**form.factory**）  
```
$formFactory->create($type, $data, $options);
```  

[createFormBuilder()](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Controller/Controller.html#createFormBuilder())（服务：**form.factory**）  
```
$formFactory->createBuilder('form', $data, $options);
```  

[createNotFoundException()](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Controller/Controller.html#createNotFoundException())  
```
new NotFoundHttpException($message, $previous);
```  

[forward()](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Controller/Controller.html#forward())（服务：**http_kernel**）  
```
use Symfony\Component\HttpKernel\HttpKernelInterface;
// ...

$request = ...;
$attributes = array_merge($path, array('_controller' => $controller));
$subRequest = $request->duplicate($query, null, $attributes);
$httpKernel->handle($subRequest, HttpKernelInterface::SUB_REQUEST);
```  

[generateUrl()](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Controller/Controller.html#generateUrl())（服务：**router**）  
```
$router->generate($route, $params, $absolute);
```  

[getDoctrine()](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Controller/Controller.html#getDoctrine())（服务：**doctrine**）  
>*简单的注入 doctrine 而不是从容器中抓取它*  

[getUser()](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Controller/Controller.html#getUser())（服务：**security.token_storage**）  
```
$user = null;
$token = $tokenStorage->getToken();
if (null !== $token && is_object($token->getUser())) {
     $user = $token->getUser();
}
```  

[isGranted()](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Controller/Controller.html#isGranted())（服务：**security.authorization_checker**）  
```
$authChecker->isGranted($attributes, $object);
```  

[redirect()](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Controller/Controller.html#redirect())  
```
use Symfony\Component\HttpFoundation\RedirectResponse;

return new RedirectResponse($url, $status);
```  

[render()](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Controller/Controller.html#render())（服务：**templating**）  
```
$templating->renderResponse($view, $parameters, $response);
```  

[renderView()](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Controller/Controller.html#renderView())（服务：**templating**）  
```
$templating->render($view, $parameters);
```  

[stream()](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Controller/Controller.html#stream())（服务：**templating**）  
```
use Symfony\Component\HttpFoundation\StreamedResponse;

$templating = $this->templating;
$callback = function () use ($templating, $view, $parameters) {
    $templating->stream($view, $parameters);
}

return new StreamedResponse($callback);
```  

>**getRequest** 已经被弃用了。作为替代，你的 controller 有一个处理方法叫做 **Request $request**。参数的排序并不重要，但是必须提供 typehint。  


