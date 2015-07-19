# 如何创建一个控制台命令

组件一节（[The Console Component](http://symfony.com/doc/current/components/console/introduction.html)）的控制台页介绍了如何创建控制台命令。这个指导文章介绍了在 Symfony 框架下创建控制台命令的不同之处。  

## 自动注册命令

为了使得控制台命令在 Symfony 下自动可用，在你的 bundle 中创建一个 **Command** 目录并且以 **Command.php** 为后缀创建你想要的命令。举例来说，如果你想要扩展 AppBundle 来在命令行运行，那么创建 **GreetCommand.php** 并且将下列代码添加进去：  

```
// src/AppBundle/Command/GreetCommand.php
namespace AppBundle\Command;

use Symfony\Bundle\FrameworkBundle\Command\ContainerAwareCommand;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Output\OutputInterface;

class GreetCommand extends ContainerAwareCommand
{
    protected function configure()
    {
        $this
            ->setName('demo:greet')
            ->setDescription('Greet someone')
            ->addArgument(
                'name',
                InputArgument::OPTIONAL,
                'Who do you want to greet?'
            )
            ->addOption(
                'yell',
                null,
                InputOption::VALUE_NONE,
                'If set, the task will yell in uppercase letters'
            )
        ;
    }

    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $name = $input->getArgument('name');
        if ($name) {
            $text = 'Hello '.$name;
        } else {
            $text = 'Hello';
        }

        if ($input->getOption('yell')) {
            $text = strtoupper($text);
        }

        $output->writeln($text);
    }
}
```  

现在这个命令将会自动运行：  

```
$ php app/console demo:greet Fabien
```  

## 在服务容器中注册命令

就像控制器一样，命令也可以被作为服务声明。详见[专门的指导书条目](http://symfony.com/doc/current/cookbook/console/commands_as_services.html)。  

## 从服务容器中获取服务

通过使用 [ContainerAwareCommand](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Command/ContainerAwareCommand.html) 作为命令的基础类（而不是更基本的[命令](http://api.symfony.com/2.7/Symfony/Component/Console/Command/Command.html)），你就可以访问服务容器。换句话说，你可以访问任何设置的服务：  

```
protected function execute(InputInterface $input, OutputInterface $output)
{
    $name = $input->getArgument('name');
    $logger = $this->getContainer()->get('logger');

    $logger->info('Executing command for '.$name);
    // ...
}
```  

然而，由于[容器的范围](http://symfony.com/doc/current/cookbook/service_container/scopes.html)这个代码并不能对一些服务起作用。举例来说，如果你想要获得 **request** 服务或者其它和它相关的服务，你会看到下列错误：  

```
You cannot create a service ("request") of an inactive scope ("request").
```  

假设下面的例子使用了 **translator** 服务通过控制台命令翻译一些内容：  

```
protected function execute(InputInterface $input, OutputInterface $output)
{
    $name = $input->getArgument('name');
    $translator = $this->getContainer()->get('translator');
    if ($name) {
        $output->writeln(
            $translator->trans('Hello %name%!', array('%name%' => $name))
        );
    } else {
        $output->writeln($translator->trans('Hello!'));
    }
}
```  

如果你深入挖掘 Translator 组件类的话，你将看到 request 服务被要求获得局部进入内容被翻译的地方：  

```
// vendor/symfony/symfony/src/Symfony/Bundle/FrameworkBundle/Translation/Translator.php
public function getLocale()
{
    if (null === $this->locale && $this->container->isScopeActive('request')
        && $this->container->has('request')) {
        $this->locale = $this->container->get('request')->getLocale();
    }

    return $this->locale;
}
```  

因此，当使用命令中的 translator 服务时，你将会收到前述的 “*You cannot create a service of an inactive scope*” 的错误信息。这种情况的解决方法就像在翻译之前设置区域值一样容易：  

```
protected function execute(InputInterface $input, OutputInterface $output)
{
    $name = $input->getArgument('name');
    $locale = $input->getArgument('locale');

    $translator = $this->getContainer()->get('translator');
    $translator->setLocale($locale);

    if ($name) {
        $output->writeln(
            $translator->trans('Hello %name%!', array('%name%' => $name))
        );
    } else {
        $output->writeln($translator->trans('Hello!'));
    }
}
```  

然而对于其它的服务的解决方法就可能有些复杂了。获取更多细节信息，详见[如何使用范围](http://symfony.com/doc/current/cookbook/service_container/scopes.html)。  

## 调用其他命令

如果你需要启用一个需要运行其他命令的命令的话详见[调用存在的命令](http://symfony.com/doc/current/components/console/introduction.html#calling-existing-command)。  

## 测试命令

当测试满堆栈框架的部分命令时，[Symfony\Bundle\FrameworkBundle\Console\Application](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/Console/Application.html) 应当被使用而不是 [Symfony\Component\Console\Application](http://api.symfony.com/2.7/Symfony/Component/Console/Application.html)：  

```
use Symfony\Component\Console\Tester\CommandTester;
use Symfony\Bundle\FrameworkBundle\Console\Application;
use AppBundle\Command\GreetCommand;

class ListCommandTest extends \PHPUnit_Framework_TestCase
{
    public function testExecute()
    {
        // mock the Kernel or create one depending on your needs
        $application = new Application($kernel);
        $application->add(new GreetCommand());

        $command = $application->find('demo:greet');
        $commandTester = new CommandTester($command);
        $commandTester->execute(
            array(
                'name'    => 'Fabien',
                '--yell'  => true,
            )
        );

        $this->assertRegExp('/.../', $commandTester->getDisplay());

        // ...
    }
}
```  

> 在上述特定的情况下，name 参数和 --yell 选项对于命令执行不是强制的，但是进行展示这样你就可以理解在调用命令时如何自定义他们。  

为了能够完全使用服务容器为你的控制台测试服务你可以从 **KernelTestCase** 扩展你的测试：  

```
use Symfony\Component\Console\Tester\CommandTester;
use Symfony\Bundle\FrameworkBundle\Console\Application;
use Symfony\Bundle\FrameworkBundle\Test\KernelTestCase;
use AppBundle\Command\GreetCommand;

class ListCommandTest extends KernelTestCase
{
    public function testExecute()
    {
        $kernel = $this->createKernel();
        $kernel->boot();

        $application = new Application($kernel);
        $application->add(new GreetCommand());

        $command = $application->find('demo:greet');
        $commandTester = new CommandTester($command);
        $commandTester->execute(
            array(
                'name'    => 'Fabien',
                '--yell'  => true,
            )
        );

        $this->assertRegExp('/.../', $commandTester->getDisplay());

        // ...
    }
}
```