# 如何使用匹配器有条件地启用分析器

默认情况下，分析器只在开发环境中被启用。但可以想象的是作为一个开发商想要看到分析器即使在是产品中。另一种情况可能是，仅当一个管理员登录时，您想要启用分析器。你可以通过使用匹配器在这些情况下实现分析器的启用。  

## 使用内置的匹配  
symfony 提供了一个内置的匹配器用来匹配路径和 IPS 。例如，如果你需要仅当 168.0.0.1 IP 访问网页时显示分析器，那么你可以使用这个配置：   
YAML 
```  
# app/config/config.yml
framework:
    # ...
    profiler:
        matcher:
            ip: 168.0.0.1
``` 
XML:
```
<!-- app/config/config.xml -->
<framework:config>
    <framework:profiler
        ip="168.0.0.1"
    />
</framework:config>
```

PHP:
```
// app/config/config.php
$container->loadFromExtension('framework', array(
    'profiler' => array(
        'ip' => '168.0.0.1',
    ),
));  

```

还可以设置路径选项来定义应启用分析器的路径。 例如，设置它为 ^/admin/ 将使仅为 ^/admin/ 网址来启用分析器。  

## 创建一个自定义匹配器
你也可以创建一个自定义的匹配器。这是一个检查是否启用或不启用分析器的服务。创建这个服务，创建一个类实现 requestmatcherinterface 接口。这个接口需要一个方法：matches()。 此方法返回假以禁用事件分析器和真来启用分析器。  

```
// src/AppBundle/Profiler/SuperAdminMatcher.php
namespace AppBundle\Profiler;

use Symfony\Component\Security\Core\Authorization\AuthorizationCheckerInterface;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\RequestMatcherInterface;

class SuperAdminMatcher implements RequestMatcherInterface
{
    protected $authorizationChecker;

    public function __construct(AuthorizationCheckerInterface $authorizationChecker)
    {
        $this->authorizationChecker = $authorizationChecker;
    }

    public function matches(Request $request)
    {
        return $this->authorizationChecker->isGranted('ROLE_SUPER_ADMIN');
    }
}
```  

 > 在 Symfony 2.6 中介绍了 [AuthorizationCheckerInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Authorization/AuthorizationCheckerInterface.html) 。之前，你必须使用[ SecurityContextInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Core/SecurityContextInterface.html) 中的 isGranted 方法。

然后，您需要配置服务：  
YAML 
```  
# app/config/services.yml
services:
    app.profiler.matcher.super_admin:
        class: AppBundle\Profiler\SuperAdminMatcher
        arguments: ["@security.authorization_checker"]
``` 
XML:
```
<!-- app/config/services.xml -->
<services>
    <service id="app.profiler.matcher.super_admin"
        class="AppBundle\Profiler\SuperAdminMatcher">
        <argument type="service" id="security.authorization_checker" />
</services>
```

PHP:
```
// app/config/services.php
use Symfony\Component\DependencyInjection\Definition;
use Symfony\Component\DependencyInjection\Reference;

$container->setDefinition('app.profiler.matcher.super_admin', new Definition(
    'AppBundle\Profiler\SuperAdminMatcher',
    array(new Reference('security.authorization_checker'))
);

```  


> Symfony 2.6 中介绍了安全认证检查服务（security.authorization_checker service） ，你必须使用 security.context 服务中的 isGranted() 方法。  

现在服务已经被注册，唯一剩下要做的就是配置分析器使用该服务作为匹配器：

YAML 
```  
# app/config/config.yml
framework:
    # ...
    profiler:
        matcher:
            service: app.profiler.matcher.super_admin
``` 
XML:
```
<!-- app/config/config.xml -->
<framework:config>
    <!-- ... -->
    <framework:profiler
        service="app.profiler.matcher.super_admin"
    />
</framework:config>
```

PHP:
```
// app/config/config.php
$container->loadFromExtension('framework', array(
    // ...
    'profiler' => array(
        'service' => 'app.profiler.matcher.super_admin',
    ),
));

```  


这项工作的许可为 Creative Commons Attribution-Share Alike 3.0 Unported [License](http://creativecommons.org/licenses/by-sa/3.0/)。 
