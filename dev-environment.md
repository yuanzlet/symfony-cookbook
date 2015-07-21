# 如何在开发时使用电子邮件

当开发应用程序发送电子邮件时，您通常不希望在开发过程中发送电子邮件到指定的收件人。如果您使用的是 SwiftmailerBundle 的 Symfony，您可以轻松地通过配置设置完成，不用对您的应用程序编码做任何改变。当遇到在开发时使用电子邮件的情况，主要有两个选择：（a）完全禁用发送电子邮件或者（b）所有电子邮件发送到一个特定的地址(可选的例外)。

## 禁用发送

您可以禁用发送电子邮件通过设置 **disable_delivery** 选项为 **true**。这是标准分配中 **test** 环境中的默认值。如果您在 **test** 具体配置中做这个，那么当您运行测试的时候邮件不会被发送，但是在 **prod** 和 **dev** 环境下就会继续被发送。

YAML

```
# app/config/config_test.yml
swiftmailer:
    disable_delivery:  true
```

XML

```
<!-- app/config/config_test.xml -->

<!--
    xmlns:swiftmailer="http://symfony.com/schema/dic/swiftmailer"
    http://symfony.com/schema/dic/swiftmailer http://symfony.com/schema/dic/swiftmailer/swiftmailer-1.0.xsd
-->

<swiftmailer:config
    disable-delivery="true" />
```

PHP

```
// app/config/config_test.php
$container->loadFromExtension('swiftmailer', array(
    'disable_delivery'  => "true",
));
```

如果您也想在 **dev** 环境中禁用转发，仅在 **config_dev.yml** 文件中添加一个相同的配置。

## 发送到一个指定位置

您也可以选择将所有的电子邮件发送到一个具体地址，而不是当发送消息时实际指定的地址。这可以通过选项 **delivery_address** 完成：

YAML

```
# app/config/config_dev.yml
swiftmailer:
    delivery_address: dev@example.com
```

XML

```
<!-- app/config/config_dev.xml -->

<!--
    xmlns:swiftmailer="http://symfony.com/schema/dic/swiftmailer"
    http://symfony.com/schema/dic/swiftmailer http://symfony.com/schema/dic/swiftmailer/swiftmailer-1.0.xsd
-->

<swiftmailer:config delivery-address="dev@example.com" />
```

PHP

```
// app/config/config_dev.php
$container->loadFromExtension('swiftmailer', array(
    'delivery_address'  => "dev@example.com",
));
```

现在，假设您正在发送电子邮件到 **recipient@example.com**。

```
public function indexAction($name)
{
    $message = \Swift_Message::newInstance()
        ->setSubject('Hello Email')
        ->setFrom('send@example.com')
        ->setTo('recipient@example.com')
        ->setBody(
            $this->renderView(
                'HelloBundle:Hello:email.txt.twig',
                array('name' => $name)
            )
        )
    ;
    $this->get('mailer')->send($message);

    return $this->render(...);
}
```

在 **dev** 环境中，电子邮件将被发送到 **dev@example.com**。Swfit Mailor 会为电子邮件添加一个额外的标题，**X-Swift-To**，包含替代的地址，所以您仍可以看到谁会收到电子邮件。

除了 **to** 地址之外，这也会阻止电子邮件发送到任何设置为 **CC** 或者 **BCC** 的地址。Swift Mailer 会为覆盖它们的地址的电子邮件添加额外的标题。**CC** 和 **BCC** 地址的标题分别是 **X-Swift-Cc** 和 **X-Swift-Bcc**。

## 发送到指定地址但有例外

假设您想将所有的电子邮件重新发送到一个指定的地址，（像上面的到 **dev@example.com**）。但是您毕竟可能想查阅一些发送到指定地址的电子邮件，并且没有被重新发送（即使它在 dev 环境中）。这可以通过添加选项 **delivery_whitelist** 完成：

YAML

```
# app/config/config_dev.yml
swiftmailer:
    delivery_address: dev@example.com
    delivery_whitelist:
       # all email addresses matching this regex will *not* be
       # redirected to dev@example.com
       - "/@specialdomain.com$/"

       # all emails sent to admin@mydomain.com won't
       # be redirected to dev@example.com too
       - "/^admin@mydomain.com$/"
```

XML

```
<!-- app/config/config_dev.xml -->

<?xml version="1.0" charset="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:swiftmailer="http://symfony.com/schema/dic/swiftmailer">

<swiftmailer:config delivery-address="dev@example.com">
    <!-- all email addresses matching this regex will *not* be redirected to dev@example.com -->
    <swiftmailer:delivery-whitelist-pattern>/@specialdomain.com$/</swiftmailer:delivery-whitelist-pattern>

    <!-- all emails sent to admin@mydomain.com won't be redirected to dev@example.com too -->
    <swiftmailer:delivery-whitelist-pattern>/^admin@mydomain.com$/</swiftmailer:delivery-whitelist-pattern>
</swiftmailer:config>
```

PHP

```
// app/config/config_dev.php
$container->loadFromExtension('swiftmailer', array(
    'delivery_address'  => "dev@example.com",
    'delivery_whitelist' => array(
        // all email addresses matching this regex will *not* be
        // redirected to dev@example.com
        '/@specialdomain.com$/',

        // all emails sent to admin@mydomain.com won't be
        // redirected to dev@example.com too
        '/^admin@mydomain.com$/',
    ),
));
```

在以上例子中，所有的电子邮件消息会重新发送到 **dev@example.com**，除了发送到 **admin@mydomain.com** 地址的消息或者其他属于 **specialdomain.com** 域的电子邮件地址的消息，将会正常发送。

## 从 Web 调试工具栏查看

当您在 **dev** 环境中使用 web 调试工具栏时，您可以在一个单一的反应中查看任何邮件。工具栏上的电子邮件图标会显示发送了多少电子邮件。如果您点击它，一个报告显示发送电子邮件的细节。

如果您发送一封电子邮件，然后立即重定向到另一个网页，网页调试工具栏将不显示电子邮件图标或在下一页面不显示报告。

反而，您可以在 **config_dev.yml** 文件中设置选项 **intercept_redirects**  为 **true**，将会导致重新转到停止并允许您打开发送电子邮件的细节的报告。

YAML

```
# app/config/config_dev.yml
web_profiler:
    intercept_redirects: true
```

XML

```
<!-- app/config/config_dev.xml -->

<!--
    xmlns:webprofiler="http://symfony.com/schema/dic/webprofiler"
    xsi:schemaLocation="http://symfony.com/schema/dic/webprofiler
    http://symfony.com/schema/dic/webprofiler/webprofiler-1.0.xsd">
-->

<webprofiler:config
    intercept-redirects="true"
/>
```

PHP

```
// app/config/config_dev.php
$container->loadFromExtension('web_profiler', array(
    'intercept_redirects' => 'true',
));
```

另外，您可以在重新定向和由之前请求的提交 URL 搜索（例如 **/contact/handle**）之后，打开分析器。分析器的搜索功能允许您为任何过去的请求加载分析器信息。

