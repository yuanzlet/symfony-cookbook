# 如何对电子邮件错误配置 Monolog

[Monolog](https://github.com/Seldaek/monolog) 可以当应用程序出现错误的时候被配置来发送电子邮件。这个配置需要一些嵌入的 handlers 来避免接收太多的电子邮件。这个配置起初看起来会很复杂但是每一个 handler 在出故障时都是很直接的。  

YAML:  

```
# app/config/config_prod.yml
monolog:
    handlers:
        mail:
            type:         fingers_crossed
            # 500 errors are logged at the critical level
            action_level: critical
            # to also log 400 level errors (but not 404's):
            # action_level: error
            # excluded_404s:
            #     - ^/
            handler:      buffered
        buffered:
            type:    buffer
            handler: swift
        swift:
            type:       swift_mailer
            from_email: error@example.com
            to_email:   error@example.com
            # or list of recipients
            # to_email:   [dev1@example.com, dev2@example.com, ...]
            subject:    An Error Occurred!
            level:      debug
```  

XML:  

```
<!-- app/config/config_prod.xml -->
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:monolog="http://symfony.com/schema/dic/monolog"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                        http://symfony.com/schema/dic/monolog http://symfony.com/schema/dic/monolog/monolog-1.0.xsd">

    <monolog:config>
        <monolog:handler
            name="mail"
            type="fingers_crossed"
            action-level="critical"
            handler="buffered"
            <!--
            To also log 400 level errors (but not 404's):
            action-level="error"
            And add this child inside this monolog:handler
            <monolog:excluded-404>^/</monolog:excluded-404>
            -->
        />
        <monolog:handler
            name="buffered"
            type="buffer"
            handler="swift"
        />
        <monolog:handler
            name="swift"
            type="swift_mailer"
            from-email="error@example.com"
            subject="An Error Occurred!"
            level="debug">

            <monolog:to-email>error@example.com</monolog:to-email>

            <!-- or multiple to-email elements -->
            <!--
            <monolog:to-email>dev1@example.com</monolog:to-email>
            <monolog:to-email>dev2@example.com</monolog:to-email>
            ...
            -->
        </monolog:handler>
    </monolog:config>
</container>
```  

PHP:  

```
// app/config/config_prod.php
$container->loadFromExtension('monolog', array(
    'handlers' => array(
        'mail' => array(
            'type'         => 'fingers_crossed',
            'action_level' => 'critical',
            // to also log 400 level errors (but not 404's):
            // 'action_level' => 'error',
            // 'excluded_404s' => array(
            //     '^/',
            // ),
            'handler'      => 'buffered',
        ),
        'buffered' => array(
            'type'    => 'buffer',
            'handler' => 'swift',
        ),
        'swift' => array(
            'type'       => 'swift_mailer',
            'from_email' => 'error@example.com',
            'to_email'   => 'error@example.com',
            // or a list of recipients
            // 'to_email'   => array('dev1@example.com', 'dev2@example.com', ...),
            'subject'    => 'An Error Occurred!',
            'level'      => 'debug',
        ),
    ),
));
```  

**邮件** handler 是一个 **fingers_crossed** 的 handler，这就意味着这只有在行为层才会被触发，在这个例子中到达了**临界**。**临界**层只会触发 5xx HTTP 代码错误。如果一旦到达这一层 **fingers_crossed** 的 handler 将会记录所有的消息且不管它们是哪一层。**handler** 设置意味着输出之后传递到 **缓冲** handler。  

> 如果你想要 400 和 500 层次错误触发电子邮件，将 **action_level** 设置成 **critical** 而不是 **error**。可以参照上面例子的代码。  

**缓冲** handler 简单地保留着请求的所有信息然后一口气将他们传递到嵌入的 handler 中。如果不使用这个 handler 那么每条消息将会分别发送电子邮件。这就是后来传递到 **swift** handler 的。这是专门处理向你发送错误电子邮件的 handler。这个的设置很直接，对于来去的地址和主题来说。  

你可以将这些 handler 与其他的合并这样错误就会在发出邮件的同时记录在服务器中：  

YAML:  

```
# app/config/config_prod.yml
monolog:
    handlers:
        main:
            type:         fingers_crossed
            action_level: critical
            handler:      grouped
        grouped:
            type:    group
            members: [streamed, buffered]
        streamed:
            type:  stream
            path:  "%kernel.logs_dir%/%kernel.environment%.log"
            level: debug
        buffered:
            type:    buffer
            handler: swift
        swift:
            type:       swift_mailer
            from_email: error@example.com
            to_email:   error@example.com
            subject:    An Error Occurred!
            level:      debug
```  

XML:  

```
<!-- app/config/config_prod.xml -->
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:monolog="http://symfony.com/schema/dic/monolog"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                        http://symfony.com/schema/dic/monolog http://symfony.com/schema/dic/monolog/monolog-1.0.xsd">

    <monolog:config>
        <monolog:handler
            name="main"
            type="fingers_crossed"
            action_level="critical"
            handler="grouped"
        />
        <monolog:handler
            name="grouped"
            type="group"
        >
            <member type="stream"/>
            <member type="buffered"/>
        </monolog:handler>
        <monolog:handler
            name="stream"
            path="%kernel.logs_dir%/%kernel.environment%.log"
            level="debug"
        />
        <monolog:handler
            name="buffered"
            type="buffer"
            handler="swift"
        />
        <monolog:handler
            name="swift"
            from-email="error@example.com"
            to-email="error@example.com"
            subject="An Error Occurred!"
            level="debug"
        />
    </monolog:config>
</container>
```  

PHP:  

```
// app/config/config_prod.php
$container->loadFromExtension('monolog', array(
    'handlers' => array(
        'main' => array(
            'type'         => 'fingers_crossed',
            'action_level' => 'critical',
            'handler'      => 'grouped',
        ),
        'grouped' => array(
            'type'    => 'group',
            'members' => array('streamed', 'buffered'),
        ),
        'streamed'  => array(
            'type'  => 'stream',
            'path'  => '%kernel.logs_dir%/%kernel.environment%.log',
            'level' => 'debug',
        ),
        'buffered'    => array(
            'type'    => 'buffer',
            'handler' => 'swift',
        ),
        'swift' => array(
            'type'       => 'swift_mailer',
            'from_email' => 'error@example.com',
            'to_email'   => 'error@example.com',
            'subject'    => 'An Error Occurred!',
            'level'      => 'debug',
        ),
    ),
));
```  

这个使用了 **group** handler 来向两个组员发送消息，**buffered** 和 **stream** handlers。现在这些消息既被记录在日志中又被电子邮件发送出去了。  
