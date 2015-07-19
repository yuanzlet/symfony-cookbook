# 如何在路由参数中允许&quot;/&quot;字符

有时，您需要包含一个 / 参数来组成 URL 。例如，采取经典的 /hello/{username} 路径。默认情况下，/hello/Fabien 将匹配该条路径而不是 /hello/Fabien/Kris 。这是因为 Symfony 使用这个字符作为路径各部分之间的分隔符。 

本指南包括如何修改路径使 /hello/Fabien/Kris 匹配 /hello/{username}，且此时 {username} 等于 Fabien/Kris。  

## 配置路径

默认情况下，Symfony 的路由组件要求的参数匹配下面的正则表达式的路径：[ / ] + ^。这意味着除了 / 外所有的字符都是被允许的。   

你必须通过指定一个更宽松的正则路径明确地允许/ 成为你参数的一部分。
  
Annotations：

```Annotations
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

class DemoController
{
    /**
     * @Route("/hello/{username}", name="_hello", requirements={"username"=".+"})
     */
    public function helloAction($username)
    {
        // ...
    }
}
```

YAML:

```YAML
_hello:
    path:     /hello/{username}
    defaults: { _controller: AppBundle:Demo:hello }
    requirements:
        username: .+
```

XML:

```XML
<?xml version="1.0" encoding="UTF-8" ?>

<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="_hello" path="/hello/{username}">
        <default key="_controller">AppBundle:Demo:hello</default>
        <requirement key="username">.+</requirement>
    </route>
</routes>
```

PHP:

```PHP
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;

$collection = new RouteCollection();
$collection->add('_hello', new Route('/hello/{username}', array(
    '_controller' => 'AppBundle:Demo:hello',
), array(
    'username' => '.+',
)));

return $collection;
```
  
就是这样！现在，{username} 参数就可以包含 / 字符了。