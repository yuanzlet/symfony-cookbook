# 如何定制错误页

在 Symfony 应用程序中，所有的错误都会被认为是异常，不论只是一个 404 Not Found 错误还是你的代码中的一些异常引起的重大错误。  

在[开发环境](http://symfony.com/doc/current/cookbook/configuration/environments.html)中，Symfony 缓存所有的异常并且通过一个拥有很多调试信息的异常页来帮助你发现问题的根源：  

![](http://symfony.com/doc/current/_images/exceptions-in-dev-environment.png)  

由于这个页面包含了很多敏感的内部信息，Symfony 不会将其在产品环境中展示。作为替代，它只会展示一个普通的简单的**错误页**：  

![](http://symfony.com/doc/current/_images/errors-in-prod-environment.png)  

产品环境下产生的错误页可以按照你的要求进行不同方式的定制：  

1. 如果你想要改变你的错误页的内容和风格来适应你的应用程序，那么就[重写默认错误页模板](http://symfony.com/doc/current/cookbook/controller/error_pages.html#use-default-exception-controller)；
2. 如果你想要引入 Symfony 使用的逻辑来产生错误页，那么就[重写默认异常控制器](http://symfony.com/doc/current/cookbook/controller/error_pages.html#custom-exception-controller)；
3. 如果你需要完全控制异常处理执行你自己的逻辑，那么[使用 kernel.exception 事件](http://symfony.com/doc/current/cookbook/controller/error_pages.html#use-kernel-exception-event)。

## 重写默认错误页模板 ##

当加载错误页的时候，内部的 [ExceptionController](http://api.symfony.com/2.7/Symfony/Bundle/TwigBundle/Controller/ExceptionController.html) 被用来产生一个 Twig 模板给用户看。  

这个控制器使用的是 HTTP 状态代码，请求格式和下列逻辑决定了模板文件名：  

1. 找一个给定的格式和状态代码的模板（就像 **error404.json.twig** 或者 **error500.html.twig**）；
2. 如果不存在以前的模板，抛弃状态代码寻找给定格式的一般的模板（就像 **error.json.twig** 或者 **error.xml.twig**）；
3. 如果上述所说的模板都不存在，那就用普通的 HTML 模板（**error.html.twig**）。

为了重写这些模板，简单的依靠标准的 Symfony 的[重写 bundle 内部的模板](http://symfony.com/doc/current/book/templating.html#overriding-bundle-templates)的方法：将他们放到 **app/Resources/TwigBundle/views/Exception/** 目录。  

典型的工程返回的 HTML 和 JSON 页面，可能像下面那样：  

```
app/
└─ Resources/
   └─ TwigBundle/
      └─ views/
         └─ Exception/
            ├─ error404.html.twig
            ├─ error403.html.twig
            ├─ error.html.twig      # All other HTML errors (including 500)
            ├─ error404.json.twig
            ├─ error403.json.twig
            └─ error.json.twig      # All other JSON errors (including 500)
```  

## 404 错误模板的例子 ##

将 404 错误模板重写成 HTML 页，创建一个位于 **app/Resources/TwigBundle/views/Exception/** 的新的 **error404.html.twig** 模板：  

```
{# app/Resources/TwigBundle/views/Exception/error404.html.twig #}
{% extends 'base.html.twig' %}

{% block body %}
    <h1>Page not found</h1>

    {# example security usage, see below #}
    {% if app.user and is_granted('IS_AUTHENTICATED_FULLY') %}
        {# ... #}
    {% endif %}

    <p>
        The requested page couldn't be located. Checkout for any URL
        misspelling or <a href="{{ path('homepage') }}">return to the homepage</a>.
    </p>
{% endblock %}
```  

万一你需要他们，**ExceptionController** 通过分别储存在 HTTP 状态代码和信息中的 **status_code** 和 **status_text** 变量向错误页传递一些信息。  

>你可以通过执行 [HttpExceptionInterface](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/Exception/HttpExceptionInterface.html) 来定制状态代码并且需要 **getStatusCode()** 方法。除此之外，**status_code ** 将会默认成 500.

>在开发环境中展示的异常页可以和错误页一样被自定义。为标准的 HTML 异常页创建一个新的 **exception.html.twig** 模板或者为 JSON 异常页创建一个 **exception.json.twig**。  

### 当在错误模板中使用安全功能时避免异常出现 ###

自定义设计模板时的最常见的一个误区就是在错误模板（或者是其他错误模所继承的模板）中使用 **is_granted()** 功能。如果你那样做了，你将会看到 Symfony 出现异常。  

这个问题的原因就是路由在安全之前完成了。如果 404 错误出现，安全层不能够加载并且因此 **is_granted()** 功能未定义。解决方法就是在使用这个功能之前添加下列的检查：  

```
{% if app.user and is_granted('...') %}
    {# ... #}
{% endif %}
```  

### 在开发环境测试错误页 ###

当你在开发环境的时候，Symfony 将会展示出一个大大的*异常*页而不是你的新的自定义的错误页。所以，你如何看到错误页长什么样子并且调试它呢？  

推荐的解决方法就是使用名为 [WebfactoryExceptionsBundle](https://github.com/webfactory/exceptions-bundle) 的第三方 bundle。这个 bundle 提供了一个特殊的测试控制器可以允许你以任意的 HTTP 状态代码显示自定义的错误页甚至当 **kernel.debug** 设置为 **true** 也可以。  

### 在开发环境测试错误页 ###

默认的 **ExceptionController** 也允许在开发环境下预览你的*错误*页。  

>这个特征是在 Symfony 2.6 中引进的，在以前，第三方的 bundle [WebfactoryExceptionsBundle](https://github.com/webfactory/exceptions-bundle) 可以起到相同的作用。  

使用这一特征，你需要在你的 **routing_dev.yml** 中进行如下定义：  

```YAML
# app/config/routing_dev.yml
_errors:
    resource: "@TwigBundle/Resources/config/routing/errors.xml"
    prefix:   /_error
```  

```XML
<!-- app/config/routing_dev.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        http://symfony.com/schema/routing/routing-1.0.xsd">

    <import resource="@TwigBundle/Resources/config/routing/errors.xml"
        prefix="/_error" />
</routes>
```  

```PHP
// app/config/routing_dev.php
use Symfony\Component\Routing\RouteCollection;

$collection = new RouteCollection();
$collection->addCollection(
    $loader->import('@TwigBundle/Resources/config/routing/errors.xml')
);
$collection->addPrefix("/_error");

return $collection;
```  

如果你是用老版本的 Symfony，你可能需要像你的 **routing_dev.yml** 文件中添加这个。如果你是从 scratch 开始的，那么 [Symfony 标准版本](https://github.com/symfony/symfony-standard/)已经包含这个了。  

添加了这条路线，你可以像以下那样使用网址来用给定的 HTML 状态代码或者给定的状态代码和格式预览*错误*页：  

```
http://localhost/app_dev.php/_error/{statusCode}
http://localhost/app_dev.php/_error/{statusCode}.{format}
```  

## 重写默认的 ExceptionController ##

如果你需要更多的超出仅仅重写模板的灵活性，那么你可以改变生产错误页的控制器。举例来说，你可能需要向你的模板中传递一些附加变量。  

为了完成这个，在你的应用程序的任意的地方创建一个新的控制器并且设置 [twig.exception_controller](http://symfony.com/doc/current/reference/configuration/twig.html#config-twig-exception-controller) 配置选项来指向它：  

```YAML
# app/config/config.yml
twig:
    exception_controller:  AppBundle:Exception:showException
```  

```XML
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:twig="http://symfony.com/schema/dic/twig"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/twig
        http://symfony.com/schema/dic/twig/twig-1.0.xsd">

    <twig:config>
        <twig:exception-controller>AppBundle:Exception:showException</twig:exception-controller>
    </twig:config>
</container>
```  

```PHP
// app/config/config.php
$container->loadFromExtension('twig', array(
    'exception_controller' => 'AppBundle:Exception:showException',
    // ...
));
```  

TwigBundle 使用的 [ExceptionListener](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/EventListener/ExceptionListener.html) 类作为 **kernel.exception** 事件的监听器创建了会发送到你的控制器的请求。除此之外，你的控制器还会被传递两个参数：  

**exception**
由被处理的异常所创建的 [FlattenException](http://api.symfony.com/2.7//Symfony/Component/Debug/Exception/FlattenException.html) 实例。  

**logger**
一个在默写环境下可能为**空**的 [DebugLoggerInterface](http://api.symfony.com/2.7//Symfony/Component/HttpKernel/Log/DebugLoggerInterface.html) 实例。

代替创建你能从 scratch 创建的新的异常控制器，当然，也可以扩展默认的 [ExceptionController](http://api.symfony.com/2.7/Symfony/Bundle/TwigBundle/Controller/ExceptionController.html)。这种情况下，你可能想要重写 showAction() 和 findTemplate() 方法中的一个或者都重写。后一个方法定位了将要使用的模板。  

>[错误页预览](http://symfony.com/doc/current/cookbook/controller/error_pages.html#testing-error-pages)对你自己建立控制器也可以用。  

## 处理 kernel.exception 事件 ##

当出现异常的时候，[HttpKernel](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/HttpKernel.html) 类就会抓住它并且发送到 **kernel.exception** 事件。这给你将异常以不同的方式转换成 **Response** 的权利。  

处理这类事件确实比以前所介绍的更加有力量，而且需要对 Symfony 的内部构件有全面的了解。假设你的代码出现特殊的异常并且对于你的应用程序的域有特殊意义会怎样。  

为 **kernel.exception** 事件编写你自己的监听器使得你可以近距离观察异常并且依据它采取不同的行动。这些行动可能包括记录异常，将用户重新定向到另一个页面或者调用特定的错误页。  

>如果你的监听器在 [GetResponseForExceptionEvent](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/Event/GetResponseForExceptionEvent.html) 调用 **setResponse()**，事件，传播将会被禁止并且将会向客户发出回应。  

这个方法使得你可以创建集中化和层次化的错误处理：而不是一次又一次的在不同的控制器抓取（处理）相同的异常，你只需要一个（或者几个）监听器来处理他们。  

>参见 [ExceptionListener](http://api.symfony.com/2.7/Symfony/Component/Security/Http/Firewall/ExceptionListener.html) 类的代码作为一个先进的这个类型的监听器的真实案例。这个监听器处理各种各样的你的应用程序出现的安全相关的异常（如 [AccessDeniedException](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Exception/AccessDeniedException.html)）并且采取像将用户重新定向到登陆页，登出他们以及其他的方法。  


