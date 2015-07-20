# 在遗留的应用上使用 Symfony Session

> **2.3** 能够与传统 PHP 会话结合的能力在 Symfony 2.3 中介绍。

如果您整合 Symfony 的全栈框架和从 **session_start()** 开始会话的遗留应用程序的话，通过使用 PHP Bridge session，可以使您仍然能够使用 Symfony 会话管理。

如果应用程序已设置它自己的 PHP 保存处理程序，您可以使 **handler_id** 指定为空：

```YAML
framework:
    session:
        storage_id: session.storage.php_bridge
        handler_id: ~
```

```XML
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:framework="http://symfony.com/schema/dic/symfony">

    <framework:config>
        <framework:session storage-id="session.storage.php_bridge"
            handler-id="null"
        />
    </framework:config>
</container>
```

```PHP
$container->loadFromExtension('framework', array(
    'session' => array(
        'storage_id' => 'session.storage.php_bridge',
        'handler_id' => null,
));
```

否则，如果问题仅仅是你不能避免应用程序以 **session_start()** 启动会话，你仍可以通过指定保存处理程序来使用一个基于 Symfony 的会话保存处理程序，就像下面的例子那样：


```YAML
framework:
    session:
        storage_id: session.storage.php_bridge
        handler_id: session.handler.native_file
```

```XML
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:framework="http://symfony.com/schema/dic/symfony">

    <framework:config>
        <framework:session storage-id="session.storage.php_bridge"
            handler-id="session.storage.native_file"
        />
    </framework:config>
</container>
```

```PHP
$container->loadFromExtension('framework', array(
    'session' => array(
        'storage_id' => 'session.storage.php_bridge',
        'handler_id' => 'session.storage.native_file',
));
```

> 如果遗留应用程序需要它自己的会话保存处理程序，请不要重写它。代替的是设置 **handler_id: ~**。请注意，在会话开始后，会话保存处理程序将不能更改。如果应用程序在 Symfony 初始化之前开始对话，会话保存程序会早已设置好。这种情况下，你需要 **handler_id: ~**。只有当您确认遗留应用程序可以无副作用的使用 Symfony 保存处理程序并且会话不在 Symfony 初始化之前启动的情况下重写保存程序。

从 [Integrating with Legacy Sessions](http://symfony.com/doc/current/components/http_foundation/session_php_bridge.html) 获取更多细节。
