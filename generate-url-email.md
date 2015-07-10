# 如何在控制台生成 URL 和 发送邮件

不幸的是，命令行环境不知道你的虚拟主机或者域的名称。这就意味着如果你使用控制台命令生成了绝对的 URL 你将可能像 **http://localhost/foo/bar** 一样结束而并没有什么用。  

为了解决这个问题，你需要配置“请求环境”，这是一种受欢迎的说明方式也就是你需要设置你的环境使得它知道当生成 URL 的时候该用哪一个。  

这里有两种设置请求环境的方法：在应用程序层面上以及每一个命令层面。  

## 全局地设置请求环境 ##

为了设置被 URL 生成器所使用的请求环境，你可以将他所使用的参数定义成默认值来改变默认的 host （localhost）和策略（http）。你也可以设置基本路径如果 Symfony 不在根目录运行。  

记住这个不是通过简单的网页请求影响 URL 生成器，由于那些会重写默认值。  

```YAML
# app/config/parameters.yml
parameters:
    router.request_context.host: example.org
    router.request_context.scheme: https
    router.request_context.base_url: my/path
```  

```XML
<!-- app/config/parameters.xml -->
<?xml version="1.0" encoding="UTF-8"?>

<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    <parameters>
        <parameter key="router.request_context.host">example.org</parameter>
        <parameter key="router.request_context.scheme">https</parameter>
        <parameter key="router.request_context.base_url">my/path</parameter>
    </parameters>
</container>
```  

```PHP
// app/config/config_test.php
$container->setParameter('router.request_context.host', 'example.org');
$container->setParameter('router.request_context.scheme', 'https');
$container->setParameter('router.request_context.base_url', 'my/path');
```  

## 为每个命令配置请求环境 ##

只在一个命令中改变你可以简单地将请求环境从 **router** 服务中取出然后重写它的设置：  

```
// src/AppBundle/Command/DemoCommand.php

// ...
class DemoCommand extends ContainerAwareCommand
{
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $context = $this->getContainer()->get('router')->getContext();
        $context->setHost('example.com');
        $context->setScheme('https');
        $context->setBaseUrl('my/path');

        // ... your code here
    }
}
```  

## 使用内存假脱机 ##

>当使用 Symfony 2.3+ 和 SwiftmailerBundle 2.3.5+ 时，内存假脱机现在是在 CLI 中自动处理的，下列代码就不在需要了。  

在控制台命令中发送邮件和[如何发送邮件](http://symfony.com/doc/current/cookbook/email/email.html)指导中描述的一样除了内存假脱机被占用。  

当使用内存假脱机时（更多信息详见[如何假脱机邮件](http://symfony.com/doc/current/cookbook/email/spool.html)指导），你必须知道由于 Symfony 处理控制台命令的方式，邮件将不会被自动发送。你必须自己处理队列。使用下列代码发送你的控制台命令中的邮件：  

```
$message = new \Swift_Message();

// ... prepare the message

$container = $this->getContainer();
$mailer = $container->get('mailer');

$mailer->send($message);

// now manually flush the queue
$spool = $mailer->getTransport()->getSpool();
$transport = $container->get('swiftmailer.transport.real');

$spool->flushQueue($transport);
```  

另外的一个选项是创建一个只用于控制台命令的环境并且使用一种不同的假脱机方法。  

>只有当内存假脱机被使用时照顾假脱机。如果你使用文件假脱机（或者不完全假脱机），没必要手动在命令中清除队列。  


