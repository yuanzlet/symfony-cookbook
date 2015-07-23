# 如何不用一个自定义的控制器渲染一个模板

通常，当您需要创建一个页面，您需要创建一个控制器并且从该控制器中呈现模板。但如果您仅仅呈现一个简单的模板，并且不需要传递给它的任何数据，则完全没必要创建控一个制器，通过使用内置的 **FrameworkBundle:Template:template** 控制器就可以达到目的。

例如，假设您想要呈现 **static/privacy.html.twig** 模板，并且不需要给它传递任何变量。那么您可以这样做而无需创建一个控制器：

YAML:

```
acme_privacy:
    path: /privacy
    defaults:
        _controller: FrameworkBundle:Template:template
        template:    static/privacy.html.twig
```

XML:

```
<?xml version="1.0" encoding="UTF-8" ?>

<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="acme_privacy" path="/privacy">
        <default key="_controller">FrameworkBundle:Template:template</default>
        <default key="template">static/privacy.html.twig</default>
    </route>
</routes>
```

PHP:

```
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;

$collection = new RouteCollection();
$collection->add('acme_privacy', new Route('/privacy', array(
    '_controller'  => 'FrameworkBundle:Template:template',
    'template'     => 'static/privacy.html.twig',
)));

return $collection;
```

**FrameworkBundle:Template:template** 控制器将简单地呈现您把它当做默认模板传递的任何模板。

当然可以也使用这个技巧把控制器嵌入到模板中来展现这个模板。但由于把控制器嵌入到模板内的目的通常是在自定义的控制器中准备某些数据，这可能只是在您想要缓存这个页面的一部分的时候有用 (请参见[缓存静态模板](http://symfony.com/doc/current/cookbook/templating/render_without_controller.html#cookbook-templating-no-controller-caching))。

Twig：

```
{{ render(url('acme_privacy')) }}
```

PHP:

```
<?php echo $view['actions']->render(
    $view['router']->generate('acme_privacy', array(), true)
) ?>
```

## 缓存的静态模板

因为通常使用这种方法可以实现模板静态化，所以对它们进行缓存会比较有意义。幸运的是，这相对来说比较容易，通过配置您的路径中的几个其他变量，您就可以控制您的页面如何缓存：

YAML:

```
acme_privacy:
    path: /privacy
    defaults:
        _controller:  FrameworkBundle:Template:template
        template:     'static/privacy.html.twig'
        maxAge:       86400
        sharedAge:    86400
```

XML:

```
<?xml version="1.0" encoding="UTF-8" ?>

<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="acme_privacy" path="/privacy">
        <default key="_controller">FrameworkBundle:Template:template</default>
        <default key="template">static/privacy.html.twig</default>
        <default key="maxAge">86400</default>
        <default key="sharedAge">86400</default>
    </route>
</routes>
```

PHP:

```
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;

$collection = new RouteCollection();
$collection->add('acme_privacy', new Route('/privacy', array(
    '_controller'  => 'FrameworkBundle:Template:template',
    'template'     => 'static/privacy.html.twig',
    'maxAge'       => 86400,
    'sharedAge' => 86400,
)));

return $collection;
```

**MaxAge** 和 **sharedAge** 的值用于修改在控制器中创建的响应对象。对缓存的详细信息，请参阅 [HTTP 缓存](http://symfony.com/doc/current/book/http_cache.html)。

这里也有一个**私有**变量 (此处未显示)。在默认情况下，响应将予以公开，只要传递了 **maxAge** 或 **sharedAge** 。如果设置为 **true**，响应将被标记为私有。
