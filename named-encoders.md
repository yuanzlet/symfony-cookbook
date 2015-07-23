# 如何动态选择密码加密算法

通常情况下，我们通过配置一个密码编码器使它能够适用于特定的类的所有实例来实现它可以被所有用户使用：

YAML:

```
# app/config/security.yml
security:
    # ...
    encoders:
        Symfony\Component\Security\Core\User\User: sha512
```

XML:

```
<!-- app/config/security.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<srv:container xmlns="http://symfony.com/schema/dic/security"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:srv="http://symfony.com/schema/dic/services"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd"
>
    <config>
        <!-- ... -->
        <encoder class="Symfony\Component\Security\Core\User\User"
            algorithm="sha512"
        />
    </config>
</srv:container>
```

PHP:

```
// app/config/security.php
$container->loadFromExtension('security', array(
    // ...
    'encoders' => array(
        'Symfony\Component\Security\Core\User\User' => array(
            'algorithm' => 'sha512',
        ),
    ),
));
```

另一个选择是使用一个"指定的"编码器，然后选择您想要动态使用的编码器。

在前面的示例中，您已经为 **Acme\UserBundle\Entity\User** 设置了 **sha512** 算法。对于普通的用户这可能是足够安全的，但如果您想您的管理员拥有更强的算法，例如 **bcrypt**。那么可以通过制定的编码器来实现：

YAML:

```
# app/config/security.yml
security:
    # ...
    encoders:
        harsh:
            algorithm: bcrypt
            cost: 15
```

XML:

```
<!-- app/config/security.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<srv:container xmlns="http://symfony.com/schema/dic/security"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:srv="http://symfony.com/schema/dic/services"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd"
>

    <config>
        <!-- ... -->
        <encoder class="harsh"
            algorithm="bcrypt"
            cost="15" />
    </config>
</srv:container>
```

PHP:

```
// app/config/security.php
$container->loadFromExtension('security', array(
    // ...
    'encoders' => array(
        'harsh' => array(
            'algorithm' => 'bcrypt',
            'cost'      => '15'
        ),
    ),
));
```

在这里我们将创建名为 **harsh** 的编码器。为了使**用户**实例可以使用它，该类必须实现 [EncoderAwareInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Encoder/EncoderAwareInterface.html) 接口。该接口应该具有一个 - **getEncoderName** - 方法，该方法应返回编码器的名称:

```
// src/Acme/UserBundle/Entity/User.php
namespace Acme\UserBundle\Entity;

use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Security\Core\Encoder\EncoderAwareInterface;

class User implements UserInterface, EncoderAwareInterface
{
    public function getEncoderName()
    {
        if ($this->isAdmin()) {
            return 'harsh';
        }

        return null; // use the default encoder
    }
}
```
