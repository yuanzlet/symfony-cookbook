# 配置 Session 文件的保存目录

在默认情况下, Symfony Standard Edition 应用了 **php.ini** 这个全局值来为 **session.save_handler** 和 **session.save_path** 决定选择哪里来存储会话数据。这都是由于以下的配置。

YAML:

```YAML
# app/config/config.yml
framework:
    session:
        # handler_id set to null will use default session handler from php.ini
        handler_id: ~
```

XML:

```XML
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:framework="http://symfony.com/schema/dic/symfony"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/symfony
        http://symfony.com/schema/dic/symfony/symfony-1.0.xsd"
>
    <framework:config>
        <!-- handler-id set to null will use default session handler from php.ini -->
        <framework:session handler-id="null" />
    </framework:config>
</container>
```

PHP:

```PHP
// app/config/config.php
$container->loadFromExtension('framework', array(
    'session' => array(
        // handler_id set to null will use default session handler from php.ini
        'handler_id' => null,
    ),
));
```

由于有这个配置，想要改变您的会话元数据的储存位置的话就全部都要依靠 **php.ini** 配置来实现了。

然而，如果您有以下的配置的话，Symfony 将把会话数据存储在 **%kernel.cache_dir%/sessions** 这个缓存目录的文件夹里。这意味着如果您清空缓存的话，所有的最近会话也将被删除。

YAML:

```YAML
# app/config/config.yml
framework:
    session: ~
```

XML:

```XML
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:framework="http://symfony.com/schema/dic/symfony"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/symfony
        http://symfony.com/schema/dic/symfony/symfony-1.0.xsd"
>
    <framework:config>
        <framework:session />
    </framework:config>
</container>
```

PHP:

```PHP
// app/config/config.php
$container->loadFromExtension('framework', array(
    'session' => array(),
));
```

利用另一个目录来存储会话数据是一个当您清空缓存时确保最近的会话没有丢失的常用方法。

> 使用不同的会话保存操作是一个很好的（可能更复杂些）在 Symfony 中可提供的管理会话的方法。到  [Configuring Sessions and Save Handlers](http://symfony.com/doc/current/components/http_foundation/session_configuration.html) 可以来看看会话保存处理程序的讨论。在 cookbook 中还有个还有个链接是关于在[数据库](http://symfony.com/doc/current/cookbook/configuration/pdo_session_storage.html)中存储会话的。

如果您想要改变 Symfony 存储会话数据的目录的话，您只需要改变框架配置就可以了。在下面的例子中，会把会话目录改变到 **app/sessions**：

YAML:

```YAML
# app/config/config.yml
framework:
    session:
        handler_id: session.handler.native_file
        save_path: "%kernel.root_dir%/sessions"
```

XML:

```XML
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:framework="http://symfony.com/schema/dic/symfony"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/symfony
        http://symfony.com/schema/dic/symfony/symfony-1.0.xsd"
>
    <framework:config>
        <framework:session handler-id="session.handler.native_file"
            save-path="%kernel.root_dir%/sessions"
        />
    </framework:config>
</container>
```

PHP:

```PHP
// app/config/config.php
$container->loadFromExtension('framework', array(
    'session' => array(
        'handler_id' => 'session.handler.native_file',
        'save_path'  => '%kernel.root_dir%/sessions',
    ),
));
```