# 如何不用自定义控制器配置重定向

有时，一个 URL 网址需要重定向到另一个 URL 网址。你可以通过创建一个新的唯一任务是重定向的控制器动作来实现它，但使用的 FrameworkBundle 的 RedirectController 会更容易。  

你可以使用网页名称（如 homepage ）来重定向到一个特定的路径（例如  /about ）或一个特定的路由。  

## 使用路径重定向
假定您的应用程序的 / 路径没有默认控制器，并且您希望将这些请求重定向到 /app 。您将需要使用[urlRedirect()](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Controller/RedirectController.html#urlRedirect()) 动作来重定向到这个新的 URL：  


YAML:
```
# app/config/routing.yml

# load some routes - one should ultimately have the path "/app"
AppBundle:
    resource: "@AppBundle/Controller/"
    type:     annotation
    prefix:   /app

# redirecting the root
root:
    path: /
    defaults:
        _controller: FrameworkBundle:Redirect:urlRedirect
        path: /app
        permanent: true
```

XML:
```

<!-- app/config/routing.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        http://symfony.com/schema/routing/routing-1.0.xsd">

    <!-- load some routes - one should ultimately have the path "/app" -->
    <import resource="@AppBundle/Controller/"
        type="annotation"
        prefix="/app"
    />

    <!-- redirecting the root -->
    <route id="root" path="/">
        <default key="_controller">FrameworkBundle:Redirect:urlRedirect</default>
        <default key="path">/app</default>
        <default key="permanent">true</default>
    </route>
</routes>

```

PHP:
```
// app/config/routing.php
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;

$collection = new RouteCollection();

// load some routes - one should ultimately have the path "/app"
$appRoutes = $loader->import("@AppBundle/Controller/", "annotation");
$appRoutes->setPrefix('/app');

$collection->addCollection($appRoutes);

// redirecting the root
$collection->add('root', new Route('/', array(
    '_controller' => 'FrameworkBundle:Redirect:urlRedirect',
    'path'        => '/app',
    'permanent'   => true,
)));

return $collection;

```  

在这个例子中，你为 / 路径设置一个路由，让 RedirectController 将它重定向到 /app 。那么永久性的开关告诉动作发出 301 HTTP 状态代码代替默认的 302 HTTP 状态代码。

## 使用路由重定向
假设你正将你的网站从 WordPress 迁徙到 Symfony，你想重定向 /wp-admin 的路线到sonata_admin_dashboard 的路由，但是你不知道路径，只知道路由名称。那么此时可以使用[redirect()](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Controller/RedirectController.html#redirect()) 动作来实现：  


YAML:
```
# app/config/routing.yml

# ...

# redirecting the admin home
root:
    path: /wp-admin
    defaults:
        _controller: FrameworkBundle:Redirect:redirect
        route: sonata_admin_dashboard
        permanent: true
```

XML:
```
<!-- app/config/routing.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        http://symfony.com/schema/routing/routing-1.0.xsd">

    <!-- ... -->

    <!-- redirecting the admin home -->
    <route id="root" path="/wp-admin">
        <default key="_controller">FrameworkBundle:Redirect:redirect</default>
        <default key="route">sonata_admin_dashboard</default>
        <default key="permanent">true</default>
    </route>
</routes>

```

PHP:
```
// app/config/routing.php
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;

$collection = new RouteCollection();
// ...

// redirecting the root
$collection->add('root', new Route('/wp-admin', array(
    '_controller' => 'FrameworkBundle:Redirect:redirect',
    'route'       => 'sonata_admin_dashboard',
    'permanent'   => true,
)));

return $collection;
```

> 因为你需要重定向到一个路由而不是一个路径，所需的选项被称为重定向的行动路线，而不是在 url 重定向中的行动路径。  

这项工作的许可为 Creative Commons Attribution-Share Alike 3.0 Unported [License](http://creativecommons.org/licenses/by-sa/3.0/)


