#如何从数据库(实体提供者)读取安全用户

Symfony 的安全系统可以通过活动目录或开放授权服务器像数据库一样从任何地方加载安全用户。这篇文章将告诉你如何通过一个 Doctrine entity 从数据库加载用户信息。

## 前言

> 在开始之前，您应该检查 [FOSUserBundle](https://github.com/FriendsOfSymfony/FOSUserBundle)。这个外部包允许您从数据库加载用户信息(就像你会在这里学到的)并为您提供内置路径和控制器用于一些事务，比如登录、注册和忘记密码。但是，如果您需要大量自定义用户系统或者如果你想了解工作原理，本教程是更好的。

通过 Doctrine entity 加载用户信息有两个基本步骤：

1. [创建您的用户对象](http://symfony.com/doc/current/cookbook/security/entity_provider.html#security-crete-user-entity)；
2. [把 security.yml 配置为从对象加载](http://symfony.com/doc/current/cookbook/security/entity_provider.html#security-config-entity-provider)。

之后，您可以了解更多关于[禁止非活动用户](http://symfony.com/doc/current/cookbook/security/entity_provider.html#security-advanced-user-interface)，[使用自定义查询](http://symfony.com/doc/current/cookbook/security/entity_provider.html#authenticating-someone-with-a-custom-entity-provider)和[序列化会话用户](http://symfony.com/doc/current/cookbook/security/entity_provider.html#cookbook-security-serialize-equatable)的相关信息。

## 1） 创建您的用户实体

此项，假设您在一个**应用程序包**内已经有一个**用户**对象包含下列字段：**id**，**username**，**password**，**email** 和 **isActive**。

```PHP
// src/AppBundle/Entity/User.php
namespace AppBundle\Entity;

use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Security\Core\User\UserInterface;

/**
 * @ORM\Table(name="app_users")
 * @ORM\Entity(repositoryClass="AppBundle\Entity\UserRepository")
 */
class User implements UserInterface, \Serializable
{
    /**
     * @ORM\Column(type="integer")
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    private $id;

    /**
     * @ORM\Column(type="string", length=25, unique=true)
     */
    private $username;

    /**
     * @ORM\Column(type="string", length=64)
     */
    private $password;

    /**
     * @ORM\Column(type="string", length=60, unique=true)
     */
    private $email;

    /**
     * @ORM\Column(name="is_active", type="boolean")
     */
    private $isActive;

    public function __construct()
    {
        $this->isActive = true;
        // may not be needed, see section on salt below
        // $this->salt = md5(uniqid(null, true));
    }

    public function getUsername()
    {
        return $this->username;
    }

    public function getSalt()
    {
        // you *may* need a real salt depending on your encoder
        // see section on salt below
        return null;
    }

    public function getPassword()
    {
        return $this->password;
    }

    public function getRoles()
    {
        return array('ROLE_USER');
    }

    public function eraseCredentials()
    {
    }

    /** @see \Serializable::serialize() */
    public function serialize()
    {
        return serialize(array(
            $this->id,
            $this->username,
            $this->password,
            // see section on salt below
            // $this->salt,
        ));
    }

    /** @see \Serializable::unserialize() */
    public function unserialize($serialized)
    {
        list (
            $this->id,
            $this->username,
            $this->password,
            // see section on salt below
            // $this->salt
        ) = unserialize($serialized);
    }
}
```

为了让代码简短，一些 getter 和 setter 方法没有显示。但你可以通过运行生成这些：

```
$ php app/console doctrine:generate:entities AppBundle/Entity/User
```

接下来，确保[创建数据库表](http://symfony.com/doc/current/book/doctrine.html#book-doctrine-creating-the-database-tables-schema)：

```
$ php app/console doctrine:schema:update --force
```

### 什么是用户接口？

目前为止，这只是一个普通的对象。 但是为了在安全系统中使用此类，它必须实现[用户接口](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/UserInterface.html)。这要求该类有以下五种方法：

- [getRoles()](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/UserInterface.html#getRoles())
- [getPassword()](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/UserInterface.html#getPassword())
- [getSalt()](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/UserInterface.html#getSalt())
- [getUsername()](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/UserInterface.html#getUsername())
- [eraseCredentials()](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/UserInterface.html#eraseCredentials())

要了解这五个方法的更多信息，请参见[用户接口](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/UserInterface.html)。

### 序列化和反序列化方法做什么？

在每个请求末，用户对象是序列化会话的。对下一个请求，它是非序列化的。要帮助 PHP 正确做到这一点，您需要实现**可序列化**。但你不必序列化任何东西：你只需要几个字段（上面所示的那些加上一些额外的字段，如果你决定实现 [AdvancedUserInterface](http://symfony.com/doc/current/cookbook/security/entity_provider.html#security-advanced-user-interface) 的话）。对于每个请求，**id** 用于从数据库中查询一个新的**用户**对象。

想要了解更多吗？详见[理解序列化和用户如何在会话中进行保存](http://symfony.com/doc/current/cookbook/security/entity_provider.html#cookbook-security-serialize-equatable)。

## 2） 从对象加载配置安全性

现在，您已经有了一个实现了**用户接口**的**用户**对象，你只需要在 **security.yml** 中把这些告诉 Symfony 的安全系统。

在这个例子中，用户将通过 HTTP 基本身份验证输入用户名和密码。Symfony 会查询一个和用户名匹配的用户对象，然后检查密码(通常检查密码的用时较短)：

YAML:

```
# app/config/security.yml
security:
    encoders:
        AppBundle\Entity\User:
            algorithm: bcrypt

    # ...

    providers:
        our_db_provider:
            entity:
                class: AppBundle:User
                property: username
                # if you're using multiple entity managers
                # manager_name: customer

    firewalls:
        default:
            pattern:    ^/
            http_basic: ~
            provider: our_db_provider

    # ...
```

XML:

```
<!-- app/config/security.xml -->
<config>
    <encoder class="AppBundle\Entity\User"
        algorithm="bcrypt"
    />

    <!-- ... -->

    <provider name="our_db_provider">
        <entity class="AppBundle:User" property="username" />
    </provider>

    <firewall name="default" pattern="^/" provider="our_db_provider">
        <http-basic />
    </firewall>

    <!-- ... -->
</config>
```

PHP:

```
// app/config/security.php
$container->loadFromExtension('security', array(
    'encoders' => array(
        'AppBundle\Entity\User' => array(
            'algorithm' => 'bcrypt',
        ),
    ),
    // ...
    'providers' => array(
        'our_db_provider' => array(
            'entity' => array(
                'class'    => 'AppBundle:User',
                'property' => 'username',
            ),
        ),
    ),
    'firewalls' => array(
        'default' => array(
            'pattern' => '^/',
            'http_basic' => null,
            'provider' => 'our_db_provider',
        ),
    ),
    // ...
));
```

首先，**encoder** 部分告诉 Symfony 期望使用 **bcrypt** 对数据库中的密码进行编码。第二，**providers** 部分创建一个叫 **our\_db\_provider** 的 "user provider"，它知道通过 **username** 属性在您的 **AppBundle:User** 对象中查询。其中，**our\_db\_provider** 这个名字并不重要：它只需要在你的防火墙下匹配 **provider** 的密钥的值。或者，如果您没有在防火墙下设置 **provider** 密钥，第一个 “user provider” 会被自动使用。

> If you're using PHP 5.4 or lower, you'll need to install the ircmaxell/password-compat library via Composer in order to be able to use the bcrypt encoder:如果您使用的是PHP 5.4 或更低版本，为了能够使用 **bcrypt** 编码器，您需要通过 Composer 安装 **ircmaxell/password-compat** 库：

> ```
> {
>     "require": {
>         ...
>         "ircmaxell/password-compat": "~1.0.3"
>     }
> }
> ```

### 创建您的第一个用户

要添加用户，您可以实现一个[注册表单](http://symfony.com/doc/current/cookbook/doctrine/registration_form.html)或添加一些 [fixtures](https://symfony.com/doc/master/bundles/DoctrineFixturesBundle/index.html)。这只是一个普通的对象，所以没什么棘手，除了您需要对每个用户的密码进行加密。但别担心，Symfony 会给您一个用来实现此事的服务。有关详细信息，请参见[动态编码密码](http://symfony.com/doc/current/book/security.html#security-encoding-password)。

下面是从 MySQL 中导出的 app\_users 表，包含了用户 admin 和密码 admin （密码是加密过的）。

```
$ mysql> SELECT * FROM app_users;
+----+----------+--------------------------------------------------------------+--------------------+-----------+
| id | username | password                                                     | email              | is_active |
+----+----------+--------------------------------------------------------------+--------------------+-----------+
|  1 | admin    | $2a$08$jHZj/wJfcVKlIwr5AvR78euJxYK7Ku5kURNhNx.7.CSIJ3Pq6LEPC | admin@example.com  |         1 |
+----+----------+--------------------------------------------------------------+--------------------+-----------+
```

> 您需要一个 salt 属性吗？
> 如果您使用 **bcrypt**，不需要。否则，是的。所有的密码必须用一个 salt 进行哈希处理，但是 **bcrypt** 内部做了这件事。由于本教程使用 **bcrypt** ，**User** 中的 **getSalt()** 方法只能返回**空值**（它没有被使用）。如果你使用了一个不同的算法，您需要在**用户**对象中取消对 **salt** 行的注释，并且添加一个持久的 **salt** 属性。

## 禁止非活动用户(AdvancedUserInterface)

如果用户 **isActive** 属性设置为 **false** (例如，**is_active** 在数据库中是 0)，用户仍然可以正常登录到网站。这是很容易修正的。

排除非活动用户，更改您的用户类来实现 AdvancedUserInterface。这扩展了互动演示界面，所以你只需要新的界面：

```
// src/AppBundle/Entity/User.php

use Symfony\Component\Security\Core\User\AdvancedUserInterface;
// ...

class User implements AdvancedUserInterface, \Serializable
{
    // ...

    public function isAccountNonExpired()
    {
        return true;
    }

    public function isAccountNonLocked()
    {
        return true;
    }

    public function isCredentialsNonExpired()
    {
        return true;
    }

    public function isEnabled()
    {
        return $this->isActive;
    }

    // serialize and unserialize must be updated - see below
    public function serialize()
    {
        return serialize(array(
            // ...
            $this->isActive
        ));
    }
    public function unserialize($serialized)
    {
        list (
            // ...
            $this->isActive
        ) = unserialize($serialized);
    }
}
```

该 AdvancedUserInterface 接口添加四个额外的方法来验证帐户状态：

- [isAccountNonExpired()](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/AdvancedUserInterface.html#isAccountNonExpired()) 检查用户帐户是否已过期;
- [isAccountNonLocked()](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/AdvancedUserInterface.html#isAccountNonLocked()) 检查用户是否已被锁定;
- [isCredentialsNonExpired()](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/AdvancedUserInterface.html#isCredentialsNonExpired()) 检查用户凭据是否(密码)已过期;
- [isEnabled()](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/AdvancedUserInterface.html#isEnabled()) 检查用户是否已启用.

如果*任何*这些返回 **false**，则用户不会被允许登录。你可以选择坚持所有这些的属性，或任何你需要的(在这个例子中，**isActive** 是从数据库中选出的唯一属性)。

那么方法之间的区别是什么？每个方法返回一个略有不同的错误消息(当你在登录模板提交它们到自定义模式时，这些可以被转换)。

> 如果您使用 **AdvancedUserInterface**，您还需要添加任何由这些方法使用的属性(如 **isActive**)到 **serialize()** 和 **unserialize()** 方法。如果你不这样做，您的用户可能无法从每个请求上的会话中正确反序列化。

恭喜您！您的数据库加载安全系统已完成所有设置！接下来，添加一个真正的登录表单代替 HTTP 基本身份验证或继续阅读其他主题。

## 使用自定义查询加载用户

如果用户可以用他们的用户名或电子邮件登录，这将是很好的，这在数据库中都是独特的。不幸的是，本机实体提供者只能通过单个用户的属性处理查询。

要做到这一点，使您的**用户资料库**执行一个特殊的 [UserProviderInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/UserProviderInterface.html)。此接口需要三个方法：**loadUserByUsername($username)**，**refreshUser(UserInterface $user)** 和 **supportsClass($class)**：

```
// src/AppBundle/Entity/UserRepository.php
namespace AppBundle\Entity;

use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Security\Core\User\UserProviderInterface;
use Symfony\Component\Security\Core\Exception\UsernameNotFoundException;
use Symfony\Component\Security\Core\Exception\UnsupportedUserException;
use Doctrine\ORM\EntityRepository;

class UserRepository extends EntityRepository implements UserProviderInterface
{
    public function loadUserByUsername($username)
    {
        $user = $this->createQueryBuilder('u')
            ->where('u.username = :username OR u.email = :email')
            ->setParameter('username', $username)
            ->setParameter('email', $username)
            ->getQuery()
            ->getOneOrNullResult();

        if (null === $user) {
            $message = sprintf(
                'Unable to find an active admin AppBundle:User object identified by "%s".',
                $username
            );
            throw new UsernameNotFoundException($message);
        }

        return $user;
    }

    public function refreshUser(UserInterface $user)
    {
        $class = get_class($user);
        if (!$this->supportsClass($class)) {
            throw new UnsupportedUserException(
                sprintf(
                    'Instances of "%s" are not supported.',
                    $class
                )
            );
        }

        return $this->find($user->getId());
    }

    public function supportsClass($class)
    {
        return $this->getEntityName() === $class
            || is_subclass_of($class, $this->getEntityName());
    }
}
```

有关这些方法的详细信息，请参阅 [UserProviderInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/UserProviderInterface.html)。

> 别忘了将 repository 类添加到[该实体的映射定义](http://symfony.com/doc/current/book/doctrine.html#book-doctrine-custom-repository-classes)。

为了完成这一点，只需在 **security.yml** 中移除用户提供者的 **property** 键值。

YAML:

```
# app/config/security.yml
security:
    # ...
    providers:
        our_db_provider:
            entity:
                class: AppBundle:User
    # ...
```

XML:

```
<!-- app/config/security.xml -->
<config>
    <!-- ... -->

    <provider name="our_db_provider">
        <entity class="AppBundle:User" />
    </provider>

    <!-- ... -->
</config>
```

PHP:

```
// app/config/security.php
$container->loadFromExtension('security', array(
    ...,
    'providers' => array(
        'our_db_provider' => array(
            'entity' => array(
                'class' => 'AppBundle:User',
            ),
        ),
    ),
    ...,
));
```

这告诉 Symfony 不要为用户自动查询。相反，当有人登录，**用户资料库**上的 **loadUserByUsername()** 方法将被调用。

## 了解序列化以及用户如何在会话中保存

如果你关心在**用户**类中 **serialize()** 方法的重要性或如何将用户对象序列化或反序列化，那么这一节适合于你。如果不是，可以跳过此节。

一旦用户登录，整个用户对象序列化到会话。对下一个请求，用户对象反序列化。然后，**id** 属性的值是用来从数据库中查询一个新的用户对象。最后，新的用户对象与反序列化的用户对象进行比较，以确保它们表示相同的用户。例如，如果由于某种原因，两个用户对象上的**用户名**不匹配，则出于安全原因，该用户将被注销。

尽管这一切会自动发生，但有一些重要的副作用。

首先，[Serializable](http://php.net/manual/en/class.serializable.php)  接口和其**序列化**和**反序列化**方法被添加到允许**用户**类序列化的会话。这可能是也可能不是根据您的设置完成的，但它可能是个好主意。从理论上讲，只有 **id** 需要被序列化，因为 [refreshUser()](http://api.symfony.com/2.7/Symfony/Bridge/Doctrine/Security/User/EntityUserProvider.html#refreshUser()) 方法在每个使用该 **id** 的请求上刷新用户(如上所述)。这给我们一个 "fresh" 用户对象。

但 Symfony 也使用**用户名**、**salt** 和**密码**来验证用户请求之间没有改变(如果你执行它，它也会调用你的 **AdvancedUserInterface** 方法)。未能序列化这些会导致你在每个请求上被注销。如果您的用户实现 [EquatableInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Core/User/EquatableInterface.html)，而不是检查这些属性，你的 **isEqualTo** 方法只是调用，那么您可以检查所需的任何属性。除非你理解这一点，您可能不需要实现该接口或担心这些。
