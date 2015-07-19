# 切换分析器存储


默认情况下，配置文件将收集的数据存储在缓存目录中的文件中。你可以通过 dsn ，用户名，密码和寿命的选项来控制存储。例如，下面的配置使用 MySQL 作为分析器的生命周期为一小时的存储方式：  
YAML:
```
# app/config/config.yml
framework:
    profiler:
        dsn:      "mysql:host=localhost;dbname=%database_name%"
        username: "%database_user%"
        password: "%database_password%"
        lifetime: 3600
```

XML:
```
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:framework="http://symfony.com/schema/dic/symfony"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/symfony
        http://symfony.com/schema/dic/symfony/symfony-1.0.xsd"
>
    <framework:config>
        <framework:profiler
            dsn="mysql:host=localhost;dbname=%database_name%"
            username="%database_user%"
            password="%database_password%"
            lifetime="3600"
        />
    </framework:config>
</container>

```

PHP:
```
// app/config/config.php

// ...
$container->loadFromExtension('framework', array(
    'profiler' => array(
        'dsn'      => 'mysql:host=localhost;dbname=%database_name%',
        'username' => '%database_user',
        'password' => '%database_password%',
        'lifetime' => 3600,
    ),
));
```

[HttpKernel 组件](http://symfony.com/doc/current/components/http_kernel/introduction.html)目前支持以下几种分析器器存储驱动程序：  

- 文件
- sqlite
- mysql
- mongodb
- memcache
- memcached
- redis  

这项工作的许可为 Creative Commons Attribution-Share Alike 3.0 Unported [License](http://creativecommons.org/licenses/by-sa/3.0/)

