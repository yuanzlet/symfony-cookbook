# 如何创建自定义用户提供者

Symfony 的标准身份验证过程的一部分取决于"用户服务提供者"。当用户提交一个用户名和密码时，身份验证层便请求配置用户信息的程序返回一个用户对象来提供用户名。随后 Symfony 会检查此用户的密码是否正确，如果是正确的就会生成一个安全令牌，使用户在当前会话期间保持已经验证过的身份。当然不确定的是，Symfony 有一个 "in_memory" 和一个 "entity" 用户提供程序。在本节中，您可以了解到如何创建您自己的用户提供程序，如果您的用户通过一个自定义的数据库，一个文件，或者是这个例子中的 - 网络服务来访问该程序，那么该提供程序会非常有用。

## 创建一个用户类

首先，无论您从哪里获得用户数据，您都需要创建一个表示该数据的用户类。用户可以看到任何您想要和包含的数据。唯一的要求是该类实现[用户接口](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/UserInterface.html "用户接口")。因此如下接口中的方法应该定义在自定义用户类中: [getRoles()]( http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/UserInterface.html#getRoles()"getRoles()")、 [getPassword()]( http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/UserInterface.html#getPassword()"getPassword()")、 [getSalt()]( http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/UserInterface.html#getSalt()"getSalt()")、 [getUsername()]( http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/UserInterface.html#getRoles()"getUsername()")、 [eraseCredentials()]( http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/UserInterface.html#eraseCredentials()"eraseCredentials()")。它也可能有助于实现 [EquatableInterface]( http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/EquatableInterface.html "EquatableInterface") 接口，在这个接口中定义了一个方法来检查用户是否等于当前用户。此接口需要包含 [isEqualTo()](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/EquatableInterface.html#isEqualTo() "isEqualTo()") 方法。

您的 **WebserviceUser** 如下所示：

```
// src/Acme/WebserviceUserBundle/Security/User/WebserviceUser.php
namespace Acme\WebserviceUserBundle\Security\User;

use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Security\Core\User\EquatableInterface;

class WebserviceUser implements UserInterface, EquatableInterface
{
    private $username;
    private $password;
    private $salt;
    private $roles;

    public function __construct($username, $password, $salt, array $roles)
    {
        $this->username = $username;
        $this->password = $password;
        $this->salt = $salt;
        $this->roles = $roles;
    }

    public function getRoles()
    {
        return $this->roles;
    }

    public function getPassword()
    {
        return $this->password;
    }

    public function getSalt()
    {
        return $this->salt;
    }

    public function getUsername()
    {
        return $this->username;
    }

    public function eraseCredentials()
    {
    }

    public function isEqualTo(UserInterface $user)
    {
        if (!$user instanceof WebserviceUser) {
            return false;
        }

        if ($this->password !== $user->getPassword()) {
            return false;
        }

        if ($this->salt !== $user->getSalt()) {
            return false;
        }

        if ($this->username !== $user->getUsername()) {
            return false;
        }

        return true;
    }
}
```

如果您想让您的用户有更多或者更详细的信息，比如像用户的“姓”，那么您可以添加一个“姓”的字段来存放该数据。

## 创建一个用户提供程序

现在，您有一个**用户类**，您将创建用户提供程序，该程序可以从一些网络服务中抓取用户的信息，创建一个 **WebserviceUser**  对象，并且给它填充上数据。

用户提供程序只是一个需要实现 [UserProviderInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/UserProviderInterface.html "UserProviderInterface") 接口的普通 PHP 类，在该接口中需要定义的三种方法: **loadUserByUsername($username)**,**refreshUser (UserInterface $user)** 和 **supportsClass($class)**。有关更多详细信息，请参阅 [UserProviderInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/UserProviderInterface.html "UserProviderInterface")。

该接口可以如同下面的代码描述一样：

```
// src/Acme/WebserviceUserBundle/Security/User/WebserviceUserProvider.php
namespace Acme\WebserviceUserBundle\Security\User;

use Symfony\Component\Security\Core\User\UserProviderInterface;
use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Security\Core\Exception\UsernameNotFoundException;
use Symfony\Component\Security\Core\Exception\UnsupportedUserException;

class WebserviceUserProvider implements UserProviderInterface
{
    public function loadUserByUsername($username)
    {
        // make a call to your webservice here
        $userData = ...
        // pretend it returns an array on success, false if there is no user

        if ($userData) {
            $password = '...';

            // ...

            return new WebserviceUser($username, $password, $salt, $roles);
        }

        throw new UsernameNotFoundException(
            sprintf('Username "%s" does not exist.', $username)
        );
    }

    public function refreshUser(UserInterface $user)
    {
        if (!$user instanceof WebserviceUser) {
            throw new UnsupportedUserException(
                sprintf('Instances of "%s" are not supported.', get_class($user))
            );
        }

        return $this->loadUserByUsername($user->getUsername());
    }

    public function supportsClass($class)
    {
        return $class === 'Acme\WebserviceUserBundle\Security\User\WebserviceUser';
    }
}
```

## 为用户提供程序创建一个服务

现在您可以把用户提供程序作为一种服务:

YAML:

```
# src/Acme/WebserviceUserBundle/Resources/config/services.yml
services:
    webservice_user_provider:
        class: Acme\WebserviceUserBundle\Security\User\WebserviceUserProvider
```

XML:

```
<!-- src/Acme/WebserviceUserBundle/Resources/config/services.xml -->
<services>
    <service id="webservice_user_provider" class="Acme\WebserviceUserBundle\Security\User\WebserviceUserProvider" />
</services>
```

PHP:

```
// src/Acme/WebserviceUserBundle/Resources/config/services.php
use Symfony\Component\DependencyInjection\Definition;

$container->setDefinition(
    'webservice_user_provider',
    new Definition('Acme\WebserviceUserBundle\Security\User\WebserviceUserProvider')
);
```

> 真正的实现用户提供程序可能会需要有一些依赖关系或配置选项或是其他服务并且把它们在服务定义中设定为参数。

> 如何确保该服务文件被导入，详情请参阅带有输入的[导入配置](http://symfony.com/doc/current/book/service_container.html#service-container-imports-directive)。

## 修改 security.yml

在您的安全配置中包含了所有的属性。我们把用户提供程序添加到"安全"一节中的提供程序的列表中并且为用户提供程序 (例如"web 服务") 选择一个名称，然后记录下您刚刚定义的服务的id。

YAML:

```
# app/config/security.yml
security:
    providers:
        webservice:
            id: webservice_user_provider
```

XML:

```
<!-- app/config/security.xml -->
<config>
    <provider name="webservice" id="webservice_user_provider" />
</config>
```

PHP:

```
// app/config/security.php
$container->loadFromExtension('security', array(
    'providers' => array(
        'webservice' => array(
            'id' => 'webservice_user_provider',
        ),
    ),
));
```

Symfony 也需要知道如何对网络上的用户输入的密码进行编码，例如用户通过填写一个登录表单提交的密码。您可以通过在您的安全配置文件中添加一行有关“编码器”一节中提到的代码来实现上述功能：

YAML:

```
# app/config/security.yml
security:
    encoders:
        Acme\WebserviceUserBundle\Security\User\WebserviceUser: sha512
```

XML:

```
<!-- app/config/security.xml -->
<config>
    <encoder class="Acme\WebserviceUserBundle\Security\User\WebserviceUser">sha512</encoder>
</config>
```

PHP:

```
// app/config/security.php
$container->loadFromExtension('security', array(
    'encoders' => array(
        'Acme\WebserviceUserBundle\Security\User\WebserviceUser' => 'sha512',
    ),
));
```

当创建您的用户的时候（不管这些用户是怎样创建的）不管您的密码最初是怎样被编码的，它的值都必须符合密码编码规则。当一个用户提交他的密码的时候，混淆值将被追加到密码上，然后用相应的算法对它进行编码，然再在和 getPassword() 方法返回的哈希值进行比较。此外，根据您的选项，密码可能有编码多次并且被编成 base64 进制码。

> ## 关于密码如何进行编码的细节信息

> Symfony 使用特定的方法来合并混淆值并且在和已经编码好的密码比较之前就完成对密码的编码。如果 getSalt() 没有返回任何值，那么仅仅使用了您在 security.yml 中指定的算法对您提交的密码进行编码。如果明确指定了一种混淆值，那么下面列举的值已经被创建了，并且通过该算法对其进行散列处理:

> **$password.'{'.$salt.'}';**

> 如果外部用户通过不同的方法来对他们的密码进行混淆加密，那么你就需要做更多的工作，以保证 Symfony 能正确的对密码进行编码。这超出了本章的介绍范围，但可以通过包含 **MessageDigestPasswordEncoder** 子类并且重载 **mergePasswordAndSalt** 方法。

> 此外，在默认情况下，哈希值将会被多次编码然后被转化为 64 进制编码。了解详情请参阅 MessageDigestPasswordEncoder 一章。为了防止出现这种情况，请在您的配置文件中这样配置:

YAML:

```
# app/config/security.yml
security:
    encoders:
        Acme\WebserviceUserBundle\Security\User\WebserviceUser:
            algorithm: sha512
            encode_as_base64: false
            iterations: 1
```

XML:

```
<!-- app/config/security.xml -->
<config>
    <encoder class="Acme\WebserviceUserBundle\Security\User\WebserviceUser"
        algorithm="sha512"
        encode-as-base64="false"
        iterations="1"
    />
</config>
```

PHP:

```
// app/config/security.php
$container->loadFromExtension('security', array(
    'encoders' => array(
        'Acme\WebserviceUserBundle\Security\User\WebserviceUser' => array(
            'algorithm'         => 'sha512',
            'encode_as_base64'  => false,
            'iterations'        => 1,
        ),
    ),
));
```
