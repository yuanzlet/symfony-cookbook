# 如何配置 Monolog 从日志中排除 404 错误

有时候你的日志充满了不想看到的 404 HTTP 错误，举例来说，当攻击者扫描你的应用的一些知名的应用程序路径时（例如 /phpmyadmin）。当使用 **fingers_crossed** handler 时，你可以基于一个在 MonologBundle 配置中的正常的解释来拒绝记录这些日志：  

YAML:  

```
# app/config/config.yml
monolog:
    handlers:
        main:
            # ...
            type: fingers_crossed
            handler: ...
            excluded_404s:
                - ^/phpmyadmin
```

XML:  

```
<!-- app/config/config.xml -->
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:monolog="http://symfony.com/schema/dic/monolog"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/monolog
        http://symfony.com/schema/dic/monolog/monolog-1.0.xsd"
>
    <monolog:config>
        <monolog:handler type="fingers_crossed" name="main" handler="...">
            <!-- ... -->
            <monolog:excluded-404>^/phpmyadmin</monolog:excluded-404>
        </monolog:handler>
    </monolog:config>
</container>
```

PHP:  

```
// app/config/config.php
$container->loadFromExtension('monolog', array(
    'handlers' => array(
        'main' => array(
            // ...
            'type'          => 'fingers_crossed',
            'handler'       => ...,
            'excluded_404s' => array(
                '^/phpmyadmin',
            ),
        ),
    ),
));
```


