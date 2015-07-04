# 如何建立一个传统的登录表单

> 如果您是某种数据库的用户并需要一个登录表单，那么您应该考虑使用 [FOSUserBundle](https://github.com/FriendsOfSymfony/FOSUserBundle)，它可以帮助您建立您的 User 对象并提供了多路由和控制器用于常见任务，如登录，注册，忘记密码。

本节中，您将构建一个传统的登录表单。当然，当用户登录时，您可以像数据库一样在任何地方加载用户信息。详见 [B) Configuring how Users are Loaded](http://symfony.com/doc/current/book/security.html#security-user-providers)。

本章假定您已经遵循了 [security chapter](http://symfony.com/doc/current/book/security.html) 的开始，并使用 **http_basic** 身份验证正常工作。

首先，在您的防火墙下启用表单登录：

```YAML
# app/config/security.yml
security:
    # ...

    firewalls:
        default:
            anonymous: ~
            http_basic: ~
            form_login:
                login_path: /login
                check_path: /login_check
```

```XML
<!-- app/config/security.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<srv:container xmlns="http://symfony.com/schema/dic/security"
    xmlns:srv="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">

    <config>
        <firewall name="main">
            <anonymous />
            <form-login login-path="/login" check-path="/login_check" />
        </firewall>
    </config>
</srv:container>
```

```PHP
// app/config/security.php
$container->loadFromExtension('security', array(
    'firewalls' => array(
        'main' => array(
            'anonymous'  => array(),
            'form_login' => array(
                'login_path' => '/login',
                'check_path' => '/login_check',
            ),
        ),
    ),
));
```

> **login_path** 和 **check_path** 也可以是路径名称(但不能有强制性的通配符，例如 **/login/{foo}** 在 **foo** 没有默认值)。

现在，在安全系统启动身份验证过程时，它将用户重定向到登录表单 **/login**。直观上说，执行此登录表单是您的工作。首先，在包中创建一个 **SecurityController**：

```PHP
// src/AppBundle/Controller/SecurityController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class SecurityController extends Controller
{
}
```

接下来，创建两个路径：一个路径是为了每个在您 **form_login** 配置下的路径创建(**/login** 和 **/login_check**)：

```Annotations
// src/AppBundle/Controller/SecurityController.php

// ...
use Symfony\Component\HttpFoundation\Request;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

class SecurityController extends Controller
{
    /**
     * @Route("/login", name="login_route")
     */
    public function loginAction(Request $request)
    {
    }

    /**
     * @Route("/login_check", name="login_check")
     */
    public function loginCheckAction()
    {
        // this controller will not be executed,
        // as the route is handled by the Security system
    }
}
```

```YAML
# app/config/routing.yml
login_route:
    path:     /login
    defaults: { _controller: AppBundle:Security:login }

login_check:
    path: /login_check
    # no controller is bound to this route
    # as it's handled by the Security system
```

```XML
<!-- app/config/routing.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        http://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="login_route" path="/login">
        <default key="_controller">AppBundle:Security:login</default>
    </route>

    <route id="login_check" path="/login_check" />
    <!-- no controller is bound to this route
         as it's handled by the Security system -->
</routes>
```

```PHP
// app/config/routing.php
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;

$collection = new RouteCollection();
$collection->add('login_route', new Route('/login', array(
    '_controller' => 'AppBundle:Security:login',
)));

$collection->add('login_check', new Route('/login_check', array()));
// no controller is bound to this route
// as it's handled by the Security system

return $collection;
```

很好！接下来，向 **loginAction** 添加逻辑，它将展示登录表单：

```PHP
// src/AppBundle/Controller/SecurityController.php

public function loginAction(Request $request)
{
    $authenticationUtils = $this->get('security.authentication_utils');

    // get the login error if there is one
    $error = $authenticationUtils->getLastAuthenticationError();

    // last username entered by the user
    $lastUsername = $authenticationUtils->getLastUsername();

    return $this->render(
        'security/login.html.twig',
        array(
            // last username entered by the user
            'last_username' => $lastUsername,
            'error'         => $error,
        )
    );
}
```

> 2.6 **security.authentication_utils** 服务和 [AuthenticationUtils](http://api.symfony.com/2.7/Symfony/Component/Security/Http/Authentication/AuthenticationUtils.html) 类在 Symfony 2.6 中做了介绍。

不要被这种控制器所迷惑。正如您将看到，当用户提交表单时，安全系统会自动为您处理表单提交。如果用户提交了一个无效用户名或密码，此控制器会从安全系统中读取表单提交的错误，以便它可以显示给用户。

换句话说，您的工作是显示登录表单和可能发生的任何登录错误，但安全系统本身负责检查提交的用户名和密码并对用户进行身份验证。

最后，创建模板：

```Twig
{# app/Resources/views/security/login.html.twig #}
{# ... you will probably extends your base template, like base.html.twig #}

{% if error %}
    <div>{{ error.messageKey|trans(error.messageData, 'security') }}</div>
{% endif %}

<form action="{{ path('login_check') }}" method="post">
    <label for="username">Username:</label>
    <input type="text" id="username" name="_username" value="{{ last_username }}" />

    <label for="password">Password:</label>
    <input type="password" id="password" name="_password" />

    {#
        If you want to control the URL the user
        is redirected to on success (more details below)
        <input type="hidden" name="_target_path" value="/account" />
    #}

    <button type="submit">login</button>
</form>
```

```PHP
<!-- src/Acme/SecurityBundle/Resources/views/Security/login.html.php -->
<?php if ($error): ?>
    <div><?php echo $error->getMessage() ?></div>
<?php endif ?>

<form action="<?php echo $view['router']->generate('login_check') ?>" method="post">
    <label for="username">Username:</label>
    <input type="text" id="username" name="_username" value="<?php echo $last_username ?>" />

    <label for="password">Password:</label>
    <input type="password" id="password" name="_password" />

    <!--
        If you want to control the URL the user
        is redirected to on success (more details below)
        <input type="hidden" name="_target_path" value="/account" />
    -->

    <button type="submit">login</button>
</form>
```

> 传递到模板的**错误**变量是一个 [AuthenticationException](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Exception/AuthenticationException.html) 的实例。它可能包含更多的信息，甚至有关身份验证失败的敏感信息，所以要明智地使用它！

表单的样式不限，但要符合以下的一些要求：

- 表单必须 POST 到 **/login_check**，因为这是您在 **security.yml** 中 **form_login** 键下的配置。

- 用户名必须有 **_username** 名称，密码必须有 **_password** 名称。

> 实际上，这一切都可以在 **form_login** 键下配置。详见 [Form Login Configuration](http://symfony.com/doc/current/reference/configuration/security.html#reference-security-firewall-form-login)

> 当前的登录表单不能抵御 **CSRF 攻击**。阅读 [Using CSRF Protection in the Login Form](http://symfony.com/doc/current/cookbook/security/csrf_in_login_form.html) 了解如何保护您的登录表单。

就是这样！当您提交表单时，安全系统会自动检查用户凭据，然后验证该用户或将用户发送回显示错误信息的登录表单页面。

下面来回顾一下整个过程：

1. 用户试图访问受保护的资源；
2. 防火墙通过将用户重定向到登录表单 (**/login**) 来启动身份检验过程；
3. **/login** 页面通过本例中创建的路径和控制器来展现登录表单；
4. 用户将登录表单提交到 **/login_check**；
5. 安全系统将拦截该请求，检查用户提交的凭据，如果他们是正确的，用户进行身份验证，如果他们不正确，将用户重定向回登录表单。

## 在成功后重定向

如果提交凭据正确，用户将被重定向到请求的原始页面（例如 **/admin/foo**）。如果最初用户直接登录页面，他们就会被重定向到主页。这都是自定义的，例如，允许您将用户重定向到特定的 URL。

有关这方面的更多细节以及一般如何自定义表单登录过程，详见 [How to Customize your Form Login](http://symfony.com/doc/current/cookbook/security/form_login.html)

## 避免常见的陷阱

在设置您的登录表单时，注意一些常见的陷阱。

### 1. 创建正确的路径

首先，请确保您已经正确地定义了 **/login** 和 **/login\_check** 的路径，并且它们正确地对应于 **login_path** 和 **check_path** 的配置值。这里配置错误可能意味着您被重定向至 404 页，而不是登录页面，或提交登录表单时不执行任何操作（只是一遍又一遍地看到登录表单）。

### 2. 确保登录页面不安全（重定向至循环）

此外，确保登录页面可由匿名用户访问。例如，下面的配置-它请求 **ROLE_ADMIN** 角色的所有 URL(包括 **/login** URL)，将导致循环重定向。

```YAML
# app/config/security.yml

# ...
access_control:
    - { path: ^/, roles: ROLE_ADMIN }
```

```XML
<!-- app/config/security.xml -->

<!-- ... -->
<access-control>
    <rule path="^/" role="ROLE_ADMIN" />
</access-control>
```

```PHP
// app/config/security.php

// ...
'access_control' => array(
    array('path' => '^/', 'role' => 'ROLE_ADMIN'),
),
```

通过添加匹配 /login/* 的访问控制，不请求身份验证来修复此问题：

```YAML
# app/config/security.yml

# ...
access_control:
    - { path: ^/login, roles: IS_AUTHENTICATED_ANONYMOUSLY }
    - { path: ^/, roles: ROLE_ADMIN }
```

```XML
<!-- app/config/security.xml -->

<!-- ... -->
<access-control>
    <rule path="^/login" role="IS_AUTHENTICATED_ANONYMOUSLY" />
    <rule path="^/" role="ROLE_ADMIN" />
</access-control>
```

```PHP
// app/config/security.php

// ...
'access_control' => array(
    array('path' => '^/login', 'role' => 'IS_AUTHENTICATED_ANONYMOUSLY'),
    array('path' => '^/', 'role' => 'ROLE_ADMIN'),
),
```

此外，如果您的防火墙不允许匿名用户(没有 **anonymous** 键)，您需要为登录页创建一个特殊的防火墙来允许匿名用户：

```YAML
# app/config/security.yml

# ...
firewalls:
    # order matters! This must be before the ^/ firewall
    login_firewall:
        pattern:   ^/login$
        anonymous: ~
    secured_area:
        pattern:    ^/
        form_login: ~
```

```XML
<!-- app/config/security.xml -->

<!-- ... -->
<firewall name="login_firewall" pattern="^/login$">
    <anonymous />
</firewall>
<firewall name="secured_area" pattern="^/">
    <form-login />
</firewall>
```

```PHP
// app/config/security.php

// ...
'firewalls' => array(
    'login_firewall' => array(
        'pattern'   => '^/login$',
        'anonymous' => array(),
    ),
    'secured_area' => array(
        'pattern'    => '^/',
        'form_login' => array(),
    ),
),
```

### 3. 确保 /login_check 在防火墙后面

接下来，确保您的 **check\_path** URL (例如 **/login\_check**)是在您正在使用的表单登录的防火墙后面(在本例中，单个防火墙匹配所有 URL，包括 **/login\_check**)。如果 **/login\_check** 不匹配任何防火墙，您会收到一个 **Unable to find the controller for path "/login\_check"** 的异常。

### 4. 多个防火墙不共享相同的安全环境

如果您正在使用多个防火墙并且您对一个防火墙进行身份验证，您将不会自动对任何其它防火墙进行身份验证。不同的防火墙就像不同的安全系统。为此，您必须为不同的防火墙显式指定相同的 [Firewall Context](http://symfony.com/doc/current/reference/configuration/security.html#reference-security-firewall-context)。但是通常对于大多数应用程序来说，有一个主要的防火墙就足够了。

### 5. 路径错误页是不被防火墙覆盖的

由于路径是在安全性之前被确定，404 错误页面不受任何防火墙控制。这意味着您不能做安全检查甚至访问这些页面上的用户对象。有关详细信息，请参阅 [How to Customize Error Pages](http://symfony.com/doc/current/cookbook/controller/error_pages.html)。
