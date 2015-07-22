# 如何使用云服务发送电子邮件

从一个生产系统发送电子邮件的要求不同于你的发展设置，因为你不想被电子邮件的数量、发送速率或者发送者地址所限制。因此，[使用 Gmail] 或者相似的服务不是一个选择。如果设置或者保持您自己可信的邮件服务器让您很头疼，这里有一个简单的解决方案：利用云服务来发送您的邮件。

本教程展示给您整合 [Amazon 的简单电子邮件服务(SES)](http://aws.amazon.com/cn/ses/) 到 Symfony 是多么简单。

> 您可以将相同的技术用于其他邮件服务。因为大多时候只不过是为 Swift Mailer 配置一个 SMTP 终结点。

在 Symfony 配置中，根据 [SES 控制台](https://console.aws.amazon.com/ses/)提供的消息改变 Swift Mailer 的设置 **transport**, **host**, **port** 和 **encryption**。在 SES 控制台中创建您个人的 SMTP 证书并借助所提供的 **username** 和 **password** 完成任务。

YAML

```
# app/config/config.yml
swiftmailer:
    transport:  smtp
    host:       email-smtp.us-east-1.amazonaws.com
    port:       465 # different ports are available, see SES console
    encryption: tls # TLS encryption is required
    username:   AWS_ACCESS_KEY  # to be created in the SES console
    password:   AWS_SECRET_KEY  # to be created in the SES console
```

XML

```
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:swiftmailer="http://symfony.com/schema/dic/swiftmailer"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/swiftmailer
        http://symfony.com/schema/dic/swiftmailer/swiftmailer-1.0.xsd">

    <!-- ... -->
    <swiftmailer:config
        transport="smtp"
        host="email-smtp.us-east-1.amazonaws.com"
        port="465"
        encryption="tls"
        username="AWS_ACCESS_KEY"
        password="AWS_SECRET_KEY"
    />
</container>
```

PHP

```
// app/config/config.php
$container->loadFromExtension('swiftmailer', array(
    'transport'  => 'smtp',
    'host'       => 'email-smtp.us-east-1.amazonaws.com',
    'port'       => 465,
    'encryption' => 'tls',
    'username'   => 'AWS_ACCESS_KEY',
    'password'   => 'AWS_SECRET_KEY',
));
```

**port** 和 **encryption** 密钥在 Symfony 标准版本配置中默认情况下是不显示的，但是您可以简单地按照需要添加。  

就是这样，您已经准备好通过云来发送邮件了。

> 如果您使用 Symfony 标准版本，在 **parameters.yml** 中配置参数，并在您的配置文件中使用。这允许每个应用程序的安装有不同的 Swift Mailer。例如，在开发中使用 Gmail，在生产中使用云服务。

```
# app/config/parameters.yml
parameters:
    # ...
    mailer_transport:  smtp
    mailer_host:       email-smtp.us-east-1.amazonaws.com
    mailer_port:       465 # different ports are available, see SES console
    mailer_encryption: tls # TLS encryption is required
    mailer_user:       AWS_ACCESS_KEY # to be created in the SES console
    mailer_password:   AWS_SECRET_KEY # to be created in the SES console
```

> 如果您更倾向于使用 Amazon SES，请注意以下几点：

•	您需要注册 [Amazon 网页服务(AWS)](http://aws.amazon.com/cn/)；

•	每一个在 **From** 或 **Return-Path**（返回地址）使用的发送者地址需要由主控者验证。您也可以验证整个域；

•	最初你是在一个受限制的模式。在被允许发送给任意收件人之前，您需要请求访问许可；

•	SES 可能受到指控。
