# 如何写一个自定义的 Twig 扩展

书写扩展的主要目的就是把经常使用的代码移动到一个可重用的类中，比如说添加国际化支持。扩展可以定义标签、 筛选、 测试、 操作符、 全局变量、 函数和节点访客。

创建扩展也使得在编译执行的时候和代码在运行的时候使您的代码能更好地分离。这样的话，能使您的代码运行得更快。

> 在编写您自己的扩展之前, 请看一看 [Twig 官方的扩展库](https://github.com/twigphp/Twig-extensions)。

## 创建扩展类

> 这本书介绍如何编写到 Twig 1.12 版本以后的自定义的 Twig 扩展。如果您使用的是旧版本，请阅读[以前的 Twig 扩展文件](http://twig.sensiolabs.org/doc/advanced_legacy.html#creating-an-extension)。

如果想要使用您的自定义的功能，首先您必须创建一个 Twig 扩展类。如下面的例子，比如您需要创建一个价格筛选器来把一个给定的数字转换为价格的格式：

```
// src/AppBundle/Twig/AppExtension.php
namespace AppBundle\Twig;

class AppExtension extends \Twig_Extension
{
    public function getFilters()
    {
        return array(
            new \Twig_SimpleFilter('price', array($this, 'priceFilter')),
        );
    }

    public function priceFilter($number, $decimals = 0, $decPoint = '.', $thousandsSep = ',')
    {
        $price = number_format($number, $decimals, $decPoint, $thousandsSep);
        $price = '$'.$price;

        return $price;
    }

    public function getName()
    {
        return 'app_extension';
    }
}
```

> 除了自定义筛选器，你也可以添加自定义函数和注册全局变量。

## 把扩展注册为一种服务

现在，您必须让服务容器知道您新创建了新的 Twig 扩展：

YAML:

```
# app/config/services.yml
services:
    app.twig_extension:
        class: AppBundle\Twig\AppExtension
        public: false
        tags:
            - { name: twig.extension }
```

XML:

```
<!-- app/config/services.xml -->
<services>
    <service id="app.twig_extension"
        class="AppBundle\Twig\AppExtension"
        public="false">
        <tag name="twig.extension" />
    </service>
</services>
```

PHP:

```
// app/config/services.php
use Symfony\Component\DependencyInjection\Definition;

$container
    ->register('app.twig_extension', '\AppBundle\Twig\AppExtension')
    ->setPublic(false)
    ->addTag('twig.extension');
```

> 请记住 Twig 扩展并不是慢慢的加载。这意味有更高的可能性，您将会得到一个 [ServiceCircularReferenceException](http://api.symfony.com/2.7/Symfony/Component/DependencyInjection/Exception/ServiceCircularReferenceException.html)（服务器循环引用异常） 或 [ScopeWideningInjectionException](http://api.symfony.com/2.7/Symfony/Component/DependencyInjection/Exception/ScopeWideningInjectionException.html)（超出作用域注入异常），如果任何服务 (或在这种情况下的 Twig 扩展) 是依赖于请求服务的。想了解更多的信息去看看[如何在作用域中工作](http://symfony.com/doc/current/cookbook/service_container/scopes.html)。

## 使用自定义的扩展

使用您新创建的 Twig 扩展和使用其他扩展没有什么不同：

```
{# outputs $5,500.00 #}
{{ '5500'|price }}
```

将其他参数传递到您的筛选器：

```
{# outputs $5500,2516 #}
{{ '5500.25155'|price(4, ',', '') }}
```

## 进一步的学习

如果想要对 Twig 扩展有更深入的了解，请看看 [Twig 扩展文件](http://twig.sensiolabs.org/doc/advanced.html#creating-an-extension)。
