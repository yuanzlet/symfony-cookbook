# 如何对显示控制台信息配置 Monolog

使用当命令得到执行的时候传递的 [OutputInterface](http://api.symfony.com/2.7/Symfony/Component/Console/Output/OutputInterface.html) 实例的特定[信息显示层面](http://symfony.com/doc/current/components/console/introduction.html#verbosity-levels)消息，可以使用控制台来打印。  

> 作为替代，你也可以使用控制台组件提供的[独立的 PSR-3 日志记录器](http://symfony.com/doc/current/components/console/logger.html)。  

当很多的日志需要记录时，依靠信息显示设置（**-v, -vv, -vvv**）来打印信息就很麻烦，因为调用需要有条件覆盖。代码很快就会变得冗长。举例来说：  

```
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

protected function execute(InputInterface $input, OutputInterface $output)
{
    if ($output->getVerbosity() >= OutputInterface::VERBOSITY_DEBUG) {
        $output->writeln('Some info');
    }

    if ($output->getVerbosity() >= OutputInterface::VERBOSITY_VERBOSE) {
        $output->writeln('Some more info');
    }
}
```

没有使用这些语法方法来检验每一个信息显示层，MonologBridge 提供了一个 ConsoleHandler，它可以监听控制台事件并且能够基于目前的日志水平以及控制台信息显示来将日志消息记录到控制台输出中。  

上述例子就可以被写成下面这样：  

```
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

protected function execute(InputInterface $input, OutputInterface $output)
{
    // assuming the Command extends ContainerAwareCommand...
    $logger = $this->getContainer()->get('logger');
    $logger->debug('Some info');

    $logger->notice('Some more info');
}
```

依赖于命令运行的信息显示层以及用户的配置（看下面），这些消息可能会也可能不会向控制台展示。如果他们被展示，他们将会被加上时间标记并且被适当上色。除此之外，错误日志将会被写到错误输出中（php://stderr）。这里没有必要再有条件地处理信息显示设置。  

Monolog 控制台 handler 在 Monolog 配置中可用。这在 Symfony Standard Edition 2.4 中也是默认的。  

YAML:  

```
# app/config/config.yml
monolog:
    handlers:
        console:
            type: console
```

XML:  

```
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:monolog="http://symfony.com/schema/dic/monolog">

    <monolog:config>
        <monolog:handler name="console" type="console" />
    </monolog:config>
</container>
```

PHP:  

```
// app/config/config.php
$container->loadFromExtension('monolog', array(
    'handlers' => array(
        'console' => array(
           'type' => 'console',
        ),
    ),
));
```

使用 **verbosity_levels** 选项你可以适应信息显示以及日志层面的映射。在上述的例子中也将会展示一些正常信息显示模式下的通知（而不仅是警告）。除此之外，它将会只使用由定制的 **my_channel** 频道记录的消息并且通过定制的格式器来改变现实风格（更多详细信息参见 [MonologBundle 指南](http://symfony.com/doc/current/reference/configuration/monolog.html)）

YAML:  

```
# app/config/config.yml
monolog:
    handlers:
        console:
            type:   console
            verbosity_levels:
                VERBOSITY_NORMAL: NOTICE
            channels: my_channel
            formatter: my_formatter
```

XML:  

```
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:monolog="http://symfony.com/schema/dic/monolog">

    <monolog:config>
        <monolog:handler name="console" type="console" formatter="my_formatter">
            <monolog:verbosity-level verbosity-normal="NOTICE" />
            <monolog:channel>my_channel</monolog:channel>
        </monolog:handler>
    </monolog:config>
</container>
```

PHP:  

```
// app/config/config.php
$container->loadFromExtension('monolog', array(
    'handlers' => array(
        'console' => array(
            'type' => 'console',
            'verbosity_levels' => array(
                'VERBOSITY_NORMAL' => 'NOTICE',
            ),
            'channels' => 'my_channel',
            'formatter' => 'my_formatter',
        ),
    ),
));
```

YAML:  

```
# app/config/services.yml
services:
    my_formatter:
        class: Symfony\Bridge\Monolog\Formatter\ConsoleFormatter
        arguments:
            - "[%%datetime%%] %%start_tag%%%%message%%%%end_tag%% (%%level_name%%) %%context%% %%extra%%\n"
```

XML:  

```
<!-- app/config/services.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

     <services>
        <service id="my_formatter" class="Symfony\Bridge\Monolog\Formatter\ConsoleFormatter">
            <argument>[%%datetime%%] %%start_tag%%%%message%%%%end_tag%% (%%level_name%%) %%context%% %%extra%%\n</argument>
        </service>
     </services>

</container>
```

PHP:  

```
// app/config/services.php
$container
    ->register('my_formatter', 'Symfony\Bridge\Monolog\Formatter\ConsoleFormatter')
    ->addArgument('[%%datetime%%] %%start_tag%%%%message%%%%end_tag%% (%%level_name%%) %%context%% %%extra%%\n')
;
```
