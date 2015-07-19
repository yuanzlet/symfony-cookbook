# 如何使用 Varnish 加速我的Web站点

由于 Symfony 的缓存使用的是标准的 HTTP 缓存头文件，[Symfony 反向代理](http://symfony.com/doc/current/book/http_cache.html#symfony-gateway-cache) 可以很容易地就被其它的反向代理取代。[Varnish](https://www.varnish-cache.org/) 是一个强大的，开源的，HTTP 加速器可以提供缓存内容加速并且包括支持 [Edge Side Includes](http://symfony.com/doc/current/book/http_cache.html#edge-side-includes)。  

## 使 Symfony 信任反向代理

Varnish 自动在请求中将 IP 地址作为 **X-Forwarded-For** 转发并且去掉 **X-Forwarded-Proto** 的头信息。如果你不将 Varnish 设置为可信的代理，Symfony 将会把所有的从 Varnish 主机发出的请求看作是不安全的 HTTP 连接而不是真正的客户。  

记得在 Symfony 的配置中设置 [framework.trusted_proxies](http://symfony.com/doc/current/reference/configuration/framework.html#reference-framework-trusted-proxies) 以便于 Varnish 可以被看做是可信的代理并且要使用 [X-Forwarded](http://symfony.com/doc/current/cookbook/cache/varnish.html#varnish-x-forwarded-headers) 头信息。  

## 路由和 X-FORWARDED 头文件

为了确保 Symfony 路由使用 Varnish 产生正确的链接地址，要在 **X-Forwarded-Port** 的头信息中添加正确的端口数字。这个端口取决于你的设置。  

假设外部连接来自于默认的 HTTP 80 端口。对于 HTTPS 连接，有另外一个代理（由于 Varnish 并不是自己代理 HTTPS ）位于默认的 HTTPS 端口 443 处理 SSL 终止然后将请求作为 HTTP 请求转发到 Varnish 伴随一个 **X-Forwarded-Proto** 文件头。在这种情况下，向你的 Varnish 添加如下配置：  

```
sub vcl_recv {
    if (req.http.X-Forwarded-Proto == "https" ) {
        set req.http.X-Forwarded-Port = "443";
    } else {
        set req.http.X-Forwarded-Port = "80";
    }
}
```  

## Cookies 和缓存

默认设置下，一个安全的缓存代理不会缓存任何东西当请求发送的信息中含有 [cookies 或者一个基本的身份验证头文件](http://symfony.com/doc/current/book/http_cache.html#http-cache-introduction)。这是因为页面的内容应该是依赖 cookie 的值或者身份验证头文件。  

如果你确定知道后端从来不使用会话或者基本的身份验证，Varnish  从请求中移除了相应的头文件组织客户绕过缓存。在实践中，你网站的部分将会需要会话，例如使用 [CSRF Protection](http://symfony.com/doc/current/book/forms.html#forms-csrf) 的形式。在这种情况下，确保[只有在需要的时候才开启一个会话](http://symfony.com/doc/current/cookbook/session/avoid_session_start.html)并且在不再需要这个会话的时候清除它。或者，你也可以阅读[包含 CSRF Protection 表单的缓存页面](http://symfony.com/doc/current/cookbook/cache/form_csrf_caching.html)。  

由 JavaScript 创建的 Cookies 只能应用于前端，例如当使用 Google Analytics 时，还是要发送到服务器。这些 cookies 和后端没有关系并且也不应该影响缓存决策。设置你的 Varnish 缓存来[清除 cookies 的头文件](https://www.varnish-cache.org/trac/wiki/VCLExampleRemovingSomeCookies)。你想要留着会话 cookie，如果有一个的话，并且摆脱其它的 cookie，这样如果没有活动的会话页面就可以缓存了。除非你更改 PHP 的默认配置，否则你的会话 cookie 也会有相同的 PHPSESSID：  

```
sub vcl_recv {
    // Remove all cookies except the session ID.
    if (req.http.Cookie) {
        set req.http.Cookie = ";" + req.http.Cookie;
        set req.http.Cookie = regsuball(req.http.Cookie, "; +", ";");
        set req.http.Cookie = regsuball(req.http.Cookie, ";(PHPSESSID)=", "; \1=");
        set req.http.Cookie = regsuball(req.http.Cookie, ";[^ ][^;]*", "");
        set req.http.Cookie = regsuball(req.http.Cookie, "^[; ]+|[; ]+$", "");

        if (req.http.Cookie == "") {
            // If there are no more cookies, remove the header to get page cached.
            remove req.http.Cookie;
        }
    }
}

```  

> 如果每一个用户的内容不同，但是依赖于用户的角色，解决办法就是分开每组的缓存。这个模式由 [User Context](http://foshttpcachebundle.readthedocs.org/en/latest/features/user-context.html) 名称下的 [FOSHttpCacheBundle](http://foshttpcachebundle.readthedocs.org/) 负责实现和解释。  

## 确保一致的缓存行为

Varnish 根据你的应用程序发出的缓存头文件来决定如何缓存内容。然而，Varnish 4 之前的 versions 不遵循 **Cache-Control: no-cache**, **no-store** 和 **private**。为了确保一致的行为，如果你还在用 Varnish 3 的话就要使用如下的配置：  

```
sub vcl_fetch {
    /* By default, Varnish3 ignores Cache-Control: no-cache and private
       https://www.varnish-cache.org/docs/3.0/tutorial/increasing_your_hitrate.html#cache-control
     */
    if (beresp.http.Cache-Control ~ "private" ||
        beresp.http.Cache-Control ~ "no-cache" ||
        beresp.http.Cache-Control ~ "no-store"
    ) {
        return (hit_for_pass);
    }
}
```

> 你可以在一个 VCL 文件中看到 Varnish 的默认行为：Varnish 3 的 [default.vcl](https://www.varnish-cache.org/trac/browser/bin/varnishd/default.vcl?rev=3.0)，Varnish 4 的 [builtin.vcl](https://www.varnish-cache.org/trac/browser/bin/varnishd/builtin.vcl?rev=4.0)。  

## 启用 Edge Side Includes (ESI)

就像在 [Edge Side Includes](http://symfony.com/doc/current/book/http_cache.html#edge-side-includes) 一节中介绍的那样，Symfony 侦测它是否和理解 ESI 保留的代理进行会话。当你使用 Symfony 的反向代理时，你不需要做任何事。但是如果你想要使用 Varnish 替代 Symfony 解决 ESI 标签，你需要在 Varnish 中进行一些设置。Symfony 使用 Akamai 描述的 Edge Architecture 中 **Surrogate-Capability** 头文件。  

> Varnish 只支持 ESI 标签下的 src 属性（onerror 和 alt 被忽视了）。  

首先，设置 Varnish 从而使得向后端应用程序转移的请求通过添加 **Surrogate-Capability** 头文件的方式获得它的 ESI 支持：  

```
sub vcl_recv {
    // Add a Surrogate-Capability header to announce ESI support.
    set req.http.Surrogate-Capability = "abc=ESI/1.0";
}
```  

> 头文件的 **abc** 部分不是很重要，除非你有需要宣传它们的能力的多个“代理”。阅读 [Surrogate-Capability Header](http://www.w3.org/TR/edge-arch) 获取更多信息。  

接下来，优化 Varnish 从而使得它仅仅在存在至少一个 ESI 标签时通过检查 Symfony 自动添加的 **Surrogate-Control** 头文件解析回复内容：  

Varnish4:

```
sub vcl_backend_response {
    // Check for ESI acknowledgement and remove Surrogate-Control header
    if (beresp.http.Surrogate-Control ~ "ESI/1.0") {
        unset beresp.http.Surrogate-Control;
        set beresp.do_esi = true;
    }
}
```  

Varnish3:
```
sub vcl_fetch {
    // Check for ESI acknowledgement and remove Surrogate-Control header
    if (beresp.http.Surrogate-Control ~ "ESI/1.0") {
        unset beresp.http.Surrogate-Control;
        set beresp.do_esi = true;
    }
}
```  

> 如果你遵循有关确保缓存一致性的建议，那些 VCL 函数已经存在。你只需将代码添加到函数的末尾，他们不会互相干扰。  

## 缓存失效

如果你想要缓存的内容经常更改并且还能为用户最近的版本服务，你需要使那些内容失效。然而[缓存失效](http://tools.ietf.org/html/rfc2616#section-13.10)允许你在过期前清除你的代理内容，这会增加你的缓存设置的复杂性。  

> 开源的 [FOSHttpCacheBundle](http://foshttpcachebundle.readthedocs.org/) 将缓存失效的痛苦通过帮助你组织你的缓存和失效设置减轻。
> [FOSHttpCacheBundle](http://foshttpcachebundle.readthedocs.org/) 的文档解释了如何设置 Varnish 和其它的反向代理的缓存失效。  