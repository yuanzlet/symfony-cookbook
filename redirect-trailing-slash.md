# 使用结尾反斜线重定向 URL 

这个指导书的目的是演示如何将有尾部反斜线的 URL 重定向为相同的不包含尾部反斜线的 URL）。
  
首先创建一个控制器，将匹配任意包含尾部反斜线的 URL，删除尾随的反斜线（如果有查询参数的话请保持），并重定向到一个 301 恢复状态码的新的网址：
```
// src/AppBundle/Controller/RedirectingController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;

class RedirectingController extends Controller
{
    public function removeTrailingSlashAction(Request $request)
    {
        $pathInfo = $request->getPathInfo();
        $requestUri = $request->getRequestUri();

        $url = str_replace($pathInfo, rtrim($pathInfo, ' /'), $requestUri);

        return $this->redirect($url, 301);
    }
}
```

在此之后，创建一个路由到该控制器使任意一个有尾部反斜线的 URL 访问时匹配该路由；一定要把这条路线放在你的系统中，解释如下：

YAML:
```
remove_trailing_slash:
    path: /{url}
    defaults: { _controller: AppBundle:Redirecting:removeTrailingSlash }
    requirements:
        url: .*/$
    methods: [GET]
```

XML:
```

<routes xmlns="http://symfony.com/schema/routing">
    <route id="remove_trailing_slash" path="/{url}" methods="GET">
        <default key="_controller">AppBundle:Redirecting:removeTrailingSlash</default>
        <requirement key="url">.*/$</requirement>
    </route>
</routes>

```

PHP:
```
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;

$collection = new RouteCollection();
$collection->add(
    'remove_trailing_slash',
    new Route(
        '/{url}',
        array(
            '_controller' => 'AppBundle:Redirecting:removeTrailingSlash',
        ),
        array(
            'url' => '.*/$',
        ),
        array(),
        '',
        array(),
        array('GET')
    )
);
```  

> 在旧版浏览器中重定向一个 POST 请求是不能很好地实现的。在重定向后由于一些原因一个 POST 请求的 302 会发送一个 GET请求，因为这个原因，这里的路线只能匹配 GET 请求。  

> 请确保在你的路由配置中路由列表的最末端包含此路由。否则，重定向一个包含有反斜线的路由是危险的（包括Symfony 的核心路由）。  


这项工作的许可为 Creative Commons Attribution-Share Alike 3.0 Unported [License](http://creativecommons.org/licenses/by-sa/3.0/)

