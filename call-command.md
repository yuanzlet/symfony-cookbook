# 如何从 Controller 调用一个命令

[控制台组件文档](http://symfony.com/doc/current/components/console/introduction.html)讲解了如何创建控制台命令。本指导文章将会讲解如何直接从 Controller 调用一个控制台命令。  

你可能有执行只有在控制台命令中可用的某些功能的需要。通常情况下，你应该重构命令然后向服务中移动一些能够在 controller 中应用的逻辑。然而，当命令是第三方函数库的一部分时，你就会不想修正或者复制它们的代码。作为替代你可以直接执行命令。  

> 和直接从控制台的直接调用相比，由于请求堆栈总开销，所以从 controller 调用命令稍微有一些性能上的影响。  

试想你想要将假脱机的 Swift Mailer 的信息通过[使用 swiftmailer:spool:send 命令](http://symfony.com/doc/current/cookbook/email/spool.html)发送。通过下列代码从你的 controller 的内部运行这个命令：  

```
// src/AppBundle/Controller/SpoolController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Console\Application;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\Console\Input\ArrayInput;
use Symfony\Component\Console\Output\BufferedOutput;
use Symfony\Component\HttpFoundation\Response;

class SpoolController extends Controller
{
    public function sendSpoolAction($messages = 10)
    {
        $kernel = $this->get('kernel');
        $application = new Application($kernel);
        $application->setAutoExit(false);

        $input = new ArrayInput(array(
           'command' => 'swiftmailer:spool:send',
           '--message-limit' => $messages,
        ));
        // You can use NullOutput() if you don't need the output
        $output = new BufferedOutput();
        $application->run($input, $output);

        // return the output, don't use if you used NullOutput()
        $content = $output->fetch();

        // return new Response(""), if you used NullOutput()
        return new Response($content);
    }
}
```  

## 显示彩色的命令输出

通过第二个参数告诉 **BufferedOutput** 这是可以装饰的，这将会返回 Ansi 颜色编码内容。 [SensioLabs AnsiToHtml 转换器](https://github.com/sensiolabs/ansi-to-html)可以用于将这个转化成彩色的 HTML。  

首先，请求包：  

```
$ composer require sensiolabs/ansi-to-html
```  

现在在你的 controller 中应用：  

```
// src/AppBundle/Controller/SpoolController.php
namespace AppBundle\Controller;

use SensioLabs\AnsiConverter\AnsiToHtmlConverter;
use Symfony\Component\Console\Output\BufferedOutput;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\HttpFoundation\Response;
// ...

class SpoolController extends Controller
{
    public function sendSpoolAction($messages = 10)
    {
        // ...
        $output = new BufferedOutput(
            OutputInterface::VERBOSITY_NORMAL,
            true // true for decorated
        );
        // ...

        // return the output
        $converter = new AnsiToHtmlConverter();
        $content = $output->fetch();

        return new Response($converter->convert($content));
    }
}
```  

**AnsiToHtmlConverter** 也可以注册[成为 Twig 扩展](https://github.com/sensiolabs/ansi-to-html#twig-integration)，并且支持可选的主题。