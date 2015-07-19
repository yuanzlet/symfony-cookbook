# 如何使用 Apache Router

> **使用 Apache Router 不再是一个明智之举**。应用程序的路由选择的小小的增加不至于大费周章地升级路由配置。  

> Apache Router 将会在 Symfony 3 中移除，我们强烈建议不要在你的应用程序中应用它。  

Symfony 也快速的提供了各种各样的方法来通过一些微小的调整增加速度。其中的一种方法就是让 Apache 直接处理路由，而不是由 Symfony 直接处理。  

> Apache Router 在 Symfony 2.5 中被弃用并且将在 Symfony 3.0 中移除。因为 Router 的 PHP 安装启用被提升时，而得到的效果不再那么明显（然而很难复制这种行为）。  

## 改变 Router 配置参数

为了转储 Apache route 你必须调整一些配置参数来告诉 Symfony 使用默认的 **ApacheUrlMatcher** 作为替代：  

YAML:

```YAML
# app/config/config_prod.yml
parameters:
    router.options.matcher.cache_class: ~ # disable router cache
    router.options.matcher_class: Symfony\Component\Routing\Matcher\ApacheUrlMatcher
```  

XML:

```XML
<!-- app/config/config_prod.xml -->
<parameters>
    <parameter key="router.options.matcher.cache_class">null</parameter> <!-- disable router cache -->
    <parameter key="router.options.matcher_class">
        Symfony\Component\Routing\Matcher\ApacheUrlMatcher
    </parameter>
</parameters>
```  

PHP:

```PHP
// app/config/config_prod.php
$container->setParameter('router.options.matcher.cache_class', null); // disable router cache
$container->setParameter(
    'router.options.matcher_class',
    'Symfony\Component\Routing\Matcher\ApacheUrlMatcher'
);
```  

> 记得 [ApacheUrlMatcher](http://api.symfony.com/2.7/Symfony/Component/Routing/Matcher/ApacheUrlMatcher.html) 扩展了 [UrlMatcher](http://api.symfony.com/2.7/Symfony/Component/Routing/Matcher/UrlMatcher.html) 所以即使你不更新 mod_rewrite 规则，所有的也能正常工作（因为在 **ApacheUrlMatcher::match()** 末尾 **parent::match()** 的调用已经完成）。  

## 生成 mod_rewrite 规则

为了检验已经生效，为 AppBundle 创建一个基本的路由：  

YAML:

```YAML
# app/config/routing.yml
hello:
    path: /hello/{name}
    defaults: { _controller: AppBundle:Greet:hello }
```  

XML:

```XML
<!-- app/config/routing.xml -->
<route id="hello" path="/hello/{name}">
    <default key="_controller">AppBundle:Greet:hello</default>
</route>
```  

PHP:

```PHP
// app/config/routing.php
$collection->add('hello', new Route('/hello/{name}', array(
    '_controller' => 'AppBundle:Greet:hello',
)));
``` 

现在生成 mod_rewrite 规则：

```
$ php app/console router:dump-apache -e=prod --no-debug
```  

输出应该大致如下所示：  

```
# skip "real" requests
RewriteCond %{REQUEST_FILENAME} -f
RewriteRule .* - [QSA,L]

# hello
RewriteCond %{REQUEST_URI} ^/hello/([^/]+?)$
RewriteRule .* app.php [QSA,L,E=_ROUTING__route:hello,E=_ROUTING_name:%1,E=_ROUTING__controller:AppBundle\:Greet\:hello]
```  

现在你可以使用新规则重写 web/.htaccess，因此这个例子就会变成下面所示的样子：  

```
<IfModule mod_rewrite.c>
    RewriteEngine On

    # skip "real" requests
    RewriteCond %{REQUEST_FILENAME} -f
    RewriteRule .* - [QSA,L]

    # hello
    RewriteCond %{REQUEST_URI} ^/hello/([^/]+?)$
    RewriteRule .* app.php [QSA,L,E=_ROUTING__route:hello,E=_ROUTING_name:%1,E=_ROUTING__controller:AppBundle\:Greet\:hello]
</IfModule>
```  

> 如果你想要充分利用这个设置，上述的过程应当在每次你添加或者改变路由的时候进行。  

就这样！现在你可以准备好使用 Apache routes 了。  

## 附加调整

为了保存一点处理的时间，将 **web/app.php** 中的 **Request** 的出现改变成 **ApacheRequest**：  

```
// web/app.php

require_once __DIR__.'/../app/bootstrap.php.cache';
require_once __DIR__.'/../app/AppKernel.php';
// require_once __DIR__.'/../app/AppCache.php';

use Symfony\Component\HttpFoundation\ApacheRequest;

$kernel = new AppKernel('prod', false);
$kernel->loadClassCache();
// $kernel = new AppCache($kernel);
$kernel->handle(ApacheRequest::createFromGlobals())->send();
``` 