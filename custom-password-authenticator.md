# 如何创建自定义表单密码验证器

想象一下，如果您想要使您的网站只能在下午两点到四点之间才能被访问。在 Symfony 2.4 之前必须创建一个自定义的令牌、 工厂、 监听器和提供者才能实现。在本节中，您将学习如何在一个登录表单（即您的用户提交他们的用户名和密码的页面）中实现上述功能 。在 Symfony 2.6 之前，您必须使用密码编码器来验证用户的密码。

## 密码身份验证器

> **2\.6** 在 Symfony 2.6 介绍了 **UserPasswordEncoderInterface** 接口。

首先，创建新的类来实现 [SimpleFormAuthenticatorInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Authentication/SimpleFormAuthenticatorInterface.html) 接口。最终，这将允许您创建自定义逻辑来对用户进行身份验证:

```
// src/Acme/HelloBundle/Security/TimeAuthenticator.php
namespace Acme\HelloBundle\Security;

use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Security\Core\Authentication\SimpleFormAuthenticatorInterface;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Authentication\Token\UsernamePasswordToken;
use Symfony\Component\Security\Core\Encoder\UserPasswordEncoderInterface;
use Symfony\Component\Security\Core\Exception\AuthenticationException;
use Symfony\Component\Security\Core\Exception\UsernameNotFoundException;
use Symfony\Component\Security\Core\User\UserProviderInterface;

class TimeAuthenticator implements SimpleFormAuthenticatorInterface
{
    private $encoder;

    public function __construct(UserPasswordEncoderInterface $encoder)
    {
        $this->encoder = $encoder;
    }

    public function authenticateToken(TokenInterface $token, UserProviderInterface $userProvider, $providerKey)
    {
        try {
            $user = $userProvider->loadUserByUsername($token->getUsername());
        } catch (UsernameNotFoundException $e) {
            throw new AuthenticationException('Invalid username or password');
        }

        $passwordValid = $this->encoder->isPasswordValid($user, $token->getCredentials());

        if ($passwordValid) {
            $currentHour = date('G');
            if ($currentHour < 14 || $currentHour > 16) {
                throw new AuthenticationException(
                    'You can only log in between 2 and 4!',
                    100
                );
            }

            return new UsernamePasswordToken(
                $user,
                $user->getPassword(),
                $providerKey,
                $user->getRoles()
            );
        }

        throw new AuthenticationException('Invalid username or password');
    }

    public function supportsToken(TokenInterface $token, $providerKey)
    {
        return $token instanceof UsernamePasswordToken
            && $token->getProviderKey() === $providerKey;
    }

    public function createToken(Request $request, $username, $password, $providerKey)
    {
        return new UsernamePasswordToken($username, $password, $providerKey);
    }
}
```

## 它是怎样工作的

很好!现在你只需要设置一些配置。但首先，你可以了解更多有关此类中的每个方法的作用。

## 1）createToken 方法

当 Symfony 开始处理请求，**createToken()** 方法会被调用，在这个方法中会创建一个 [TokenInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Authentication/Token/TokenInterface.html)  对象，并且该对象包含的所有您需要的信息都会在 **authenticateToken()** 中进行用户身份验证 (例如用户名和密码) 。

您在这里创建的所有令牌对象随后都将通过 **authenticateToken()** 方法传递给您。

## 2）supportsToken 方法

当 Symfony 调用 **createToken()** 方法后，它将会调用您创建的类中的 **supportsToken()** 方法(和任何其它身份验证监听器) 来弄清谁应该处理令牌。这仅仅是一种允许同一个防火墙使用几种身份验证机制的方法。 (通过这种方式，例如您可以首先通过证书或者 API 秘钥来对用户进行身份验证然后再回退到登录表单)。

大多数情况下，你只需要确保在这个方法中，给 **createToken()** 方法中建立的令牌返回一个真值。您的程序逻辑应该如同这个例子一样。

## 3）authenticateToken 方法

如果 **supportsToken** 方法返回 true，Symfony 将会调用 **authenticateToken()** 方法。您现在应该做的是检查令牌是否允许首先通过的用户提供程序来获得用户对象，然后通过检查密码和当前时间来登录。 

> 关于如何获取**用户**对象以及确定令牌是否有效 (例如检查密码)的"流"，可能会随着您的需求而改变。

最终，您的任务是返回一个新的并且已经"身份验证"过的令牌对象（即：至少给它设定了一个角色)，并且这个令牌对象中含有**用户**对象。

在这个方法中，需要使用密码编码器被来检查密码的有效性:

```
$passwordValid = $this->encoder->isPasswordValid($user, $token->getCredentials());
```

这已经是 Symfony 中可用的服务，并且它使用的是**编码**秘钥下的安全配置 (例如 **security.yml**) 中的密码算法。下面，您将会看到如何把它注入 到 **TimeAuthenticator** 中。

## 配置

现在，把您的 TimeAuthenticator 作为一种服务来配置:

YAML:

```
# app/config/config.yml
services:
    # ...

    time_authenticator:
        class:     Acme\HelloBundle\Security\TimeAuthenticator
        arguments: ["@security.password_encoder"]
```

XML:

```
<!-- app/config/config.xml -->
<?xml version="1.0" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">
    <services>
        <!-- ... -->

        <service id="time_authenticator"
            class="Acme\HelloBundle\Security\TimeAuthenticator"
        >
            <argument type="service" id="security.password_encoder" />
        </service>
    </services>
</container>
```

PHP:

```
// app/config/config.php
use Symfony\Component\DependencyInjection\Definition;
use Symfony\Component\DependencyInjection\Reference;

// ...

$container->setDefinition('time_authenticator', new Definition(
    'Acme\HelloBundle\Security\TimeAuthenticator',
    array(new Reference('security.password_encoder'))
));
```

接着，使用 simple_form 秘钥在安全配置中的防火墙中激活它:

YAML:

```
# app/config/security.yml
security:
    # ...

    firewalls:
        secured_area:
            pattern: ^/admin
            # ...
            simple_form:
                authenticator: time_authenticator
                check_path:    login_check
                login_path:    login
```

XML:

```
<!-- app/config/security.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<srv:container xmlns="http://symfony.com/schema/dic/security"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:srv="http://symfony.com/schema/dic/services"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">
    <config>
        <!-- ... -->

        <firewall name="secured_area"
            pattern="^/admin"
            >
            <simple-form authenticator="time_authenticator"
                check-path="login_check"
                login-path="login"
            />
        </firewall>
    </config>
</srv:container>
```

PHP:

```
// app/config/security.php

// ..

$container->loadFromExtension('security', array(
    'firewalls' => array(
        'secured_area'    => array(
            'pattern'     => '^/admin',
            'simple_form' => array(
                'provider'      => ...,
                'authenticator' => 'time_authenticator',
                'check_path'    => 'login_check',
                'login_path'    => 'login',
            ),
        ),
    ),
));
```

**Simple_form** 秘钥具有和正常的 form_login 相同的选项，但在 **simple_form** 秘钥中具有指向新服务的附加的身份验证器秘钥。有关详细信息，请参阅[表单登录配置](http://symfony.com/doc/current/reference/configuration/security.html#reference-security-firewall-form-login)。

一般情况下，如果您对如何创建一个新的登录表单不熟悉或者是不明白 **check_path** 和 **login_path** 选项，请参阅[如何自定义您的表单登录](http://symfony.com/doc/current/cookbook/security/form_login.html)。