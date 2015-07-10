# 如何在控制台命令中启用日志

控制台组件没有提供任何的立即可用的日志能力。正常来讲，你可以手动运行控制台命令观察输出结果，这就是为什么不提供日志的原因。然而，有些时候你就需要日志。举例来说，如果你是自动运行控制台命令，例如从工作或者开发脚本，很容易使用 Symfony 的日志能力而不是配置其他工具去收集控制台的输出并且编辑它。这个特别顺手如果你已经具有一些存在的聚合并且分析 Symfony 的日志的设置。  

这里有两种日志的情况你将会需要：  

- 从你的命令中手动的记录一些信息；
- 记录未被发现的错误。  

## 从控制台命令手动记录 ##

这个真的很简单。当你像“[如何创建控制台命令](http://symfony.com/doc/current/cookbook/console/console_command.html)”中描述的那样在满堆栈框架下创建一个控制台命令时，你的命令扩展 [ContainerAwareCommand](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Command/ContainerAwareCommand.html)。这也就意味着你可以很容易的通过容器访问标准日志服务并且使用它做记录：  

```
// src/AppBundle/Command/GreetCommand.php
namespace AppBundle\Command;

use Symfony\Bundle\FrameworkBundle\Command\ContainerAwareCommand;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Output\OutputInterface;
use Psr\Log\LoggerInterface;

class GreetCommand extends ContainerAwareCommand
{
    // ...

    protected function execute(InputInterface $input, OutputInterface $output)
    {
        /** @var $logger LoggerInterface */
        $logger = $this->getContainer()->get('logger');

        $name = $input->getArgument('name');
        if ($name) {
            $text = 'Hello '.$name;
        } else {
            $text = 'Hello';
        }

        if ($input->getOption('yell')) {
            $text = strtoupper($text);
            $logger->warning('Yelled: '.$text);
        } else {
            $logger->info('Greeted: '.$text);
        }

        $output->writeln($text);
    }
}
```  

依赖于你运行命令的（以及你的日志设置的）环境你会看到在 **app/logs/dev.log** 或者 **app/logs/prod.log** 中的日志条目。  

## 启用自动错误记录 ##

为了使得你的控制台自动记录你的所有命令的未捕获的错误，你可以使用 [console events](http://symfony.com/doc/current/components/console/events.html)。

>Console events 于 Symfony 2.3 中引入。  

首先在服务容器中配置一个控制台异常事件的监听器：  

```YAML
# app/config/services.yml
services:
    kernel.listener.command_dispatch:
        class: AppBundle\EventListener\ConsoleExceptionListener
        arguments:
            logger: "@logger"
        tags:
            - { name: kernel.event_listener, event: console.exception }
```  

```XML
<!-- app/config/services.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

    <services>
        <service id="kernel.listener.command_dispatch" class="AppBundle\EventListener\ConsoleExceptionListener">
            <argument type="service" id="logger"/>
            <tag name="kernel.event_listener" event="console.exception" />
        </service>
    </services>
</container>
```  

```PHP
// app/config/services.php
use Symfony\Component\DependencyInjection\Definition;
use Symfony\Component\DependencyInjection\Reference;

$definitionConsoleExceptionListener = new Definition(
    'AppBundle\EventListener\ConsoleExceptionListener',
    array(new Reference('logger'))
);
$definitionConsoleExceptionListener->addTag(
    'kernel.event_listener',
    array('event' => 'console.exception')
);
$container->setDefinition(
    'kernel.listener.command_dispatch',
    $definitionConsoleExceptionListener
);
```  

然后启用实际的监听器：  

```
// src/AppBundle/EventListener/ConsoleExceptionListener.php
namespace AppBundle\EventListener;

use Symfony\Component\Console\Event\ConsoleExceptionEvent;
use Psr\Log\LoggerInterface;

class ConsoleExceptionListener
{
    private $logger;

    public function __construct(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }

    public function onConsoleException(ConsoleExceptionEvent $event)
    {
        $command = $event->getCommand();
        $exception = $event->getException();

        $message = sprintf(
            '%s: %s (uncaught exception) at %s line %s while running console command `%s`',
            get_class($exception),
            $exception->getMessage(),
            $exception->getFile(),
            $exception->getLine(),
            $command->getName()
        );

        $this->logger->error($message, array('exception' => $exception));
    }
}
```  

在上述的代码中，当任何命令出现异常，监听器都会受到一个事件。你可以简单的通过服务配置传递日志服务的方式来记录它。你的方法会收到一个 [ConsoleExceptionEvent](http://api.symfony.com/2.7/Symfony/Component/Console/Event/ConsoleExceptionEvent.html) 对象，这个对象有获得事件和异常的信息的能力。  

## 记录非 0 的退出状态 ##

控制台的日志记录功能可以通过记录非 0的 退出状态被进一步扩展。这样你就会知道一个命令是否有任何错误，即使没有异常出现。  

首先在服务容器中创建控制台终止事件监听器：  

```YAML
# app/config/services.yml
services:
    kernel.listener.command_dispatch:
        class: AppBundle\EventListener\ErrorLoggerListener
        arguments:
            logger: "@logger"
        tags:
            - { name: kernel.event_listener, event: console.terminate }
```  

```XML
<!-- app/config/services.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

    <services>
        <service id="kernel.listener.command_dispatch" class="AppBundle\EventListener\ErrorLoggerListener">
            <argument type="service" id="logger"/>
            <tag name="kernel.event_listener" event="console.terminate" />
        </service>
    </services>
</container>
```  

```PHP
// app/config/services.php
use Symfony\Component\DependencyInjection\Definition;
use Symfony\Component\DependencyInjection\Reference;

$definitionErrorLoggerListener = new Definition(
    'AppBundle\EventListener\ErrorLoggerListener',
    array(new Reference('logger'))
);
$definitionErrorLoggerListener->addTag(
    'kernel.event_listener',
    array('event' => 'console.terminate')
);
$container->setDefinition(
    'kernel.listener.command_dispatch',
    $definitionErrorLoggerListener
);
```  

然后启用实际监听器：  

```
// src/AppBundle/EventListener/ErrorLoggerListener.php
namespace AppBundle\EventListener;

use Symfony\Component\Console\Event\ConsoleTerminateEvent;
use Psr\Log\LoggerInterface;

class ErrorLoggerListener
{
    private $logger;

    public function __construct(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }

    public function onConsoleTerminate(ConsoleTerminateEvent $event)
    {
        $statusCode = $event->getExitCode();
        $command = $event->getCommand();

        if ($statusCode === 0) {
            return;
        }

        if ($statusCode > 255) {
            $statusCode = 255;
            $event->setExitCode($statusCode);
        }

        $this->logger->warning(sprintf(
            'Command `%s` exited with status code %d',
            $command->getName(),
            $statusCode
        ));
    }
}
```  




