# 如何记录消息到不同的文件

Symfony 框架将日志消息组织到频道当中。默认情况下，这里有几个频道，包括 **doctrine**, **event**, **security**, **request** 和其他的一些。频道被印在了日志消息之中并且也能够被用来指导不同的频道到不同的地方、文件。  

默认情况下，Symfony 记录每一条进入单一文件的消息（不管频道是什么）。  

> 每一个频道和容器（使用 **container:debug** 命令来查看整个列表）中的日志服务（**monolog.logger.XXX**）相对应并且这些被注入到不同的服务中。  

## 切换频道到不同的 Handler 中

现在，假设你想要记录 **security** 频道的日志到一个不同的文件中。为了完成这个，创建一个新的 handler 并且将它配置成只记录来自于 **security** 频道的日志。你可以在在所有环境下的 **config.yml** 中添加这个来记录日志，或者只是在 **prod** 中发生 **config_prod.yml**：  

YAML:  

```
# app/config/config.yml
monolog:
    handlers:
        security:
            # log all messages (since debug is the lowest level)
            level:    debug
            type:     stream
            path:     "%kernel.logs_dir%/security.log"
            channels: [security]

        # an example of *not* logging security channel messages for this handler
        main:
            # ...
            # channels: ["!security"]
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
        <monolog:handler name="security" type="stream" path="%kernel.logs_dir%/security.log">
            <monolog:channels>
                <monolog:channel>security</monolog:channel>
            </monolog:channels>
        </monolog:handler>

        <monolog:handler name="main" type="stream" path="%kernel.logs_dir%/main.log">
            <!-- ... -->
            <monolog:channels>
                <monolog:channel>!security</monolog:channel>
            </monolog:channels>
        </monolog:handler>
    </monolog:config>
</container>
```

PHP:  

```
// app/config/config.php
$container->loadFromExtension('monolog', array(
    'handlers' => array(
        'security' => array(
            'type'     => 'stream',
            'path'     => '%kernel.logs_dir%/security.log',
            'channels' => array(
                'security',
            ),
        ),
        'main'     => array(
            // ...
            'channels' => array(
                '!security',
            ),
        ),
    ),
));
```

## YAML 说明书

你可以通过多种形式指定配置：  

```
channels: ~    # Include all the channels

channels: foo  # Include only channel "foo"
channels: "!foo" # Include all channels, except "foo"

channels: [foo, bar]   # Include only channels "foo" and "bar"
channels: ["!foo", "!bar"] # Include all channels, except "foo" and "bar"
```

## 创建你自己的信道

你可以一次将信道日志改变到一个服务。这既不是通过下面的[配置](http://symfony.com/doc/current/cookbook/logging/channels_handlers.html#cookbook-monolog-channels-config)也不是通过给你的服务添加 [monolog.logger](http://symfony.com/doc/current/reference/dic_tags.html#dic-tags-monolog) 标签并指定这个服务应该记录哪个信道的日志来完成的。有了这个标签，注入到服务中的日志记录器早就设置好使用你所指定的信道了。  

### 不使用被标签的服务来设置附加信道

使用 MonologBundle 2.4 你可以配置附加信道而不需要给你的服务加标签：  

YAML:  

```
# app/config/config.yml
monolog:
    channels: ["foo", "bar"]
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
        <monolog:channel>foo</monolog:channel>
        <monolog:channel>bar</monolog:channel>
    </monolog:config>
</container>
```

PHP:  

```
// app/config/config.php
$container->loadFromExtension('monolog', array(
    'channels' => array(
        'foo',
        'bar',
    ),
));
```

有了这个，你现在就可以使用自动注册的日志服务 **monolog.logger.foo** 将日志信息发送到 **foo** 频道了。

## 从指导书中学习更多

- [如何使用 Monolog 记录日志](http://symfony.com/doc/current/cookbook/logging/monolog.html)  
