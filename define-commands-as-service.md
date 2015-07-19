# 如何把命令定义为服务

默认情况下，Symfony 将会在每一个 bundle 的 **Command** 目录下进行检查并且自动登录你的命令。如果一个命令扩展 [ContainerAwareCommand](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Command/ContainerAwareCommand.html)，Symfony 将会甚至注入这个容器。然而为了使得这个更容易，它有一些限制：  

- 你的命令必须在 **Command** 目录下；
- 基于你的环境或者依赖性的可用性没有注册你的服务的条件；
- 你不能使用 **configure()** 方法服务容器（因为 **setContainer** 还没有调用）；
- 你不能使用同一个类创建很多命令（例如每一个都有不同的配置）。

为了解决这个问题，你可以将你的命令注册为服务并且给它加上 **console.command** 的标签：  

YAML:

```YAML
# app/config/config.yml
services:
    app.command.my_command:
        class: AppBundle\Command\MyCommand
        tags:
            -  { name: console.command }
```  

XML:

```XML
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
    http://symfony.com/schema/dic/services/services-1.0.xsd">

    <services>
        <service id="app.command.my_command"
            class="AppBundle\Command\MyCommand">
            <tag name="console.command" />
        </service>
    </services>
</container>
```  

PHP:

```PHP
// app/config/config.php
$container
    ->register(
        'app.command.my_command',
        'AppBundle\Command\MyCommand'
    )
    ->addTag('console.command')
;
```  

## 使用依赖和参数设置默认选项的值

试想你想要给 **name** 选项一个默认值。你可以传递下面的一个作为 **addOption()** 的第五个参数：  

- 一个 hardcoded 字符串；
- 一个容器参数(例如 **parameters.yml** 中的一些)；
- 服务计算过的值（例如一个仓库）。

通过扩展 **ContainerAwareCommand**，只有第一个是可能的，由于你不能在 **configure()** 方法中访问容器。作为替代，你需要注入 constructor 任何参数或者服务。举例来说，假设你将默认值储存在一些 **%command.default_name%** 参数中：  

```
// src/AppBundle/Command/GreetCommand.php
namespace AppBundle\Command;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Output\OutputInterface;

class GreetCommand extends Command
{
    protected $defaultName;

    public function __construct($defaultName)
    {
        $this->defaultName = $defaultName;

        parent::__construct();
    }

    protected function configure()
    {
        // try to avoid work here (e.g. database query)
        // this method is *always* called - see warning below
        $defaultName = $this->defaultName;

        $this
            ->setName('demo:greet')
            ->setDescription('Greet someone')
            ->addOption(
                'name',
                '-n',
                InputOption::VALUE_REQUIRED,
                'Who do you want to greet?',
                $defaultName
            )
        ;
    }

    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $name = $input->getOption('name');

        $output->writeln($name);
    }
}
```  

现在，仅仅像往常一样更新你的服务配置的参数来注入 **command.default_name** 参数：  

YAML:

```YAML
# app/config/config.yml
parameters:
    command.default_name: Javier

services:
    app.command.my_command:
        class: AppBundle\Command\MyCommand
        arguments: ["%command.default_name%"]
        tags:
            -  { name: console.command }
```  

XML:

```XML
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
    http://symfony.com/schema/dic/services/services-1.0.xsd">

    <parameters>
        <parameter key="command.default_name">Javier</parameter>
    </parameters>

    <services>
        <service id="app.command.my_command"
            class="AppBundle\Command\MyCommand">
            <argument>%command.default_name%</argument>
            <tag name="console.command" />
        </service>
    </services>
</container>
```  

PHP:

```PHP
// app/config/config.php
$container->setParameter('command.default_name', 'Javier');

$container
    ->register(
        'app.command.my_command',
        'AppBundle\Command\MyCommand',
    )
    ->setArguments(array('%command.default_name%'))
    ->addTag('console.command')
;
```  

很好，你现在有了动态的默认值！  

> 注意不要在 **configure** 中做任何工作（例如做出数据库请求），由于你的代码将会运行，即使你在使用控制台执行不同的命令。