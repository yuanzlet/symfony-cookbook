# 如何改变默认的目标路径行为

默认情况下，安全组件将保留在名为 **_security.main.target_path** 的 session 变量中最后访问的 URL 信息(主要是在 **security.yml** 中定义的防火墙的名称)。成功登录后，将用户重定向到此路径，并帮助他们继续停留在访问过的最后一个已知网页。

在某些情况下，这不是理想的。例如，当最后的请求 URL 返回的是非 HTML 或部分 HTML 响应的 XMLHttpRequest 对象，那么用户将被重定向回浏览器无法呈现的网页。

要解决此行为，您仅仅需要去继承 **ExceptionListener** 类，并且重载名为 **setTargetPath()** 的默认方法。

首先，重载您的配置文件中的 **security.exception_listener.class** 参数。这可以在您的主配置文件(在**应用程序下的配置文件**中)或导入包中配置文件中实现:

YAML:

```
# app/config/services.yml
parameters:
    # ...
    security.exception_listener.class: AppBundle\Security\Firewall\ExceptionListener
```

XML:

```
<!-- app/config/services.xml -->
<parameters>
    <!-- ... -->
    <parameter key="security.exception_listener.class">AppBundle\Security\Firewall\ExceptionListener</parameter>
</parameters>
```

PHP:

```
// app/config/services.php
// ...
$container->setParameter('security.exception_listener.class', 'AppBundle\Security\Firewall\ExceptionListener');
```

下一步，创建您自己的 ExceptionListener 监听器：

```
// src/AppBundle/Security/Firewall/ExceptionListener.php
namespace AppBundle\Security\Firewall;

use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Security\Http\Firewall\ExceptionListener as BaseExceptionListener;

class ExceptionListener extends BaseExceptionListener
{
    protected function setTargetPath(Request $request)
    {
        // Do not save target path for XHR requests
        // You can add any more logic here you want
        // Note that non-GET requests are already ignored
        if ($request->isXmlHttpRequest()) {
            return;
        }

        parent::setTargetPath($request);
    }
}
```

为您方案的需要在这里添加或多或少的逻辑。
