# 如何注册一个新的请求格式和 Mime 类型  
每个请求都有一个“格式”（如 HTML，JSON），这是用来确定在响应中返回什么类型的内容。事实上，请求格式，可以通过 [getRequestFormat()](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/Request.html#getRequestFormat()) 来看到的，是用于设置在响应对象中内容类型头的 MIME 类型的。在内部，Symfony 包含一个包含了最常见的格式（例如 HTML，JSON）及其相关的 MIME 类型（例如 text/html, application/json）的地图。当然，其它格式的MIME类型的条目可以很容易地被添加。本文档将显示如何添加 jsonp 格式和相应的 MIME 类型。  

## 配置你的新格式
FrameworkBundle 注册一个用户，来添加传入的请求的格式。  

你需要做的所有事情就是配置 jsonp 格式： 

YAML:
```
# app/config/config.yml
framework:
    request:
        formats:
            jsonp: 'application/javascript'
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
        <framework:request>
            <framework:format name="jsonp">
                <framework:mime-type>application/javascript</framework:mime-type>
            </framework:format>
        </framework:request>
    </framework:config>
</container>

```

PHP:
```
// app/config/config.php
$container->loadFromExtension('framework', array(
    'request' => array(
        'formats' => array(
            'jsonp' => 'application/javascript',
        ),
    ),
));

```

> 您也可以将多个 MIME 类型与一个格式关联，但是请注意，首选项必须是它将作为内容类型的：  

YAML:
```
# app/config/config.yml
framework:
    request:
        formats:
            csv: ['text/csv', 'text/plain']
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
        <framework:request>
            <framework:format name="csv">
                <framework:mime-type>text/csv</framework:mime-type>
                <framework:mime-type>text/plain</framework:mime-type>
            </framework:format>
        </framework:request>
    </framework:config>
</container>

```

PHP:
```
// app/config/config.php
$container->loadFromExtension('framework', array(
    'request' => array(
        'formats' => array(
            'jsonp' => array(
                'text/csv',
                'text/plain',
            ),
        ),
    ),
));

```  

这项工作的许可为 Creative Commons Attribution-Share Alike 3.0 Unported [License](http://creativecommons.org/licenses/by-sa/3.0/)
