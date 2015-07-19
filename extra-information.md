# 如何从路由向控制器传输额外的信息

默认的采集参数不一定要匹配路由路径中的占位符。事实上，你可以使用默认数组来指定额外的参数，这些变量可以作为参数传递给控制器：  

YAML:

```YAML
# app/config/routing.yml
blog:
    path:      /blog/{page}
    defaults:
        _controller: AppBundle:Blog:index
        page:        1
        title:       "Hello world!"
```

XML:

```XML
<!-- app/config/routing.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        http://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="blog" path="/blog/{page}">
        <default key="_controller">AppBundle:Blog:index</default>
        <default key="page">1</default>
        <default key="title">Hello world!</default>
    </route>
</routes>
```

PHP:

```PHP
// app/config/routing.php
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;

$collection = new RouteCollection();
$collection->add('blog', new Route('/blog/{page}', array(
    '_controller' => 'AppBundle:Blog:index',
    'page'        => 1,
    'title'       => 'Hello world!',
)));

return $collection;
```

现在，你就可以在你的控制中访问这个额外的参数了：

```
public function indexAction($page, $title)
{
    // ...
}
```

正如你所看到的，这个 $title 变量从未在路由路径中定义，但是你仍然可以从你的控制器中获取它的值。