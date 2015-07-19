# 缓存包含 CSRF 保护表单的页面

CSRF 令牌意味着每一个用户都是不同的。这就是为什么如果你尝试缓存带有表单的页面时需要注意这点。  

获取更多有关于 Symfony 中 CSRF 保护如何运作的信息，请查阅 [CSRF 保护](http://symfony.com/doc/current/book/forms.html#forms-csrf)。  

## 为什么缓存带有 CSRF 令牌的页面会出现问题

典型地，每个用户都被分配了一个特定的 CSRF 令牌，这个令牌储存在校验的会话中。这就意味着如果你*确实*想要缓存一个带有 CSRF 令牌的页面，你只能缓存*第一个*用户的 CSRF 令牌。当一个用户提交表单时，令牌就不会和储存在校验的会话中的令牌相匹配并且所有用户（除了第一个用户）都会在提交表单时 CSRF 校验失败。  

实际上，很多的反向代理（如 Varnish）都拒绝缓存包含 CSRF 保护表单的页面。这是因为 cookie 被发出从而为了阻止 PHP 会话打开并且 Varnish 的默认行为是不用 cookie 缓存 HTTP 请求。  

## 如何在应用 CSRF 保护的情况下缓存大多数页面

为了缓存包含 CSRF 令牌的页面，你可以使用像 [ESI fragments](http://symfony.com/doc/current/book/http_cache.html#edge-side-includes) 一样的更先进的缓存技术，这样你可以缓存整个页面并且将表单嵌入到 ESI tag 并且根本不用缓存。  

另外的一个选择就是通过未缓存的 AJAX 请求加载表单，但是缓存其余的 HTML 响应。  

或者你甚至可以通过未缓存的 AJAX 请求加载 CSRF 令牌并且用它替换表单中失效的值。  