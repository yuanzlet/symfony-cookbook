# 如何限定防火墙使其只允许通过指定请求

通过安全组件，你可以配置一个能通过某些请求选项的防火墙。大多数情况下，仅通过URL匹配

就可实现要求了，但某些特殊情况下，可通过进一步限定防火墙，使其禁止不满足要求的请求

通过，达到同样的目的。

> 你可以使用任何这些限制,单独或混合在一起以得到您想要的防火墙配置。

## 通过模式限定

这也是默认的限定方式，并且仅当请求URL与限定配置模式串相匹配时，才初始化有限定的防火

墙。
 
- YAML
```
# app/config/security.yml  
# ...
security:
    firewalls:
        secured_area:
            pattern: ^/admin
            # ...
```

- XML
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
        <firewall name="secured_area" pattern="^/admin">
            <!-- ... -->
        </firewall>
    </config>
</srv:container>
```

- PHP
```
// app/config/security.php

// ...
$container->loadFromExtension('security', array(
    'firewalls' => array(
        'secured_area' => array(
            'pattern' => '^/admin',
            // ...
        ),
    ),
));
```

模式是正则表达式。在本例中，仅当 URL 以 /admin 作为开始（以 ^ 作为正则开始符）时，

防火墙会被激活。如果URL与该模式 不匹配，防火墙不会被激活，因此，防火墙就有可能成功

接收请求。
  
## 通过主机限定

如果仅通过模式匹配限定不可满足要求，也可使用请求与主机匹配的方法。当配置选项主机确

立后，可以限定防火墙，使它在仅当来自某请求的主机能匹配上配置时，才开始初始化。

- YAML
```
# app/config/security.yml  

# ...
security:
    firewalls:
        secured_area:
            host: ^admin\.example\.com$
            # ...
```

- XML
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
        <firewall name="secured_area" host="^admin\.example\.com$">
            <!-- ... -->
        </firewall>
    </config>
</srv:container>
```

- PHP
```
// app/config/security.php

// ...
$container->loadFromExtension('security', array(
    'firewalls' => array(
        'secured_area' => array(
            'host' => '^admin\.example\.com$',
            // ...
        ),
    ),
));
```

主机（如上述的模式一般）是一正则表达式。在本例中，仅当主机完全与 hostname 相等（以 

^ 和 $ 决定开头结尾）时，防火墙被激活。而如果名字与该模式不匹配，防火墙不会被激活，

因此，防火墙就有可能把成功接收请求。
  
## 通过HTTP方法限定

该配置选项策略会把防火墙允许通过的 HTTP 方法限定在指令的方法里。

- YAML
```
# app/config/security.yml

# ...
security:
    firewalls:
        secured_area:
            methods: [GET, POST]
            # ...
```

- XML
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
        <firewall name="secured_area" methods="GET,POST">
            <!-- ... -->
        </firewall>
    </config>
</srv:container>
```

- PHP
```
// app/config/security.php

// ...
$container->loadFromExtension('security', array(
    'firewalls' => array(
        'secured_area' => array(
            'methods' => array('GET', 'POST'),
            // ...
        ),
    ),
));
```

在本例中，仅当请求 HTTP 方法为 GET 或 POST 时，防火墙被激活。如果请求的方法不属于指

定方法列表中，则防火墙不会被激活，有可能成功接收请求。
