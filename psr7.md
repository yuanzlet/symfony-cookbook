# The PSR-7 Bridge

> The PSR-7 bridge 将 [HttpFoundation](http://symfony.com/doc/current/components/http_foundation/index.html) 对象转换到实现 HTTP message 接口的对象，定义在 [PSR-7](http://www.php-fig.org/psr/psr-7/)。

## 安装

您可以用 2 种不同的方式安装组件：

- [通过 Composer 安装](http://symfony.com/doc/current/components/using_components.html)（**symfony/psr-http-message-bridge** on [Packagist](https://packagist.org/packages/symfony/psr-http-message-bridge)）
- 使用官方 Git 库（[https://github.com/symfony/psr-http-message-bridge](https://github.com/symfony/psr-http-message-bridge)）

Bridge 也需要一个 PSR-7 实现来允许将 HttpFoundation 对象转化为 PSR-7 对象。它为 [Zend Diactoros](https://github.com/zendframework/zend-diactoros) 提供原生支持。使用 Composer（**zendframework/zend-diactoros** on [Packagist](https://packagist.org/packages/symfony/psr-http-message-bridge)）或者查阅项目文档来安装它。

## 使用

### 从 HttpFoundation 对象到 PSR-7 的转换

Bridge 提供一个名为 [HttpMessageFactoryInterface](http://api.symfony.com/2.7/Symfony/Bridge/PsrHttpMessage/HttpMessageFactoryInterface.html) 的一个 factory 的接口，它可以从 HttpFoundation 对象构造实现 PSR-7 的接口的对象。它也提供了一个内部使用 Zend Diactoros 的默认的实现。

下面的代码片段说明了如何将一个 [Request](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/Request.html) 转换成一个 Zend Diactoros [ServerRequest](http://api.symfony.com/2.7/Zend/Diactoros/ServerRequest.html) 实现 [ServerRequestInterface](http://api.symfony.com/2.7/Psr/Http/Message/ServerRequestInterface.html) 接口：

```PHP
use Symfony\Bridge\PsrHttpMessage\Factory\DiactorosFactory;
use Symfony\Component\HttpFoundation\Request;

$symfonyRequest = new Request(array(), array(), array(), array(), array(), array('HTTP_HOST' => 'dunglas.fr'), 'Content');
// The HTTP_HOST server key must be set to avoid an unexpected error

$psr7Factory = new DiactorosFactory();
$psrRequest = $psr7Factory->createRequest($symfonyRequest);
```

现在从一个 [Response](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/Response.html) 到一个实现 [ResponseInterface](http://api.symfony.com/2.7/Psr/Http/Message/ResponseInterface.html) 接口的 Zend Diactoros [Response](http://api.symfony.com/2.7/Zend/Diactoros/Response.html)：

```PHP
use Symfony\Bridge\PsrHttpMessage\Factory\DiactorosFactory;
use Symfony\Component\HttpFoundation\Response;

$symfonyResponse = new Response('Content');

$psr7Factory = new DiactorosFactory();
$psrResponse = $psr7Factory->createResponse($symfonyResponse);
```

### 转换对象实现 PSR-7 到 HttpFoundation 的接口

另一方面，bridge 提供一个名为 [HttpFoundationFactoryInterface](http://api.symfony.com/2.7/Symfony/Bridge/PsrHttpMessage/HttpFoundationFactoryInterface.html) 的一个 factory 的接口，它可以从实现 PSR-7 的接口的对象构造 HttpFoundation 对象。

下一段代码解释如何将一个实现 [ServerRequestInterface](http://api.symfony.com/2.7/Psr/Http/Message/ServerRequestInterface.html) 接口的对象转变为一个 [Request](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/Request.html) 实例。

```PHP
use Symfony\Bridge\PsrHttpMessage\Factory\HttpFoundationFactory;

// $psrRequest is an instance of Psr\Http\Message\ServerRequestInterface

$httpFoundationFactory = new HttpFoundationFactory();
$symfonyRequest = $httpFoundationFactory->createRequest($psrRequest);
```

从一个实现 [ResponseInterface](http://api.symfony.com/2.7/Psr/Http/Message/ResponseInterface.html) 的对象到一个 [Response](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/Response.html) 实例：

```PHP
use Symfony\Bridge\PsrHttpMessage\Factory\HttpFoundationFactory;

// $psrResponse is an instance of Psr\Http\Message\ResponseInterface

$httpFoundationFactory = new HttpFoundationFactory();
$symfonyResponse = $httpFoundationFactory->createResponse($psrResponse);
```
