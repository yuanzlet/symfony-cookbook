# 如何使用 Monolog 记录日志

[Monolog](https://github.com/Seldaek/monolog) 是 Symfony 使用的 PHP 日志记录函数。它由 Python LogBook 函数产生。  

## 使用

只需要在你控制器的容器中获得 **logger** 服务就可以记录消息了：  

```
public function indexAction()
{
    $logger = $this->get('logger');
    $logger->info('I just got the logger');
    $logger->error('An error occurred');

    // ...
}
```

**logger** 服务在不同的日志水平上有不同的方法，哪种方法可用详见 [LoggerInterface](https://github.com/php-fig/log/blob/master/Psr/Log/LoggerInterface.php) 获取更多细节信息。  

## Handlers 和 Channels：在不同位置记录日志

Monolog 中每一个日志都定义了日志信道，这个将你的日志信息组织成不同的“目录”。然后，每一个频道都有一堆 handlers 来写日志（handlers 可以共享）。  

> 当在服务中注入日志编写器时你可以[使用定制的频道](http://symfony.com/doc/current/reference/dic_tags.html#dic-tags-monolog)控件这个定义了日志编写器的“频道”。  

基本的 handler 是 **StreamHandler**，它在流中记录日志（默认情况下 prod 环境下的 **app/logs/prod.log** 以及 dev 环境下的 **app/logs/dev.log**）。  

Monolog 也有一个强力的内建 handler 来在 prod 环境下记录日志：**FingersCrossedHandler**。它允许你在缓冲区储存消息并且只有当消息到达行为层次才记录它们（这个在 Symfony 的标准版本提供的配置中是**错误**的）通过将消息传递到另一个 handler 的方式。  

### 使用几个 Handlers

日志记录器用了一堆 handlers，这些 handlers 相继被调用。这就允许你很容易的以多种不同的方式来记录信息。  

YAML:  

```
# app/config/config.yml
monolog:
    handlers:
        applog:
            type: stream
            path: /var/log/symfony.log
            level: error
        main:
            type: fingers_crossed
            action_level: warning
            handler: file
        file:
            type: stream
            level: debug
        syslog:
            type: syslog
            level: error
```

XML:  

```
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:monolog="http://symfony.com/schema/dic/monolog"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/monolog
        http://symfony.com/schema/dic/monolog/monolog-1.0.xsd">

    <monolog:config>
        <monolog:handler
            name="applog"
            type="stream"
            path="/var/log/symfony.log"
            level="error"
        />
        <monolog:handler
            name="main"
            type="fingers_crossed"
            action-level="warning"
            handler="file"
        />
        <monolog:handler
            name="file"
            type="stream"
            level="debug"
        />
        <monolog:handler
            name="syslog"
            type="syslog"
            level="error"
        />
    </monolog:config>
</container>
PHP
```

PHP:  

```
// app/config/config.php
$container->loadFromExtension('monolog', array(
    'handlers' => array(
        'applog' => array(
            'type'  => 'stream',
            'path'  => '/var/log/symfony.log',
            'level' => 'error',
        ),
        'main' => array(
            'type'         => 'fingers_crossed',
            'action_level' => 'warning',
            'handler'      => 'file',
        ),
        'file' => array(
            'type'  => 'stream',
            'level' => 'debug',
        ),
        'syslog' => array(
            'type'  => 'syslog',
            'level' => 'error',
        ),
    ),
));
```

上述配置定义了一堆 handlers 这些 handlers 将会以它们被定义的顺序被调用。  

> handler 命名的“文件”不会包含在它自己中因为它被用作是 **fingers_crossed** handler 的嵌套的 handler。  

> 如果你想要在另外的配置文件中改变 MonologBundle 的配置你需要重新定义整个堆栈，它不能被合并因为顺序很重要而且合并不能控制顺序。  

### 改变格式

handler 使用 **Formatter** 来在记录日志之前格式化它。所有的 Monolog handlers 默认使用 **Monolog\Formatter\LineFormatter** 的实例但是你可以很容易的替换它，你的格式器必须实现 **Monolog\Formatter\FormatterInterface**。  

YAML:  

```
# app/config/config.yml
services:
    my_formatter:
        class: Monolog\Formatter\JsonFormatter
monolog:
    handlers:
        file:
            type: stream
            level: debug
            formatter: my_formatter
```

XML:  

```
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:monolog="http://symfony.com/schema/dic/monolog"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/monolog
        http://symfony.com/schema/dic/monolog/monolog-1.0.xsd">

    <services>
        <service id="my_formatter" class="Monolog\Formatter\JsonFormatter" />
    </services>

    <monolog:config>
        <monolog:handler
            name="file"
            type="stream"
            level="debug"
            formatter="my_formatter"
        />
    </monolog:config>
</container>
```

PHP:  

```
// app/config/config.php
$container
    ->register('my_formatter', 'Monolog\Formatter\JsonFormatter');

$container->loadFromExtension('monolog', array(
    'handlers' => array(
        'file' => array(
            'type'      => 'stream',
            'level'     => 'debug',
            'formatter' => 'my_formatter',
        ),
    ),
));
```

## 如何循环你的日志文件

经过一段时间，日志文件就会变得*很大*，不管是在开发还是产品环境下。最好的解决方法就是在他们变得太大之前使用一个像 Linux 的 [logrotate](https://fedorahosted.org/logrotate/) 命令一样的工具来循环日志文件。  

另外一个选项就是通过使用 **rotating_file** handler 来使得 Monolog 来循环文件。这个 handler 每天创建一个新的日志文件并且可以自动移除旧的日志文件。使用它，仅仅需要配置你的 handler 的 **type** 选项到 **rotating_file**：  

YAML:  

```
# app/config/config_dev.yml
monolog:
    handlers:
        main:
            type:  rotating_file
            path:  %kernel.logs_dir%/%kernel.environment%.log
            level: debug
            # max number of log files to keep
            # defaults to zero, which means infinite files
            max_files: 10
```

XML:  

```
<!-- app/config/config_dev.xml -->
<?xml version="1.0" charset="UTF-8" ?>
<container xmlns=''http://symfony.com/schema/dic/services"
    xmlns:monolog="http://symfony.com/schema/dic/monolog">

    <monolog:config>
        <monolog:handler name="main"
            type="rotating_file"
            path="%kernel.logs_dir%/%kernel.environment%.log"
            level="debug"
            <!-- max number of log files to keep
                 defaults to zero, which means infinite files -->
            max_files="10"
        />
    </monolog:config>
</container>
```

PHP:  

```
// app/config/config_dev.php
$container->loadFromExtension('monolog', array(
    'handlers' => array(
        'main' => array(
            'type'  => 'rotating_file',
            'path'  => '%kernel.logs_dir%/%kernel.environment%.log',
            'level' => 'debug',
            // max number of log files to keep
            // defaults to zero, which means infinite files
            'max_files' => 10,
        ),
    ),
));
```

## 在日志信息中添加一些额外数据

Monolog 允许你编辑记录在它被记录之前添加一些额外数据。processor 可以应用到整个 handler 堆栈也可以对一个特定的 handler 应用。  

processor 就是一个可调用的接收记录为它的第一变元。Processors 使用 **monolog.processor** DIC tag 进行设置。详见[关于它的指导](http://symfony.com/doc/current/reference/dic_tags.html#dic-tags-monolog-processor)。  

### 添加一个节/请求标记

有时候很难说明日志中的哪一条是属于哪个节或者请求的。下面的例子就会为使用 processor 的每一个请求添加一个独特的标记。  

```
namespace Acme\MyBundle;

use Symfony\Component\HttpFoundation\Session\Session;

class SessionRequestProcessor
{
    private $session;
    private $token;

    public function __construct(Session $session)
    {
        $this->session = $session;
    }

    public function processRecord(array $record)
    {
        if (null === $this->token) {
            try {
                $this->token = substr($this->session->getId(), 0, 8);
            } catch (\RuntimeException $e) {
                $this->token = '????????';
            }
            $this->token .= '-' . substr(uniqid(), -8);
        }
        $record['extra']['token'] = $this->token;

        return $record;
    }
}
```

YAML:  

```
# app/config/config.yml
services:
    monolog.formatter.session_request:
        class: Monolog\Formatter\LineFormatter
        arguments:
            - "[%%datetime%%] [%%extra.token%%] %%channel%%.%%level_name%%: %%message%%\n"

    monolog.processor.session_request:
        class: Acme\MyBundle\SessionRequestProcessor
        arguments:  ["@session"]
        tags:
            - { name: monolog.processor, method: processRecord }

monolog:
    handlers:
        main:
            type: stream
            path: "%kernel.logs_dir%/%kernel.environment%.log"
            level: debug
            formatter: monolog.formatter.session_request
```

XML:  

```
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:monolog="http://symfony.com/schema/dic/monolog"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/monolog
        http://symfony.com/schema/dic/monolog/monolog-1.0.xsd">

    <services>
        <service id="monolog.formatter.session_request"
            class="Monolog\Formatter\LineFormatter">

            <argument>[%%datetime%%] [%%extra.token%%] %%channel%%.%%level_name%%: %%message%%&#xA;</argument>
        </service>

        <service id="monolog.processor.session_request"
            class="Acme\MyBundle\SessionRequestProcessor">

            <argument type="service" id="session" />
            <tag name="monolog.processor" method="processRecord" />
        </service>
    </services>

    <monolog:config>
        <monolog:handler
            name="main"
            type="stream"
            path="%kernel.logs_dir%/%kernel.environment%.log"
            level="debug"
            formatter="monolog.formatter.session_request"
        />
    </monolog:config>
</container>
```

PHP:  

```
// app/config/config.php
$container
    ->register(
        'monolog.formatter.session_request',
        'Monolog\Formatter\LineFormatter'
    )
    ->addArgument('[%%datetime%%] [%%extra.token%%] %%channel%%.%%level_name%%: %%message%%\n');

$container
    ->register(
        'monolog.processor.session_request',
        'Acme\MyBundle\SessionRequestProcessor'
    )
    ->addArgument(new Reference('session'))
    ->addTag('monolog.processor', array('method' => 'processRecord'));

$container->loadFromExtension('monolog', array(
    'handlers' => array(
        'main' => array(
            'type'      => 'stream',
            'path'      => '%kernel.logs_dir%/%kernel.environment%.log',
            'level'     => 'debug',
            'formatter' => 'monolog.formatter.session_request',
        ),
    ),
));
```

> 如果你使用几个 handler，你也可以在 handler 层注册一个 processor 或者在频道层而不是全局注册（详见下面一节）。  

## 每个 Handler 注册 Processors

你可以使用 **monolog.processor** 标签的 **handler** 选项来为每一个 Handler 注册 Processor：  

YAML:  

```
# app/config/config.yml
services:
    monolog.processor.session_request:
        class: Acme\MyBundle\SessionRequestProcessor
        arguments:  ["@session"]
        tags:
            - { name: monolog.processor, method: processRecord, handler: main }
```

XML:  

```
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:monolog="http://symfony.com/schema/dic/monolog"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/monolog
        http://symfony.com/schema/dic/monolog/monolog-1.0.xsd">

    <services>
        <service id="monolog.processor.session_request"
            class="Acme\MyBundle\SessionRequestProcessor">

            <argument type="service" id="session" />
            <tag name="monolog.processor" method="processRecord" handler="main" />
        </service>
    </services>
</container>
```

PHP:  

```
// app/config/config.php
$container
    ->register(
        'monolog.processor.session_request',
        'Acme\MyBundle\SessionRequestProcessor'
    )
    ->addArgument(new Reference('session'))
    ->addTag('monolog.processor', array('method' => 'processRecord', 'handler' => 'main'));
```

## 每个频道注册 Processors

你可以使用 **monolog.processor** 标签的 **channel** 选项来为每一个频道注册 Processor：  

YAML:  

```
# app/config/config.yml
services:
    monolog.processor.session_request:
        class: Acme\MyBundle\SessionRequestProcessor
        arguments:  ["@session"]
        tags:
            - { name: monolog.processor, method: processRecord, channel: main }
```

XML:  

```
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:monolog="http://symfony.com/schema/dic/monolog"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/monolog
        http://symfony.com/schema/dic/monolog/monolog-1.0.xsd">

    <services>
        <service id="monolog.processor.session_request"
            class="Acme\MyBundle\SessionRequestProcessor">

            <argument type="service" id="session" />
            <tag name="monolog.processor" method="processRecord" channel="main" />
        </service>
    </services>
</container>
```

PHP:  

```
// app/config/config.php
$container
    ->register(
        'monolog.processor.session_request',
        'Acme\MyBundle\SessionRequestProcessor'
    )
    ->addArgument(new Reference('session'))
    ->addTag('monolog.processor', array('method' => 'processRecord', 'channel' => 'main'));
```
