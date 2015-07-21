# 如何使用 Gmail 发送邮件

在发展过程中，不是使用一个常规得 SMTP 服务器来发送邮件，您可能发现使用 Gmail 更简单且更实用。SwiftmailerBundle 使它变得相当简单。

不是使用您常规的 Gmail 账户，理所当然推荐的是您创建一个特别的账户。

在开发配置文件中，改变 **transport** 设置到 **gmail** 并设置 **username** 和 **password** 到 Google 证书上：

YAML

```
# app/config/config_dev.yml
swiftmailer:
    transport: gmail
    username:  your_gmail_username
    password:  your_gmail_password
```

XML

```
<!-- app/config/config_dev.xml -->
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
        transport="gmail"
        username="your_gmail_username"
        password="your_gmail_password"
    />
</container>
```

PHP

```
// app/config/config_dev.php
$container->loadFromExtension('swiftmailer', array(
    'transport' => 'gmail',
    'username'  => 'your_gmail_username',
    'password'  => 'your_gmail_password',
));
```

您完成了！

如果您正使用 Symfony 标准版本，在 **parameters.yml** 中配置参数：

```
# app/config/parameters.yml
parameters:
    # ...
    mailer_transport: gmail
    mailer_host:      ~
    mailer_user:      your_gmail_username
    mailer_password:  your_gmail_password
```

**gmail** 传送只是一个使用 **smtp** 传送的快捷方式，并设置 **encryption**, **auth_mode** 和 **host**  同 Gmail 一起工作。

取决于您的 Gmail 账户设置，您可能收到应用程序验证错误。如果您的 Gmail 账户使用2步验证，您应该[生成一个应用程序密码](https://support.google.com/accounts/answer/185833)来供 **mailer_password** 参数使用。您也应该确保您[允许安全性较低的应用程序可以访问您的 Gmail 账户](https://support.google.com/accounts/answer/6010255)。


