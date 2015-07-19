# 如何创建一个自定义路由加载器

## 什么是一个自定义路由加载器
自定义路由装载程序使您能够根据某些约定或模式生成路由。在这种情况下，一个很好的例子是在[FOSRestBundle](https://github.com/FriendsOfSymfony/FOSRestBundle) 的使用中路由是基于控制器中的动作方法的名称产生的。  
一个自定义路由加载器不会使你的软件不需要手动修改路由配置（例如 app/config/routing.yml）就注入路线。如果你的包提供的路线，无论是通过一个配置文件，如 WebProfilerBundle那样，或通过自定义路由程序，像 [FOSRestBundle](https://github.com/FriendsOfSymfony/FOSRestBundle)那样，在路由配置项中一个入口都是必须的。  
There are many bundles out there that use their own route loaders to accomplish cases like those described above, for instance FOSRestBundle, JMSI18nRoutingBundle, KnpRadBundle and SonataAdminBundle.

> 现在有许多软件使用自己的路由加载器完成以上描述，例如 instance [FOSRestBundle](https://github.com/FriendsOfSymfony/FOSRestBundle), [JMSI18nRoutingBundle](https://github.com/schmittjoh/JMSI18nRoutingBundle), [KnpRadBundle](https://github.com/KnpLabs/KnpRadBundle) 和 [SonataAdminBundle](https://github.com/sonata-project/SonataAdminBundle)。

## 加载路由 

在 Symfony 应用中的路线是由 DelegatingLoader 加载的。这种加载器采用了一些其他装载机（代表）来加载不同类型的资源，例如 YAML 文件或控制器文件中的 @Route 和 @Method 注释。专业装载机实现了 LoaderInterface 接口，因此有两个重要的方法： supports() 和 load()。  
在 Symfony 的标准版中从 routing.yml 中取出以下几行：  
```
# app/config/routing.yml
app:
    resource: @AppBundle/Controller/
    type:     annotation

```

当主加载器解析时，它会尝试所有注册代表加载器并以给定的资源 (@AppBundle/Controller/) 和类型（注释）作为参数调用它们各自的 support() 方法。只要有一个加载器返回真，其 load（）方法就会被调用，且它会返回一个包含路径对象的 RouteCollection 值。

## 创建一个自定义的加载器
为了从一些自定义源加载一些路由（即从注释，YAML 或 XML 文件以外的资源），您需要创建一个自定义路由加载器。这个加载器必须实现 LoaderInterface 接口。  
在大多数情况下，最好不要去实现自己 LoaderInterface 接口而是从 Loader 中扩展出来。
下面的示例加载器支持一种额外的加载路由资源类型。额外的类型是不重要的-你可以创造你想要的任何资源类型。该示例中的资源名称本身并不是会实际使用：
```
// src/AppBundle/Routing/ExtraLoader.php
namespace AppBundle\Routing;

use Symfony\Component\Config\Loader\Loader;
use Symfony\Component\Routing\Route;
use Symfony\Component\Routing\RouteCollection;

class ExtraLoader extends Loader
{
    private $loaded = false;

    public function load($resource, $type = null)
    {
        if (true === $this->loaded) {
            throw new \RuntimeException('Do not add the "extra" loader twice');
        }

        $routes = new RouteCollection();

        // prepare a new route
        $path = '/extra/{parameter}';
        $defaults = array(
            '_controller' => 'AppBundle:Extra:extra',
        );
        $requirements = array(
            'parameter' => '\d+',
        );
        $route = new Route($path, $defaults, $requirements);

        // add the new route to the route collection
        $routeName = 'extraRoute';
        $routes->add($routeName, $route);

        $this->loaded = true;

        return $routes;
    }

    public function supports($resource, $type = null)
    {
        return 'extra' === $type;
    }
}
```

确保您指定的控制器确实存在。在这种情况下，你要在  AppBundle 的 ExtraController 中创建一个 extraAction 方法：
```
// src/AppBundle/Controller/ExtraController.php
namespace AppBundle\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class ExtraController extends Controller
{
    public function extraAction($parameter)
    {
        return new Response($parameter);
    }
}
``` 

现在我们为ExtraLoader定义一个服务：  
YAML:
```
# app/config/services.yml
services:
    app.routing_loader:
        class: AppBundle\Routing\ExtraLoader
        tags:
            - { name: routing.loader }
```

XML:
```

<?xml version="1.0" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

    <services>
        <service id="app.routing_loader" class="AppBundle\Routing\ExtraLoader">
            <tag name="routing.loader" />
        </service>
    </services>
</container>

```

PHP:
```
use Symfony\Component\DependencyInjection\Definition;

$container
    ->setDefinition(
        'app.routing_loader',
        new Definition('AppBundle\Routing\ExtraLoader')
    )
    ->addTag('routing.loader')
;
``` 


注意 routing.loader 标签。 包含这个标签的所有的服务会被标记为潜在路由加载器并会被作为专业路由加载器被添加到 routing.loader 服务中，这就是一个 [DelegatingLoader ](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Routing/DelegatingLoader.html)实例。  

## 使用自定义加载器
如果你没有做其他的话，你的自定义路由程序将不会被调用。 为使用自定义加载器，你只需要添加一些额外的路由配置：  
YAML:
```
# app/config/routing.yml
app_extra:
    resource: .
    type: extra
```

XML:
```
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

    <import resource="." type="extra" />
</routes>

```

PHP:
```
// app/config/routing.php
use Symfony\Component\Routing\RouteCollection;

$collection = new RouteCollection();
$collection->addCollection($loader->import('.', 'extra'));

return $collection;
```   

这里的重要部分是类型关键字。由于这种类型是 ExtraLoader 所支持的并且需要保证它的 load() 方法被调用所以它的值应该是 “extra” 类型的。对于 ExtraLoader 来说资源关键字相对来讲是微不足道，所以它被设置为“.”。  

> 使用自定义路由加载器定义的路由将自动缓存到该框架中。所以每当你加载器类中改变某些东西时，不要忘记清除缓存。  

## 更多先进的加载器

如果你的自定义路由加载器是如上述所示从 Loader 中扩展起来的，那么你还可以利用所提供的解析器，LoaderResolver 实例来加载第二种路由资源。    

当然你还需要实现 supports() 和 load()。每当你想加载另一个资源时，例如一个 YAML 的路由配置文件，你可以调用 import() 方法如下：

```   
// src/AppBundle/Routing/AdvancedLoader.php
namespace AppBundle\Routing;

use Symfony\Component\Config\Loader\Loader;
use Symfony\Component\Routing\RouteCollection;

class AdvancedLoader extends Loader
{
    public function load($resource, $type = null)
    {
        $collection = new RouteCollection();

        $resource = '@AppBundle/Resources/config/import_routing.yml';
        $type = 'yaml';

        $importedRoutes = $this->import($resource, $type);

        $collection->addCollection($importedRoutes);

        return $collection;
    }

    public function supports($resource, $type = null)
    {
        return 'advanced_extra' === $type;
    }
}
```   

> 资源名称和进口路由配置类型可以是任意设置，通常由路由配置加载器支持（YAML，XML，PHP，注释等）。

这项工作的许可为 Creative Commons Attribution-Share Alike 3.0 Unported [License](http://creativecommons.org/licenses/by-sa/3.0/)


