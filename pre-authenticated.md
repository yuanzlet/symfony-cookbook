# 使用预认证安全防火墙

某些 web 服务器，包括 Apache 已经提供大量的身份验证模块。这些模块一般设置一些环境变量来确定哪个用户正在访问您的应用程序。不确定的是，Symfony 支持大多数的身份验证机制。这些请求被称为预身份验证请求，因为当该用户访问您的程序的时候他已经通过了身份验证。

## X.509 客户端证书身份验证

当使用客户端证书时，您的网络服务器会进行所有的身份验证过程。举个例子，在 Apache 服务器下您可以使用 **SSLVerifyClient** 需求指令。

为安全配置中的特定防火墙的启用 x 509 身份验证:

YAML:

```
# app/config/security.yml
security:
    firewalls:
        secured_area:
            pattern: ^/
            x509:
                provider: your_user_provider
```

XML:

```
<!-- app/config/security.xml -->
<?xml version="1.0" ?>
<srv:container xmlns="http://symfony.com/schema/dic/security"
    xmlns:srv="http://symfony.com/schema/dic/services">

    <config>
        <firewall name="secured_area" pattern="^/">
            <x509 provider="your_user_provider"/>
        </firewall>
    </config>
</srv:container>
```

PHP:

```
// app/config/security.php
$container->loadFromExtension('security', array(
    'firewalls' => array(
        'secured_area' => array(
            'pattern' => '^/'
            'x509'    => array(
                'provider' => 'your_user_provider',
            ),
        ),
    ),
));
```

默认情况下，防火墙向用户提供程序提供 **SSL_CLIENT_S_DN_Email** 变量，并将 **SSL_CLIENT_S_DN** 设置为 [PreAuthenticatedToken](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Authentication/Token/PreAuthenticatedToken.html) 中的凭据。您可以通过分别在 x 509 防火墙配置中设置**用户**和**凭据**密钥来重载它们。

> 身份验证提供程序只会通知用户提供程序已经做出请求的用户名。您将需要创建 (或使用) **提供程序**配置参数引用的  的"用户提供程序"(如配置示例中的 **your_user_provider**)。此提供程序将会把该用户名变成您选择的一个用户对象。有关创建或配置用户提供程序的详细信息，请参阅:

> - [如何创建一个自定义用户提供程序](http://symfony.com/doc/current/cookbook/security/custom_provider.html)

> - [如何从数据库 (实体提供程序) 加载安全用户](http://symfony.com/doc/current/cookbook/security/entity_provider.html)

## 基于验证的 REMOTE_USER

> **2\.6** 在 Symfony 2.6 介绍了 REMOTE_USER 预身份验证防火墙。

许多身份验证模块，比如为 Apache 中的 **auth_kerb** 使用 **REMOTE_USER** 环境变量来提供用户名。如果身份验证发生在请求它之前，那么应用程序将会信任此变量。

使用 **REMOTE_USER** 环境变量来配置 Symfony ，只需在安全配置中启用相应的防火墙:

YAML:

```
# app/config/security.yml
security:
    firewalls:
        secured_area:
            pattern: ^/
            remote_user:
                provider: your_user_provider
```

XML:

```
<!-- app/config/security.xml -->
<?xml version="1.0" ?>
<srv:container xmlns="http://symfony.com/schema/dic/security"
    xmlns:srv="http://symfony.com/schema/dic/services">

    <config>
        <firewall name="secured_area" pattern="^/">
            <remote-user provider="your_user_provider"/>
        </firewall>
    </config>
</srv:container>
```

PHP:

```
// app/config/security.php
$container->loadFromExtension('security', array(
    'firewalls' => array(
        'secured_area' => array(
            'pattern'     => '^/'
            'remote_user' => array(
                'provider' => 'your_user_provider',
            ),
        ),
    ),
));
```

防火墙将给您的您用户的提供程序提供 **REMOTE_USER** 环境变量。您可以通过设置 **remote_user**  防火墙配置中的用户密钥来改变变量名称。

> 就像 X509 身份验证，您需要配置"用户提供程序"。请参阅[以前的注意栏目](http://symfony.com/doc/current/cookbook/security/pre_authenticated.html#cookbook-security-pre-authenticated-user-provider-note)来获得更多的信息。
