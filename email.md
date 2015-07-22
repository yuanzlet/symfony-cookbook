# 如何发送一封电子邮件

发送电子邮件对于任何 web 应用程序来说，都是一个经典任务，并且具有特殊的复杂性和潜在的缺陷。不是重新创建轮，发送电子邮件的解决方案之一就是使用 SwiftmailerBundle，利用 [Swift Mailer](http://swiftmailer.org/) 库的能力。这个 bundle 来自于 Symfony 标准版本。

## 配置

使用 Swift Mailer 的话，您需要对您的邮件服务器配置。

> 如果不设置/使用您自己的邮件服务器，您可能想使用一个主邮件服务器，例如 [Mandrill](https://mandrill.com/)，[Sendgrid](https://sendgrid.com/)，[Amazon SES](http://aws.amazon.com/cn/ses/) 或者其他的。这些给您一个 SMTP 服务器、用户名和密码（有的时候叫做秘钥），可以在 Swift Mailer 配置中使用。

在一个标准的 Symfony 安装中，一些 **swfitmailer** 配置已经包含在内了：

YAML

```
# app/config/config.yml
swiftmailer:
    transport: "%mailer_transport%"
    host:      "%mailer_host%"
    username:  "%mailer_user%"
    password:  "%mailer_password%"
```

XML

```
<!-- app/config/config.xml -->

<!--
    xmlns:swiftmailer="http://symfony.com/schema/dic/swiftmailer"
    http://symfony.com/schema/dic/swiftmailer http://symfony.com/schema/dic/swiftmailer/swiftmailer-1.0.xsd
-->

<swiftmailer:config
    transport="%mailer_transport%"
    host="%mailer_host%"
    username="%mailer_user%"
    password="%mailer_password%" />
```

PHP

```
// app/config/config.php
$container->loadFromExtension('swiftmailer', array(
    'transport'  => "%mailer_transport%",
    'host'       => "%mailer_host%",
    'username'   => "%mailer_user%",
    'password'   => "%mailer_password%",
));
```

这些值（例如 **%mailer_transport%**）可从设置在 [parameters.yml](http://symfony.com/doc/current/best_practices/configuration.html#config-parameters-yml) 文件中的参数中读出。您可以在该文件中修饰这些值或者直接在这里设置值。

以下配置属性可用：

•	**transport** (**smtp**, **mail**, **sendmail**, 或者 **gmail**)

•	**username**

•	**password**

•	**host**

•	**port**

•	**encryption** (**tls**,或者 **ssl**)

•	**auth_mode** (**plain**, **login**,或者 **cram-md5**)

•	**spool**

    o	**type** （如何排列消息，支持 **file** 或者 **memory**，参见[如何缓存电子邮件](http://symfony.com/doc/current/cookbook/email/spool.html)）

    o	**path**（存储消息的地方）

•	**delivery_address**（一个可以发送所有电子邮件的地址）

•	**disable_delivery**（设置为 true 来完全禁用转发）

## 发送电子邮件

Swift Mailer 库通过创建、配置然后发送 **Swift_Message** 对象来工作。“mailer” 负责实际的转发消息并且通过 **mailer** 服务来访问。综上，发送一封电子邮件是相当简单的。

```
public function indexAction($name)
{
    $message = \Swift_Message::newInstance()
        ->setSubject('Hello Email')
        ->setFrom('send@example.com')
        ->setTo('recipient@example.com')
        ->setBody(
            $this->renderView(
                // app/Resources/views/Emails/registration.html.twig
                'Emails/registration.html.twig',
                array('name' => $name)
            ),
            'text/html'
        )
        /*
         * If you also want to include a plaintext version of the message
        ->addPart(
            $this->renderView(
                'Emails/registration.txt.twig',
                array('name' => $name)
            ),
            'text/plain'
        )
        */
    ;
    $this->get('mailer')->send($message);

    return $this->render(...);
}
```

把事情分解，电子邮件主体存储在一个模板中，并由 **renderView()** 方法显示。**registration.html.twig** 模板可能看起来像这样：

```
{# app/Resources/views/Emails/registration.html.twig #}
<h3>You did it! You registered!</h3>

{# example, assuming you have a route named "login" #}
To login, go to: <a href="{{ url('login') }}">...</a>.

Thanks!

{# Makes an absolute URL to the /images/logo.png file #}
<img src="{{ absolute_url(asset('images/logo.png')) }}"
```

**$message** 对象支持更多的选择，如包括附件，添加 HTML 内容，以及更多。幸运地是，Swift Mailer 在本文档中详细得讲了关于[创建信息](http://swiftmailer.org/docs/messages.html)的主题。

> 其他可用的有关 Symfony 发送电子邮件的教程文章：

•	[如何使用 Gmail 发送电子邮件](http://symfony.com/doc/current/cookbook/email/gmail.html)

•	[如何在开发过程中使用电子邮件](http://symfony.com/doc/current/cookbook/email/dev_environment.html)

•	[如何缓存电子邮件](http://symfony.com/doc/current/cookbook/email/spool.html)
