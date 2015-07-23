# 安全访问控制是如何工作的

对于每个传入的请求，Symfony 都检查每个 **access_control** 条目来找到一个匹配当前项的请求的 **access_control**。当它找到一个匹配的 **access_control** 项它便停止，只有第一个匹配的 **access_control** 用于强制执行访问。

每个 **access_control** 拥有几个可以配置以下两个不同条件的选项：

1\. [传入的请求应匹配此访问控制项](http://symfony.com/doc/current/cookbook/security/access_control.html#security-book-access-control-matching-options)

2\. [一旦它匹配成功，应实施某种形式的访问限制](http://symfony.com/doc/current/cookbook/security/access_control.html#security-book-access-control-enforcement-options)

## 1\. 匹配选项

Symfony 中每个 [access_control](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/RequestMatcher.html) 条目创建了一个 **RequestMatcher** 用于确定是否应该对每个访问使用给定的访问控制。下面的 **access_control** 选项用于匹配：

- **路径**

- **ip** 或 **ips**

- **主机**

- **方法**

以下面的 **access_control** 条目为例：

YAML:

```
# app/config/security.yml
security:
    # ...
    access_control:
        - { path: ^/admin, roles: ROLE_USER_IP, ip: 127.0.0.1 }
        - { path: ^/admin, roles: ROLE_USER_HOST, host: symfony\.com$ }
        - { path: ^/admin, roles: ROLE_USER_METHOD, methods: [POST, PUT] }
        - { path: ^/admin, roles: ROLE_USER }
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
        <access-control>
            <rule path="^/admin" role="ROLE_USER_IP" ip="127.0.0.1" />
            <rule path="^/admin" role="ROLE_USER_HOST" host="symfony\.com$" />
            <rule path="^/admin" role="ROLE_USER_METHOD" method="POST, PUT" />
            <rule path="^/admin" role="ROLE_USER" />
        </access-control>
    </config>
</srv:container>
```

PHP:

```
// app/config/security.php
$container->loadFromExtension('security', array(
    // ...
    'access_control' => array(
        array(
            'path' => '^/admin',
            'role' => 'ROLE_USER_IP',
            'ip' => '127.0.0.1',
        ),
        array(
            'path' => '^/admin',
            'role' => 'ROLE_USER_HOST',
            'host' => 'symfony\.com$',
        ),
        array(
            'path' => '^/admin',
            'role' => 'ROLE_USER_METHOD',
            'method' => 'POST, PUT',
        ),
        array(
            'path' => '^/admin',
            'role' => 'ROLE_USER',
        ),
    ),
));
```

对于每个传入的请求，基于 URI、 客户端的 IP 地址、 传入的主机名和请求的方法 Symfony 会决定使用哪个 **access_control** 。请记住，第一个匹配规则已经被使用了，并且如果 **ip**、 **主机**或**方法**没有被指定为一个条目，该 **access_control** 将匹配任何的 **ip**、 **主机**或**方法**：

<table  border="1">
<colgroup>
<col width="11%">
<col width="9%">
<col width="9%">
<col width="8%">
<col width="22%">
<col width="41%">
</colgroup>
<thead valign="bottom">
<tr><th><strong>URI</strong></th>
<th><strong>IP</strong></th>
<th><strong>主机</strong></th>
<th><strong>方法</strong></th>
<th><strong>access_control</strong></th>
<th><strong>为什么</strong></th>
</tr>
</thead>
<tbody valign="top">
<tr><td><strong>/admin/user</strong></td>
<td>127.0.0.1</td>
<td>example.com</td>
<td>GET</td>
<td>rule #1 (<strong>ROLE_USER_IP</strong>)</td>
<td>URI 匹配<strong>路径</strong> IP 匹配 <strong>ip</strong>。</td>
</tr>
<tr><td><strong>/admin/user</strong></td>
<td>127.0.0.1</td>
<td>symfony.com</td>
<td>GET</td>
<td>rule #1 (<strong>ROLE_USER_IP</strong>)</td>
<td><strong>路径</strong>和 <strong>ip</strong> 仍然匹配。这也符合 <strong>ROLE_USER_HOST</strong> 条目，但只有第一个 <strong>access_control</strong> 匹配被使用了。.</td>
</tr>
<tr><td><strong>/admin/user</strong></td>
<td>168.0.0.1</td>
<td>symfony.com</td>
<td>GET</td>
<td>rule #2 (<strong>ROLE_USER_HOST</strong>)</td>
<td><strong>Ip</strong> 不符的第一个规则，因此使用第二个规则 (符合)。</td>
</tr>
<tr><td><strong>/admin/user</strong></td>
<td>168.0.0.1</td>
<td>symfony.com</td>
<td>POST</td>
<td>rule #2 (<strong>ROLE_USER_HOST</strong>)</td>
<td>第二条规则仍然匹配。这也符合第三条规则 (<strong>ROLE_USER_METHOD</strong>)，但只有第一个匹配的 <strong>access_control</strong> 被使用了。</td>
</tr>
<tr><td><strong>/admin/user</strong></td>
<td>168.0.0.1</td>
<td>example.com</td>
<td>POST</td>
<td>rule #3 (<strong>ROLE_USER_METHOD</strong>)</td>
<td><strong>Ip</strong> 和<strong>主机</strong>与前两项不匹配，但和第三项 - <strong>ROLE_USER_METHOD</strong> - 匹配，并且它会被使用。</td>
</tr>
<tr><td><strong>/admin/user</strong></td>
<td>168.0.0.1</td>
<td>example.com</td>
<td>GET</td>
<td>rule #4 (<strong>ROLE_USER</strong>)</td>
<td><strong>Ip</strong>、 <strong>主机</strong>和<strong>方法</strong>阻止的前三项进行匹配。但因为 URI 匹配 <strong>ROLE_USER</strong> 条目的<strong>路径</strong>模式，所以它会被使用。</td>
</tr>
<tr><td><strong>/foo</strong></td>
<td>127.0.0.1</td>
<td>symfony.com</td>
<td>POST</td>
<td>matches no entries</td>
<td>它不匹配任何 <strong>access_control</strong> 规则，因为它的 URI 不匹配任何的<strong>路径</strong>值。</td>
</tr>
</tbody>
</table>

## 2\.进入执行

一旦 Symfony 决定了哪些 **access_control** 条目能匹配 (如果有的话)，那么它将执行基于角色的访问限制，**allow_if** 和 **requires_channel** 选项:

- **role** 如果用户没有所给定的角色，那么访问将会被拒绝 (在内部，将会抛出 [AccessDeniedException](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Exception/AccessDeniedException.html) 异常);

- **allow_if** 如果该表达式返回 false，那么访问将会被拒绝;

- **requires_channel** 如果传入请求通道 (例如 **http**) 和该值不匹配 (例如 **https**)，那么用户将被重定向 (例如从 **http** 重定向到 **https**，反之亦然)。

> 如果访问被拒绝，如果一个用户还没有经过身份认证，那么系统将尝试对用户进行身份验证 (如：把用户重定向到登录页)。如果用户已经登录，则将显示 403 "拒绝访问"错误页。请参阅[如何自定义错误页](http://symfony.com/doc/current/cookbook/controller/error_pages.html)的详细信息。

## 使用 IP 来匹配 access_control

当您需要只和某些 IP 地址或范围请求所匹配的 **access_control** 项，那么会出现某些特定的情况。例如，可以使用它来拒绝访问除了那些从受信任的内部服务器之外的所有请求的 URL 模式。

> 你会看到在下面的例子解释，**ips** 选项并不限制到特定的 IP 地址。相反，使用 **ips** 密钥意味着 **access_control** 条目将仅匹配此 IP 地址，并且用户会根据 **access_control** 列表从不同的 IP 地址依次访问它。

这里是关于您如何配置一些示例 **/internal*** URL 模式的示例，所以它只能从本地服务器的请求来访问：

YAML:

```
# app/config/security.yml
security:
    # ...
    access_control:
        #
        - { path: ^/internal, roles: IS_AUTHENTICATED_ANONYMOUSLY, ips: [127.0.0.1, ::1] }
        - { path: ^/internal, roles: ROLE_NO_ACCESS }
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
        <access-control>
            <rule path="^/esi" role="IS_AUTHENTICATED_ANONYMOUSLY"
                ips="127.0.0.1, ::1" />
            <rule path="^/esi" role="ROLE_NO_ACCESS" />
        </access-control>
    </config>
</srv:container>
```

PHP:

```
// app/config/security.php
$container->loadFromExtension('security', array(
    // ...
    'access_control' => array(
        array(
            'path' => '^/esi',
            'role' => 'IS_AUTHENTICATED_ANONYMOUSLY',
            'ips' => '127.0.0.1, ::1'
        ),
        array(
            'path' => '^/esi',
            'role' => 'ROLE_NO_ACCESS'
        ),
    ),
));
```

这里描述了当路径是 **/internal/something** 并且来自外部 IP 地址 **10.0.0.1** 的时候该如何去工作：

- 第一个访问控制规则将被忽略，因为虽然路径匹配但是 IP 地址不匹配，或者 IPs 列出的地址不匹配；

- 第二个访问控制规则是已经被启用的 (这里唯一的限制是路径)，所以它匹配。如果你确保没有用户具有 ROLE_NO_ACCESS，那么访问就会被拒绝 (ROLE_NO_ACCESS 可以与任何现有的角色不匹配，它只是用来始终拒绝访问的访问)。

但如果同一请求来自 **127.0.0.1** 或**:: 1** (IPv6 环回地址):

- 现在，作为**路径**和 **ip** 匹配启用的第一次的访问控制规则: 允许访问的用户总是有 **IS_AUTHENTICATED_ANONYMOUSLY** 的作用。

- 如果第一个规则成功匹配，那么第二访问规则则不会审查。

## 通过表达式来保护

一旦 **access_control** 条目相匹配，你可以通过角色秘钥或使用更复杂的逻辑与表达式中 **allow_if** 密钥拒绝访问：

YAML:

```
# app/config/security.yml
security:
    # ...
    access_control:
        -
            path: ^/_internal/secure
            allow_if: "'127.0.0.1' == request.getClientIp() or has_role('ROLE_ADMIN')"
```

XML:

```
<access-control>
    <rule path="^/_internal/secure"
        allow-if="'127.0.0.1' == request.getClientIp() or has_role('ROLE_ADMIN')" />
</access-control>
```

PHP:

```
'access_control' => array(
    array(
        'path' => '^/_internal/secure',
        'allow_if' => '"127.0.0.1" == request.getClientIp() or has_role("ROLE_ADMIN")',
    ),
),
```

在这种情况下，当用户试图从 **/_internal/secure** 开始访问任何 URL，如果 **IP** 地址是 **127.0.0.1**，或者如果用户拥有  **ROLE_ADMIN** 的角色，他们将只被授予访问权限。

在这个表达式中，您可以访问不同的变量和函数并且包括 Symfony 中的[请求](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/Request.html)对象 (见[请求](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/Request.html))。

关于其他的函数和变量的列表，请参阅[函数和变量](http://symfony.com/doc/current/cookbook/expression/expressions.html#book-security-expression-variables)。

## 迫使通道 (http,https)

您还可以要求用户通过 SSL 访问 URL；只需要使用任何 **access_control** 条目中的 **requires_channel** 参数。如果该 **access_control** 匹配成功并且请求使用的是 **http** 通道，用户将被重定向到 **https**：

YAML:

```
# app/config/security.yml
security:
    # ...
    access_control:
        - { path: ^/cart/checkout, roles: IS_AUTHENTICATED_ANONYMOUSLY, requires_channel: https }
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

    <access-control>
        <rule path="^/cart/checkout"
            role="IS_AUTHENTICATED_ANONYMOUSLY"
            requires-channel="https" />
    </access-control>
</srv:container>
```

PHP:

```
// app/config/security.php
$container->loadFromExtension('security', array(
    'access_control' => array(
        array(
            'path' => '^/cart/checkout',
            'role' => 'IS_AUTHENTICATED_ANONYMOUSLY',
            'requires_channel' => 'https',
        ),
    ),
));
```
