# 限制 Session 元数据的写入

PHP 会话的默认行为是不管会话有没有改变都保存会话。在 Symfony 中，每当会话被接入，可以用来确定会话的年龄（age）和空闲时间的元数据都会被记录（会话产生的/最后使用的）。

如果出于性能原因您想要限制会话保存的频率，这个功能在使元数据保持相对精确的情况下，调整元数据更新的间隔和减少会话保存的频率。如果其他会话数据被更改，会话始终保存。

您可以用设置 **framework.session.metadata_update_threshold** 一个大于 0 秒的值方法告诉 Symfony 不要更新元数据直到距“会话最后一次更新”时间过去了一段特定时间。

```YAML
framework:
    session:
        metadata_update_threshold: 120
```

```XML
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:framework="http://symfony.com/schema/dic/symfony"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                        http://symfony.com/schema/dic/symfony http://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

    <framework:config>
        <framework:session metadata-update-threshold="120" />
    </framework:config>

</container>
```

```PHP
$container->loadFromExtension('framework', array(
    'session' => array(
        'metadata_update_threshold' => 120,
    ),
));
```

> PHP 默认行为是不管会话有没有改变都保存会话。当使用 **framework.session.metadata_update_threshold** 时，Symfony 会将会话处理程序裹入（wrap）（配置在 **framework.session.handler_id** 中） WriteCheckSessionHandler。这将阻止任何没有被修改的会话写入。

> 请注意，如果每个请求（request）都没有写入会话，它可能会比通常更早的回收（garbage collected）。这意味着您的用户可能会比预期提前注销。
