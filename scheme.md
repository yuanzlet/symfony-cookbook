# 如何强制路由总是使用 HTTPS 或者 HTTP

有时，你想保护一些路径，确保它们总是通过 HTTPS 协议访问。路由组件允许你有计划的执行 URI 模式： 

YAML:

```YAML
secure:
    path:     /secure
    defaults: { _controller: AppBundle:Main:secure }
    schemes:  [https]
```

XML:

```XML
<?xml version="1.0" encoding="UTF-8" ?>

<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="secure" path="/secure" schemes="https">
        <default key="_controller">AppBundle:Main:secure</default>
    </route>
</routes>
```

PHP:

```PHP
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;

$collection = new RouteCollection();
$collection->add('secure', new Route('/secure', array(
    '_controller' => 'AppBundle:Main:secure',
), array(), array(), '', array('https')));

return $collection;
```  

上述的配置使得安全路由总是使用 HTTPS 访问。  

当生成安全网址时，如果目前的方案是 HTTP ，Symfony 会自动生成一个使用 HTTPS 的绝对 URL 的格式：

``` 
{# If the current scheme is HTTPS #}
{{ path('secure') }}
{# generates /secure #}

{# If the current scheme is HTTP #}
{{ path('secure') }}
{# generates https://example.com/secure #}
```   

我们对输入的请求也执行同样的要求。当使用 HTTPS 方案的时候，如果你试图用 HTTP 访问 /secure 的路径，你会被自动重定向到同一个 URL。  

上面的例子是强制使用 HTTPS 的方案，但你也可以强制 URL 总是使用 HTTP。

> 安全组件提供了另一种方式设置通过 requires_channel 设置项来执行 HTTP 或 HTTPS 。这种替代方法是更适合用于你的网站的某一区域（ 所有在 /admin 目录下的 URL ）或当你想保护定义在第三方包中的 URL 时（请在[如何强制对不同 URL 执行 HTTPS 或 HTTP](http://symfony.com/doc/current/cookbook/security/force_https.html) 中查看更多的细节）。