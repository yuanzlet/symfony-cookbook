# 如何在 Bundle 内部加载服务配置

在 Symfony 中，你可以通过应用很多的服务来找到你自己。这些服务可以在你的应用程序的 **app/config/** 目录下注册。但是如果当你想要分离 bundle 让它们应用于其它的工程中时，你就会想要让 bundle 自己拥有服务配置。这篇文章将会教你如何实现这个目标。  

## 建立一个 Extension 类

为了加载服务配置，你必须为你的 bundle 建立一个 Dependency Injection (DI) Extension。这个类在被自动检测方面有着一些约定。但是接下来你将会看到你可以如何让它转变成你喜好的样子。默认情况下，Extension 类必须符合下列约定：  

- 它必须位于 bundle 的 **DependencyInjection** 命名空间之中；  
- 它的名称要和 bundle 的一样，只是后缀由 **Bundle** 换成了 **Extension**（例如 AppBundle 的 Extension 类就会叫做 **AppExtension**，同时 AcmeHelloBundle 就会被叫做 **AcmeHelloExtension**）。  

Extension 类应当执行 [ExtensionInterface](http://api.symfony.com/2.7/Symfony/Component/DependencyInjection/Extension/ExtensionInterface.html)，但是通常情况下你只会简单的扩展 [Extension](http://api.symfony.com/2.7/Symfony/Component/DependencyInjection/Extension/Extension.html) 类：  

```
// src/Acme/HelloBundle/DependencyInjection/AcmeHelloExtension.php
namespace Acme\HelloBundle\DependencyInjection;

use Symfony\Component\HttpKernel\DependencyInjection\Extension;
use Symfony\Component\DependencyInjection\ContainerBuilder;

class AcmeHelloExtension extends Extension
{
    public function load(array $configs, ContainerBuilder $container)
    {
        // ... you'll load the files here later
    }
}
```  

## 手动注册 Extension 类

当不遵守这些约定时，你必须手动注册你的 Extension 类。为了完成这个，你必须重写 [Bundle::getContainerExtension()](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/Bundle/Bundle.html#build()) 方法来返回 extension 的实例：  

```
// ...
use Acme\HelloBundle\DependencyInjection\UnconventionalExtensionClass;

class AcmeHelloBundle extends Bundle
{
    public function getContainerExtension()
    {
        return new UnconventionalExtensionClass();
    }
}
```  

由于新的 Extension 类没有遵循命名规则，你还需要重写 [Extension::getAlias()](http://api.symfony.com/2.7/Symfony/Component/DependencyInjection/Extension/Extension.html#getAlias()) 来返回正确的 DI 别名。DI 别名是用来标识容器中的 bundle 的（例如在 **app/config/config.yml** 文件中）。默认设置下，这个是通过移除 **Extension** 后缀并且将类名称转换成下划线的形式来完成的。（例如 **AcmeHelloExtension** 的 DI 别名是 **acme_hello**）。  

## 使用 load() 方法

在 **load()** 方法中，所有关于这个 extension 的参数和服务都会被加载。这个方法不会获得实际的容器实例，但是是一个副本。这个容器只有从实际容器中取来的参数。在加载服务和参数之后，副本就会变成实际的容器，来保证所有的服务和变量也添加到实际的容器之中。  

在 **load（）** 方法中，你可以应用 PHP 代码来进行服务定义的注册，但是通常的做法都是将这些定义放到配置文件之中（使用 Yaml，XML，或者 PHP 格式）。幸运的是，你可以在 extension 中使用文件加载器。  

目前来讲，假设在你的 bundle 中的 **Resources/config** 目录中有一个名为 **services.xml** 的文件，你加载的方法如下所示：  

```
use Symfony\Component\DependencyInjection\Loader\XmlFileLoader;
use Symfony\Component\Config\FileLocator;

// ...
public function load(array $configs, ContainerBuilder $container)
{
    $loader = new XmlFileLoader(
        $container,
        new FileLocator(__DIR__.'/../Resources/config')
    );
    $loader->load('services.xml');
}
```  

其它可用的加载器是 **YamlFileLoader, PhpFileLoader 和 IniFileLoader**。

> **IniFileLoader** 只能被用于加载参数并且它只能将参数作为字符串加载。  

## 使用配置更改服务

Extension 也是处理 bundle 的特定的配置（例如 **app/config/config.yml** 中的配置）的类。 你可以通过阅读“[如何为一个 Bundle 创建友好的配置](http://symfony.com/doc/current/cookbook/bundles/configuration.html)”这篇文章来更加深入地学习。  