# 如何在路由中使用除了 GET 和 POST 的 HTTP 方法

一个请求的 HTTP 方法是可以查看是否匹配路由的请求中的一种。这种方法使在[“ Routing ”](http://symfony.com/doc/current/book/routing.html)这本书中路由那一章节的使用了 GET 和 POST 示例来介绍的。你也可以像这样使用其他 HTTP 动作。例如，如果你有一个博客文章条目，那么你可以使用相同的网址路径通过匹配 GET , PUT 和 DELETE 来显示它，改变它和删除它。

YAML:
```
blog_show:
    path:     /blog/{slug}
    defaults: { _controller: AppBundle:Blog:show }
    methods:  [GET]

blog_update:
    path:     /blog/{slug}
    defaults: { _controller: AppBundle:Blog:update }
    methods:  [PUT]

blog_delete:
    path:     /blog/{slug}
    defaults: { _controller: AppBundle:Blog:delete }
    methods:  [DELETE]
```

XML:
```
<?xml version="1.0" encoding="UTF-8" ?>

<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="blog_show" path="/blog/{slug}" methods="GET">
        <default key="_controller">AppBundle:Blog:show</default>
    </route>

    <route id="blog_update" path="/blog/{slug}" methods="PUT">
        <default key="_controller">AppBundle:Blog:update</default>
    </route>

    <route id="blog_delete" path="/blog/{slug}" methods="DELETE">
        <default key="_controller">AppBundle:Blog:delete</default>
    </route>
</routes>

```

PHP:
```
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;

$collection = new RouteCollection();
$collection->add('blog_show', new Route('/blog/{slug}', array(
    '_controller' => 'AppBundle:Blog:show',
), array(), array(), '', array(), array('GET')));

$collection->add('blog_update', new Route('/blog/{slug}', array(
    '_controller' => 'AppBundle:Blog:update',
), array(), array(), '', array(), array('PUT')));

$collection->add('blog_delete', new Route('/blog/{slug}', array(
    '_controller' => 'AppBundle:Blog:delete',
), array(), array(), '', array('DELETE')));

return $collection;

```

## 使用 _method 伪造方法
> 这里显示的 _method 功能在 Symfony 2.2 中默认是禁用的，在 Symfony 2.3 中默认启用。要在symfony 2.2 中控制它，你必须在你处理请求之前）请求调用 Request::enableHttpMethodParameterOverride（例如，在你前端控制器中） 。在 Symfony2.3 中，使用 http_method_override 选项即可。 
  
不幸的是，生活并非如此简单，因为大多数浏览器不支持通过 HTML 格式中方法发送 PUT 和 DELETE 请求。但幸运的是，Symfony 提供了一个简单的方法来绕过这一限制，通过在查询字符串或 HTTP 请求参数中添加一个 _method 方法参数，Symfony 会利用这个方法来匹配路线。如果它们的提交的不是 GET 或 POST, 那么表单会自动包含此参数的一个隐藏字段。更多信息请查看[the related chapter in the forms documentation](http://symfony.com/doc/current/book/forms.html#book-forms-changing-action-and-method) 。

这项工作的许可为 Creative Commons Attribution-Share Alike 3.0 Unported [License](http://creativecommons.org/licenses/by-sa/3.0/)




