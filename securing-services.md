# 如何在应用中保护服务和方法

在安全性一章中，您可以看到如何通过从服务容器请求 **security.authorization_checker** 并检查当前用户的角色来[保护一个控制器](http://symfony.com/doc/current/book/security.html#book-security-securing-controller)：

```
// ...http://symfony.com/doc/current/book/security.html#book-security-securing-controller
use Symfony\Component\Security\Core\Exception\AccessDeniedException;

public function helloAction($name)
{
    $this->denyAccessUnlessGranted('ROLE_ADMIN');

    // ...
}
```

您也可以通过向服务注入 **security.authorization_checker** 服务来保护。关于如何向服务注入依赖项的内容介绍请参阅本书的[服务容器](http://symfony.com/doc/current/book/service_container.html)的章节。假如您现在有一个可以发送电子邮件的 **NewsletterManager** 类 ，但是您想把它的使用权限制为具有 ROLE_NEWSLETTER_ADMIN 角色的用户，在您添加安全性之前，这个类应该是下述代码描述的这样：

```
// src/AppBundle/Newsletter/NewsletterManager.php
namespace AppBundle\Newsletter;

class NewsletterManager
{
    public function sendNewsletter()
    {
        // ... where you actually do the work
    }

    // ...
}
```

您的目标是要通过调用 **sendNewsletter()** 方法来检查用户的角色。这第一步是向对象注入  **security.authorization_checker** 服务。因为如果不进行安全性检查将会失去意义，同时这也是进行构造函数注入的一个不错的方法，构造函数注入保证了授权检查对象在 **NewsletterManager** 类中可用:

```
// src/AppBundle/Newsletter/NewsletterManager.php

// ...
use Symfony\Component\Security\Core\Authorization\AuthorizationCheckerInterface;

class NewsletterManager
{
    protected $authorizationChecker;

    public function __construct(AuthorizationCheckerInterface $authorizationChecker)
    {
        $this->authorizationChecker = $authorizationChecker;
    }

    // ...
}
```

然后在您的服务配置中，您可以注入服务:

YAML:

```
# app/config/services.yml
services:
    newsletter_manager:
        class:     "AppBundle\Newsletter\NewsletterManager"
        arguments: ["@security.authorization_checker"]
```

XML:

```
<!-- app/config/services.xml -->
<services>
    <service id="newsletter_manager" class="AppBundle\Newsletter\NewsletterManager">
        <argument type="service" id="security.authorization_checker"/>
    </service>
</services>
```

PHP:

```
// app/config/services.php
use Symfony\Component\DependencyInjection\Definition;
use Symfony\Component\DependencyInjection\Reference;

$container->setDefinition('newsletter_manager', new Definition(
    'AppBundle\Newsletter\NewsletterManager',
    array(new Reference('security.authorization_checker'))
));
```

随后可以调用 **sendNewsletter()**  方法来对注入服务进行安全性检查：

```
namespace AppBundle\Newsletter;

use Symfony\Component\Security\Core\Authorization\AuthorizationCheckerInterface;
use Symfony\Component\Security\Core\Exception\AccessDeniedException;
// ...

class NewsletterManager
{
    protected $authorizationChecker;

    public function __construct(AuthorizationCheckerInterface $authorizationChecker)
    {
        $this->authorizationChecker = $authorizationChecker;
    }

    public function sendNewsletter()
    {
        if (false === $this->authorizationChecker->isGranted('ROLE_NEWSLETTER_ADMIN')) {
            throw new AccessDeniedException();
        }

        // ...
    }

    // ...
}
```

## 使用注释保护方法

您也可以通过使用可选的 [JMSSecurityExtraBundle](https://github.com/schmittjoh/JMSSecurityExtraBundle) 包来保护带有注释的保护方法的调用。虽然在 Symfony 标准版本中不包括此包，但您可以去安装它。

如果想要启用注释功能，可以使用 security.secure_service [标记](http://symfony.com/doc/current/book/service_container.html#book-service-container-tags)来标记您想保护的服务（您还可以为所有服务自动启用此功能请参阅下面的[文本框]( http://symfony.com/doc/current/cookbook/security/securing_services.html#securing-services-annotations-sidebar)）:

YAML:

```
# app/services.yml

# ...
services:
    newsletter_manager:
        # ...
        tags:
            -  { name: security.secure_service }
```

XML:

```
<!-- app/services.xml -->
<!-- ... -->

<services>
    <service id="newsletter_manager" class="AppBundle\Newsletter\NewsletterManager">
        <!-- ... -->
        <tag name="security.secure_service" />
    </service>
</services>
```

PHP:

```
// app/services.php
use Symfony\Component\DependencyInjection\Definition;
use Symfony\Component\DependencyInjection\Reference;

$definition = new Definition(
    'AppBundle\Newsletter\NewsletterManager',
    // ...
));
$definition->addTag('security.secure_service');
$container->setDefinition('newsletter_manager', $definition);
```

您可以通过使用注释来取得和上述程序一样的效果：

```
namespace AppBundle\Newsletter;

use JMS\SecurityExtraBundle\Annotation\Secure;
// ...

class NewsletterManager
{

    /**
     * @Secure(roles="ROLE_NEWSLETTER_ADMIN")
     */
    public function sendNewsletter()
    {
        // ...
    }

    // ...
}
```

> 注释之所以可以有相同的效果是因为在程序中有一个代理类来帮您的类执行安全性检查。这也就是说，您只可以在公有和受保护的方法中使用注释，而不能在私有或者在带有 final 标记的方法中使用它们。

JMSSecurityExtraBundle 还允许您保护方法的参数和返回的值。更多的信息，请参阅 [JMSSecurityExtraBundle](https://github.com/schmittjoh/JMSSecurityExtraBundle) 文档。

## 为所有服务激活注释功能

当您在保护一个服务的方法时（如上面所述），您可以单独的标记每个服务，也可以一次性激活所有的服务的功能。如果想要这样做，可以把 **secure_all_services** 配置将选项设置为 true:

YAML:

```
# app/config/config.yml
jms_security_extra:
    # ...
    secure_all_services: true
```

XML:

```
<!-- app/config/config.xml -->
<?xml version="1.0" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jms-security-extra="http://example.org/schema/dic/jms_security_extra"
    xsi:schemaLocation="http://www.example.com/symfony/schema/ http://www.example.com/symfony/schema/hello-1.0.xsd">

    <!-- ... -->
    <jms-security-extra:config secure-controllers="true" secure-all-services="true" />

</srv:container>
```

PHP:

```
// app/config/config.php
$container->loadFromExtension('jms_security_extra', array(
    // ...
    'secure_all_services' => true,
));
```

这种方法也有一些缺点，当您激活了服务，初始页面加载可能会很慢并且会取决您已经定义了多少服务。