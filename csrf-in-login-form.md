# 在登录表单中使用 CSRF 保护

当使用一个登录表单时，您应该确保可以抵御 CSRF ([跨站点请求伪造](https://en.wikipedia.org/wiki/Cross-site_request_forgery))。安全组件已经对 CSRF 内置支持。在这篇文章中，您将学习如何在登录表单中使用它。

> 登录 CSRF 并不是特别知名。如果您想要知道更多详细信息，请参阅[锻造登录请求](https://en.wikipedia.org/wiki/Cross-site_request_forgery#Forging_login_requests)。

## 配置 CSRF 保护

首先，配置安全组件来让它可以使用 CSRF 保护。安全组件需要 CSRF 令牌提供程序。您可以使用安全组件中默认的提供程序：

YMAL:

```
# app/config/security.yml
security:
    firewalls:
        secured_area:
            # ...
            form_login:
                # ...
                csrf_provider: security.csrf.token_manager
```

XML:

```
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<srv:container xmlns="http://symfony.com/schema/dic/security"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:srv="http://symfony.com/schema/dic/services"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

    <config>
        <firewall name="secured_area">
            <!-- ... -->

            <form-login csrf-provider="security.csrf.token_manager" />
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
            'form_login' => array(
                // ...
                'csrf_provider' => 'security.csrf.token_manager',
            )
        )
    )
));
```

可以进一步配置安全组件，但这必须要能够在登录表单中使用 CSRF 攻击所需的所有信息。

## 渲染 CSRF 字段

既然该安全组件将检查 CSRF 令牌，您必须向包含 CSRF 令牌的登录表单中添加隐藏的字段。默认情况下，此字段被命名为  **_csrf_token**。该隐藏的字段必须包含 CSRF 令牌，该令牌可以使用 **csrf_token** 函数生成。该函数需要一个令牌的 ID，并且当使用登陆表单的时候该 ID 必须用来进行**身份验证**：

Twig:

```
{# src/Acme/SecurityBundle/Resources/views/Security/login.html.twig #}

{# ... #}
<form action="{{ path('login_check') }}" method="post">
    {# ... the login fields #}

    <input type="hidden" name="_csrf_token"
        value="{{ csrf_token('authenticate') }}"
    >

    <button type="submit">login</button>
</form>
```

PHP:

```
<!-- src/Acme/SecurityBundle/Resources/views/Security/login.html.php -->

<!-- ... -->
<form action="<?php echo $view['router']->generate('login_check') ?>" method="post">
    <!-- ... the login fields -->

    <input type="hidden" name="_csrf_token"
        value="<?php echo $view['form']->csrfToken('authenticate') ?>"
    >

    <button type="submit">login</button>
</form>
```

经过上述步骤，您已经可以防御 CSRF 攻击您登录表单了。

您可以通过设置 **csrf_parameter** 来更改字段的名称并通过在您的配置中设置**意愿**来更改令牌 ID ：

YAML:

```
# app/config/security.yml
security:
    firewalls:
        secured_area:
            # ...
            form_login:
                # ...
                csrf_parameter: _csrf_security_token
                intention: a_private_string
```

XML:

```
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<srv:container xmlns="http://symfony.com/schema/dic/security"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:srv="http://symfony.com/schema/dic/services"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

    <config>
        <firewall name="secured_area">
            <!-- ... -->

            <form-login csrf-parameter="_csrf_security_token"
                intention="a_private_string" />
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
            'form_login' => array(
                // ...
                'csrf_parameter' => '_csrf_security_token',
                'intention'      => 'a_private_string',
            )
        )
    )
));
```
