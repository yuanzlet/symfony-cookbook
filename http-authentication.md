# 如何在功能测试中模拟 HTTP 认证

如果您的应用程序需要 HTTP 认证，跳过服务器变量的用户名和密码来生成客户端 **createClient()**：

```
$client = static::createClient(array(), array(
    'PHP_AUTH_USER' => 'username',
    'PHP_AUTH_PW'   => 'pa$$word',
));
```

您也可以在每一个请求的基础数据上重写它：

```
$client->request('DELETE', '/post/12', array(), array(), array(
    'PHP_AUTH_USER' => 'username',
    'PHP_AUTH_PW'   => 'pa$$word',
));
```

当您的应用程序使用 **form_login** 时，您可以通过允许您的测试配置使用 HTTP 认证来简化您的测试。您可以使用以上的代码来在测试中进行认证，但是仍然使您的用户通过通常的 **form_login** 登陆。诀窍是在您的防火墙添加 **http_basic** 键（http_basic key），连同 **form_login** 键一起：

```YAML
# app/config/config_test.yml
security:
    firewalls:
        your_firewall_name:
            http_basic: ~
```

```XML
<!-- app/config/config_test.xml -->
<security:config>
    <security:firewall name="your_firewall_name">
      <security:http-basic />
   </security:firewall>
</security:config>
```

```PHP
// app/config/config_test.php
$container->loadFromExtension('security', array(
    'firewalls' => array(
        'your_firewall_name' => array(
            'http_basic' => array(),
        ),
    ),
));
```
