# 如何自定义登录表单

使用[表单登录](http://symfony.com/doc/current/cookbook/security/form_login_setup.html)来处理 Symfony 的验证是一件非常常见并且灵活的方法。几乎表单登录的每个方面都可以进行自定义，所有的默认配置都将在下一节中进行介绍。

## 表单登录参考配置

如果想要查看完整表单登录参考配置，请参考 [SecurityBundle 配置 ("安全")]( http://symfony.com/doc/current/reference/configuration/security.html)。如下列举了一些更有趣的解释选项。

## 成功登录后的重定向

当使用不同的配置选项登录成功以后，您就可以改变重定向的登录表单。在默认的情况下，表单会重定向到用户请求的 URL(即触发显示登录窗体的 URL)。比如，如果用户请求 **http://www.example.com/admin/post/18/edit**，当用户成功的登录以后，页面最终会重定向到 **http://www.example.com/admin/post/18**。这是通过把用户请求的页面存储到 session 实现的。如果 session 中没有存储一个 URL（比如用户直接访问的登录页），那么当用户成功登录以后，系统就会给用户展示默认页。你可以通过很多种方式来改变这种模式。

> 就像前面提到的，在默认情况下，显示的页面将会被重定向到用户最初请求的页面。有时候，也会出现一些问题，就像后台的 Ajax 请求“看起来”像是最后访问的 URL，导致用户访问的页面被重定向到这里，有关控制此行为的信息，请参阅[如何更改默认目标路径](http://symfony.com/doc/current/cookbook/security/target_path.html)。

## 更改默认页面

首先，默认页是可以设置的(即如果没有在 session 中存储以前的页面路径，就会将页面重定向到默认的页面)。请使用以下配置来设置 **default_security_target**（默认安全目标） 路径:

YAML：

```
# app/config/security.yml
security:
    firewalls:
        main:
            form_login:
                # ...
                default_target_path: default_security_target
```

XML：

```
<!-- app/config/security.xml -->
<config>
    <firewall>
        <form-login
            default_target_path="default_security_target"
        />
    </firewall>
</config>
```

PHP：

```
// app/config/security.php
$container->loadFromExtension('security', array(
    'firewalls' => array(
        'main' => array(
            // ...

            'form_login' => array(
                // ...
                'default_target_path' => 'default_security_target',
            ),
        ),
    ),
));
```

现在，当 session 中没有存储 URL，用户所浏览的页面将会转到 **default_security_target** 中的路径。

## 总是重定向到默认页

您可以通过设置 always_use_default_target_path 选项的值设定为真，这样的话，不管用户请求了什么 URL ，其访问的页面最终都会被重定向到默认页。

YAML:

```
# app/config/security.yml
security:
    firewalls:
        main:
            form_login:
                # ...
                always_use_default_target_path: true
```

XML:

```
<!-- app/config/security.xml -->
<config>
    <firewall>
        <form-login
            always_use_default_target_path="true"
        />
    </firewall>
</config>
```

PHP:

```
// app/config/security.php
$container->loadFromExtension('security', array(
    'firewalls' => array(
        'main' => array(
            // ...

            'form_login' => array(
                // ...
                'always_use_default_target_path' => true,
            ),
        ),
    ),
));
```

## 使用引用的 URL

为了防止以前的 URL 没有被存储在 session 中，您不妨试试改用 HTTP_REFERER 属性，因为这往往会达到相同的效果。您可以通过把 **setting use_referer** 属性的值改为 true（默认是false）：

YAML:

```
# app/config/security.yml
security:
    firewalls:
        main:
            form_login:
                # ...
                use_referer:        true
```

XML:

```
<!-- app/config/security.xml -->
<config>
    <firewall>
        <form-login
            use_referer="true"
        />
    </firewall>
</config>
```

PHP:

```
// app/config/security.php
$container->loadFromExtension('security', array(
    'firewalls' => array(
        'main' => array(
            // ...

            'form_login' => array(
                // ...
                'use_referer' => true,
            ),
        ),
    ),
));
```

## 控制重定向表单内的 URL

您还可以通过包含一个名叫 **_target_path** 的隐藏字段来重写用户通过表单本身重定向的 URL。例如，可以通过以下的程序来重定向到一些**账户**路由定义的 URL：

Twig:

```
{# src/Acme/SecurityBundle/Resources/views/Security/login.html.twig #}
{% if error %}
    <div>{{ error.message }}</div>
{% endif %}

<form action="{{ path('login_check') }}" method="post">
    <label for="username">Username:</label>
    <input type="text" id="username" name="_username" value="{{ last_username }}" />

    <label for="password">Password:</label>
    <input type="password" id="password" name="_password" />

    <input type="hidden" name="_target_path" value="account" />

    <input type="submit" name="login" />
</form>
```

PHP:

```
<!-- src/Acme/SecurityBundle/Resources/views/Security/login.html.php -->
<?php if ($error): ?>
    <div><?php echo $error->getMessage() ?></div>
<?php endif ?>

<form action="<?php echo $view['router']->generate('login_check') ?>" method="post">
    <label for="username">Username:</label>
    <input type="text" id="username" name="_username" value="<?php echo $last_username ?>" />

    <label for="password">Password:</label>
    <input type="password" id="password" name="_password" />

    <input type="hidden" name="_target_path" value="account" />

    <input type="submit" name="login" />
</form>
```

现在，用户访问页面将会被重定向到隐藏表单字段的值。这个值的属性可以是相对路径，也可以是绝对的 URL 路径，也可以是路由的名称。您甚至可以通过把 **target_path_parameter** 选项的值改为另一个值来更改隐藏表单字段的名称。

YAML:

```
# app/config/security.yml
security:
    firewalls:
        main:
            form_login:
                target_path_parameter: redirect_url
```

XML:

```
<!-- app/config/security.xml -->
<config>
    <firewall>
        <form-login
            target_path_parameter="redirect_url"
        />
    </firewall>
</config>
```

PHP:

```
// app/config/security.php
$container->loadFromExtension('security', array(
    'firewalls' => array(
        'main' => array(
            'form_login' => array(
                'target_path_parameter' => redirect_url,
            ),
        ),
    ),
));
```

## 登录失败后的重定向

除了可以将用户成功登录后的页面进行重定向，您也可以设置当用户登录失败后的重定向页面(例如用户提交了无效的用户名或密码) 。在默认情况下，用户访问的页面会重定向到登录表单页面，您也可以通过修改下面的属性来进行改变路径：

YAML:

```
# app/config/security.yml
security:
    firewalls:
        main:
            form_login:
                # ...
                failure_path: login_failure
```

XML:

```
<!-- app/config/security.xml -->
<config>
    <firewall>
        <form-login
            failure_path="login_failure"
        />
    </firewall>
</config>
```

PHP:

```
// app/config/security.php
$container->loadFromExtension('security', array(
    'firewalls' => array(
        'main' => array(
            // ...

            'form_login' => array(
                // ...
                'failure_path' => 'login_failure',
            ),
        ),
    ),
));
```
