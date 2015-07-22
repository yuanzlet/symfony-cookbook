# 如何使用多用户提供者

每一种身份验证机制 (例如 HTTP 身份验证，登录表单等) 恰好使用一个提供程序，默认情况下将使用第一个声明的用户提供程序 。但是如果您想要通过配置文件制定一些用户来验证，其他的用户信息存在数据库里该怎么办？我们可能可以通过创建一个新的供应程序并把它和以前的提供程序串联在一起：

YAML:

```
# app/config/security.yml
security:
    providers:
        chain_provider:
            chain:
                providers: [in_memory, user_db]
        in_memory:
            memory:
                users:
                    foo: { password: test }
        user_db:
            entity: { class: Acme\UserBundle\Entity\User, property: username }
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
        <provider name="chain_provider">
            <chain>
                <provider>in_memory</provider>
                <provider>user_db</provider>
            </chain>
        </provider>
        <provider name="in_memory">
            <memory>
                <user name="foo" password="test" />
            </memory>
        </provider>
        <provider name="user_db">
            <entity class="Acme\UserBundle\Entity\User" property="username" />
        </provider>
    </config>
</srv:container>
```

PHP:

```
// app/config/security.php
$container->loadFromExtension('security', array(
    'providers' => array(
        'chain_provider' => array(
            'chain' => array(
                'providers' => array('in_memory', 'user_db'),
            ),
        ),
        'in_memory' => array(
            'memory' => array(
               'users' => array(
                   'foo' => array('password' => 'test'),
               ),
            ),
        ),
        'user_db' => array(
            'entity' => array(
                'class' => 'Acme\UserBundle\Entity\User',
                'property' => 'username',
            ),
        ),
    ),
));
```

现在，所有的身份验证机制都将使用 **chain_provider**（串联提供程序），因为它是第一次被指定的。**Chain_provider** 将依次尝试从 **in_memory**（内存） 和 **user_db**（数据库用户表）提供程序中加载用户。

您还可以通过配置防火墙或个人的身份验证机制来使用一个特定的提供程序。再次申明，除非显式指定提供程序，则始终会使用第一个提供程序：

YAML:

```
# app/config/security.yml
security:
    firewalls:
        secured_area:
            # ...
            pattern: ^/
            provider: user_db
            http_basic:
                realm: "Secured Demo Area"
                provider: in_memory
            form_login: ~
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
        <firewall name="secured_area" pattern="^/" provider="user_db">
            <!-- ... -->
            <http-basic realm="Secured Demo Area" provider="in_memory" />
            <form-login />
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
            // ...
            'pattern' => '^/',
            'provider' => 'user_db',
            'http_basic' => array(
                // ...
                'provider' => 'in_memory',
            ),
            'form_login' => array(),
        ),
    ),
));
```

在此示例中，如果用户试图通过 HTTP 身份验证登录，身份验证系统将使用 **in_memory** 用户提供程序。但如果用户尝试通过表单登录，将使用 **user_db** 提供程序(因为默认它是作为一个整体防火墙)。

有关用户提供程序和防火墙配置的详细信息，请参阅[SecurityBundle 配置 ("安全")](http://symfony.com/doc/current/reference/configuration/security.html)。
