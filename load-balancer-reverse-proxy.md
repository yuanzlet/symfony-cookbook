# 如何配置 Symfony 使其工作在负载均衡和反转代理 

当你部署应用程序时，你可能将其部署在一个负载平衡器（如 AWS 弹性负载平衡器）或反向代理（如Varnish [缓存](http://symfony.com/doc/current/book/http_cache.html)）之后。  

在大多数情况下，这不会在 Symfony 中造成任何问题。 但是，当一个请求通过一个代理时，必须保证要求发送的信息使用标准 Forwarded 头或非标准的特殊 X-Forwarded-* 头。 例如，并不读  REMOTE_ADDR 报头（这将是你现在的反向代理的IP地址），用户的真实IP将被存储在一个标准的 Forwarded 像 for：="..." 头或非标准 X-Forwarded-For 报文头。
 
>  2.7 在 Symfony 2.7 中介绍了 Forwarded 头的相关内容。

如果你不配置 Symfony 去查看这些头文件，你会得到不正确的客户端的 IP 地址，无论客户端是否是通过 HTTPS 连接，客户端的端口和主机名是否被请求。  

## 解决方案：可信代理（trusted_proxies）

这是没有问题的，但是你需要告诉 Symfony 发生了这样的事情，然后反向代理 IP 地址做这件事：  

YAML:
```
# app/config/config.yml
# ...
framework:
    trusted_proxies:  [192.0.0.1, 10.0.0.0/8]
```

XML:
```

<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:framework="http://symfony.com/schema/dic/symfony"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                        http://symfony.com/schema/dic/symfony http://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

    <framework:config trusted-proxies="192.0.0.1, 10.0.0.0/8">
        <!-- ... -->
    </framework>
</container>

```

PHP:
```
// app/config/config.php
$container->loadFromExtension('framework', array(
    'trusted_proxies' => array('192.0.0.1', '10.0.0.0/8'),
));
```

在这个例子中，你说你的反向代理（或代理们）的IP地址为 192.0.0.1 或匹配使用 CIDR 符号的 IP 地址范围是 10.0.0.0/8 。更多的细节请看 [framework.trusted_proxies](http://symfony.com/doc/current/reference/configuration/framework.html#reference-framework-trusted-proxies) 选项。

这是这样！Symfony 会现在查看正确的报文头并去获得客户端的 IP 地址，主机，端口和请求是否使用 HTTPS 等信息。  

## 但是如果我的反向代理的 IP 地址是不断变化的怎么办！

一些反向代理（如亚马逊的弹性负载平衡器）没有一个静态 IP 地址或地址范围以便你可以用 CIDR 标记目标。在这种情况下，你需要-*非常小心*-相信*所有的*代理。

1. 配置您的 Web 服务器（们）不回应除你的流量负载均衡以外的其他任何客户的任何传输信息。对于 AWS，这可以通过安全组来实现。
2. 一旦你保证流量只会来自你信任的反向代理服务器，配置 Symfony 使之始终信任传入的请求。这是在您的前端控制器实现的： 
```
// web/app.php

// ...
Request::setTrustedProxies(array($request->server->get('REMOTE_ADDR')));

$response = $kernel->handle($request);
// ...
```

3. 确保在你的 app/config/config.yml 设置中的 trusted_proxies 没有被改变设置否则将改写上面的 setTrustedProxies 的调用。  

这是这样！这一步是至关重要的，您可以防止流量从所有不受信任的来源进入。如果你允许外部交通，他们可以“欺骗”自己的真实IP地址等信息。  

## 我的反向代理使用非标准（非 X-Forwarded）头
虽然 [RFC 7239](http://tools.ietf.org/html/rfc7239) 近期定义了标准的转发头来涵盖所有的代理信息，但大多数反向代理仍将信息存储在非标 X-Forwarded-* 头中。  

但是如果你的反向代理使用其他非标准的头文件，你可以配置这些（见"[Trusting Proxies](http://symfony.com/doc/current/components/http_foundation/trusting_proxies.html)"）。  

这样的代码需要在你的前端控制器激活（如 Web 应用，PHP）。

这项工作的许可为 Creative Commons Attribution-Share Alike 3.0 Unported [License](http://creativecommons.org/licenses/by-sa/3.0/)
