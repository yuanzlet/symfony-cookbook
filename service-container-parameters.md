# 如何在路由中使用服务容器参数

有时你会发现将你的路由的一部分变成全局可配置的是非常有用的。例如，如果你建立一个国际化的网站，你可能会开始在一个或两个地方。你肯定会想你的路由添加一个要求来防止用户匹配到其它地区而不是你支持的区域。  

你可以要求你的 _locale 硬编码在你所有的路径中，但是一个更好的解决方案在你的路由配置中使用一个可配置的服务容器参数：  

YAML:

```YAML
# app/config/routing.yml
contact:
    path:     /{_locale}/contact
    defaults: { _controller: AppBundle:Main:contact }
    requirements:
        _locale: "%app.locales%"
```

XML:

```XML
<!-- app/config/routing.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="contact" path="/{_locale}/contact">
        <default key="_controller">AppBundle:Main:contact</default>
        <requirement key="_locale">%app.locales%</requirement>
    </route>
</routes>
```

PHP:

```PHP
// app/config/routing.php
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;

$collection = new RouteCollection();
$collection->add('contact', new Route('/{_locale}/contact', array(
    '_controller' => 'AppBundle:Main:contact',
), array(
    '_locale' => '%app.locales%',
)));

return $collection;
```

你现在就可以在你的容器的某个地方控制和设置 app.locales 参数了：  

YAML:

```YAML
# app/config/config.yml
parameters:
    app.locales: en|es
```

XML:

```XML
<!-- app/config/config.xml -->
<parameters>
    <parameter key="app.locales">en|es</parameter>
</parameters>

```

PHP:

```PHP
// app/config/config.php
$container->setParameter('app.locales', 'en|es');
```

你还可以使用一个参数来定义你的路由路径（或者你的路径的一部分）： 

YAML:

```YAML
# app/config/routing.yml
some_route:
    path:     /%app.route_prefix%/contact
    defaults: { _controller: AppBundle:Main:contact }
```

XML:

```XML
<!-- app/config/routing.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="some_route" path="/%app.route_prefix%/contact">
        <default key="_controller">AppBundle:Main:contact</default>
    </route>
</routes>
```

PHP:

```PHP
// app/config/routing.php
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;

$collection = new RouteCollection();
$collection->add('some_route', new Route('/%app.route_prefix%/contact', array(
    '_controller' => 'AppBundle:Main:contact',
)));

return $collection;
``` 

> 就像在正常服务容器配置文件中那样，如果在你的路径中真的需要 % ，你可以通过双打来避免百分比的意义，例如 /score-50%%，它会被重处理为 /score-50%。 

然而，包括任何在 URL 的 % 字符会自动生成的 URL 编码，这个例子生成的 URL 会是 /score-50%25（ %25 是编码 % 字符的结果）。

关于在 Dependency Injection Class 下的参数处理请查看 [Using Parameters within a Dependency Injection Class](http://symfony.com/doc/current/cookbook/configuration/using_parameters_in_dic.html)。