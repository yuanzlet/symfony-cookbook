# 如何添加“记住我”登录功能

 一旦经过身份验证的用户,他们的凭证通常存储在会话中。这意味着会话结束时他们将被记录,并必须提供他们的登录细节及下次他们要访问的应用程序。你可以允许用户选择登录停留的时间比使用 cookie 会话持续的时间长，这可以通过使用 **remember_me** 防火墙选项来实现:
 
 YAML:
 
 ```
 # app/config/security.yml
firewalls:
    default:
        # ...
        remember_me:
            key:      "%secret%"
            lifetime: 604800 # 1 week in seconds
            path:     /
            # by default, the feature is enabled by checking a
            # checkbox in the login form (see below), uncomment the
            # below lines to always enable it.
            #always_remember_me: true
 ```
 
 XML:
 
 ```
 <!-- app/config/security.xml -->
<?xml version="1.0" encoding="utf-8" ?>
<srv:container xmlns="http://symfony.com/schema/dic/security"
    xmlns:srv="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">

    <config>
        <firewall name="default">
            <!-- ... -->

            <!-- by default, the feature is enabled by checking a checkbox
                 in the login form (see below), add always-remember-me="true"
                 to always enable it. -->
            <remember-me
                key      = "%secret%"
                lifetime = "604800" <!-- 1 week in seconds -->
                path     = "/"
            />
        </firewall>
    </config>
</srv:container>
 ```
 
 PHP:
 
 ```
 // app/config/security.php
$container->loadFromExtension('security', array(
    'firewalls' => array(
        'default' => array(
            // ...
            'remember_me' => array(
                'key'      => '%secret%',
                'lifetime' => 604800, // 1 week in seconds
                'path'     => '/',
                // by default, the feature is enabled by checking a
                // checkbox in the login form (see below), uncomment
                // the below lines to always enable it.
                //'always_remember_me' => true,
            ),
        ),
    ),
));
 ```

**remember_me** (下次自动登录）防火墙定义以下配置选项: 　　

**key** (必需)：

用于加密 cookie 值的内容。通常使用的**秘密**值定义在**app/config/parameters.yml** 文件中。　　

**name** (默认值:**REMEMBERME**)：

cookie 的名称用来保持用户登录。如果你在同一应用程序的多个防火墙启用 **remember_me** 功能,确保为每个防火墙的 cookie 选择一个不同的名称。否则,你将面临很多安全相关问题。

**Lifetime** (默认值:**31536000**)：

用户将持续登录的秒数。默认用户登录一年。

**path** (默认值:**/**)：

与这种特性相关联的cookie 的路径将被使用。默认情况下 cookie 将适用于整个网站，但你也可以将它限制到一个特定的部分(例如 **/forum**，**/admin**)。

**domin** (默认值:**null**)：

使用与此特性相关的 **cookie** 的域。默认情况下 cookies 使用当的前域是从 **$ _SERVER** 获得。
   
**secure** (默认值:**false**)：
 
如果该值为**真**,与此功能相关的 cookie 将被通过 HTTPS 安全连接发送给用户。

**hhttponly** (默认值:**true**)：
 
如果该值为**真**,这个特性相关的 cookie 只能通过 HTTP 协议。这意味着 cookie 不会访问脚本语言,比如 JavaScript。　
   
**remember_me_parameter** (默认值:**_remember_me**)：
   
表单字段的名称检查决定是否应该启用“记住我”功能。继续阅读这篇文章，你将知道如何附有条件地启用这个特性。　
   
**always_remember_me**(默认值:**false**) ：

如果该值为**真**,**remember_me_parameter** 的值将被忽略,“记住我”功能总是启用,不管最终用户的需求。　　
   
**token_provider** (默认值:**null**)：

定义一个 token provider 的服务的id以供使用。默认情况下,令牌存储在一个cookie中。例如,您可能想要令牌存储在一个数据库中,但在 cookie 中没有一个(散列的)版本的密码。

DoctrineBridge附带一个 **Symfony\Bridge\Doctrine\Security\RememberMe\DoctrineTokenProvider**,您可以使用。

## 强制用户退出下次自动登录的特性

为用户提供选择使用或不使用记住我的功能是个好主意,因为使用或不使用记住我并不总是合适的。通常这样做的方法是添加一个复选框登录表单。通过将复选框命名为 **_remember_me**(或您使用 **remember_me_parameter **配置的名称),当复选框被选中并且用户成功登录时,cookie 会被自动设置。因此,特定的登录表单最终可能看起来像这样:

Twig:

```
{# app/Resources/views/security/login.html.twig #}
{% if error %}
    <div>{{ error.message }}</div>
{% endif %}

<form action="{{ path('login_check') }}" method="post">
    <label for="username">Username:</label>
    <input type="text" id="username" name="_username" value="{{ last_username }}" />

    <label for="password">Password:</label>
    <input type="password" id="password" name="_password" />

    <input type="checkbox" id="remember_me" name="_remember_me" checked />
    <label for="remember_me">Keep me logged in</label>

    <input type="submit" name="login" />
</form>
```

PHP:

```
<!-- app/Resources/views/security/login.html.php -->
<?php if ($error): ?>
    <div><?php echo $error->getMessage() ?></div>
<?php endif ?>

<form action="<?php echo $view['router']->generate('login_check') ?>" method="post">
    <label for="username">Username:</label>
    <input type="text" id="username"
           name="_username" value="<?php echo $last_username ?>" />

    <label for="password">Password:</label>
    <input type="password" id="password" name="_password" />

    <input type="checkbox" id="remember_me" name="_remember_me" checked />
    <label for="remember_me">Keep me logged in</label>

    <input type="submit" name="login" />
</form>
```

在之后的访问中，用户将自动登录，同时cookie仍然是有效的。　　　

## 强制用户在访问某些资源之前认证

当用户返回到您的站点时,他们自动验证存储在 **cookie remember me** 的信息。这允许用户访问受保护的资源,只要用户实际上通过了该网站的身份验证。

然而,在某些情况下,您可能想要强制用户实际认证之前访问某些资源。例如,您可能允许“记住我”用户看到基本账户信息,然后要求他们实际上认证之前修改这些信息。

安全组件提供了一种简单的方法来做到这一点。增加了明确分配给他们的角色，用户会根据他们的验证方式自动地被分配为以下角色之一: 

**IS_AUTHENTICATED_ANONYMOUSLY**
  
自动分配给在防火墙下受部分站点保护但实际上没有登录的用户。这是匿名访问的唯一方式。

**IS_AUTHENTICATED_REMEMBERED**

自动分配给通过 remember me cookie 验证的用户。　

**IS_AUTHENTICATED_FULLY**

自动分配给在当前会话中提供了他们的登录细节的用户。

除了显式分配角色，您可以使用这些来进行访问控制。

> 如果你有 **IS_AUTHENTICATED_REMEMBERED** 角色,那么你也有 **IS_AUTHENTICATED_ANONYMOUSLY** 角色。如果你有 **IS_AUTHENTICATED_FULLY** 角色,那么你还有另外两个角色。换句> 话说,这些角色代表三个级别的增加“强度”的认证。　

您可以使用这些额外的角色更细粒度地控制访问一个网站的部分内容。例如,当通过 cookie 验证的时候，您可能想让你的用户能够在 **/account** 查看自己的账户，但必须提供他们的登录细节才能编辑账户细节。为此,您可以使用这些角色获得特定的控制器操作。编辑操作的控制器可以通过使用服务环境来获得。

在接下来的例子中,活动只在用户有 **IS_AUTHENTICATED_FULLY** 角色时允许。

```
// ...
use Symfony\Component\Security\Core\Exception\AccessDeniedException

// ...
public function editAction()
{
    $this->denyAccessUnlessGranted('IS_AUTHENTICATED_FULLY');

    // ...
}
```

如果您的应用程序是基于Symfony标准版,你也可以通过使用注释获得你的控制器:

```
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Security;

/**
 * @Security("has_role('IS_AUTHENTICATED_FULLY')")
 */
public function editAction($name)
{
    // ...
}
```

> 如果你也有一个访问控制安全配置,需要用户有一个 **ROLE_USER** 角色以便访问任何帐户,那么就会有以下情况: 

>- 如果一个没有进行身份验证或匿名身份验证的用户试图访问帐户,用户将被要求进行身份验证。

>- 一旦用户已经输入了他们的用户名和密码,假定对于你的每个配置，用户获得 **ROLE_USER** 角色，用户将会拥有 **IS_AUTHENTICATED_FULLY** 角色并能够访问帐户中的任何页面部分,>- 包括 **editAction** 控制器。　　

>- 如果用户的会话结束,当用户返回到网站,他们将能够访问每个账户页面——除了编辑页面——但要求是没有要求强制认证的。然而,当他们试图访问 **editAction** 控制器时,他们将被迫

>- 进行认证,因为他们都尚未充分认证。


如果您想了解更多以这种方式获得服务或方法的信息,请看[如何获得任何服务或您的应用程序中的方法](http://symfony.com/doc/current/cookbook/security/securing_services.html)。

   
