# 如何为一个 Bundle 创建友好的配置

如果你打开你的应用程序配置文件（通常是 **app/config/config.yml**），你将会看到一些不同设置的部分，例如 **framework**, **twig** 和 **doctrine**。这些配置的每一个都有特定的 bundle，允许基于你的设置定义高级的选项并且让 bundle 设置为低级的，复杂的。  

举例来说，下列的配置使得 FrameworkBundle 启用表集合，这个包含了一些服务以及其它相关组件的集合的定义：  

YAML:

```YAML
framework:
    form: true
```  

XML:

```XML
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:framework="http://symfony.com/schema/dic/symfony"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/symfony
        http://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

    <framework:config>
        <framework:form />
    </framework:config>
</container>
```  

PHP:

```PHP
$container->loadFromExtension('framework', array(
    'form' => true,
));
```  

> **使用参数设置你的 Bundle**
> 如果你不打算在你的工程中间共享 bundle，那么用这么高级的配置方法就是没有意义的。因为你只是在一个工程中使用 bundle，你每次都更改服务设置就好。
> 如果你*确实*想在 **config.yml** 中设置一些东西，你可以在那里创建一个参数并且在其它地方应用这个参数。  

## 使用 Bundle Extension

基本的思想是不要让用户重写个人的参数，而是可以让用户设置一些，尤其是创建，选项。作为 bundle 的开发者，你以后会通过分析配置以及加载 “Extension” 类中的正确的服务和参数。  

作为一个例子，试想你正在创建一个公共 bundle，这个 bundle 集合了 Twitter 等等。为了能够再次利用你的 bundle，你必须使得 **client_id** 和 **client_secret** 变量可配置。你的 bundle 可能如下所示：  

YAML:

```YAML
# app/config/config.yml
acme_social:
    twitter:
        client_id: 123
        client_secret: $ecret
```  

XML:

```XML
<!-- app/config/config.xml -->
<?xml version="1.0" ?>

<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:acme-social="http://example.org/dic/schema/acme_social"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">

   <acme-social:config>
       <twitter client-id="123" client-secret="$ecret" />
   </acme-social:config>

   <!-- ... -->
</container>
```  

PHP:

```PHP
// app/config/config.php
$container->loadFromExtension('acme_social', array(
    'client_id'     => 123,
    'client_secret' => '$ecret',
));
``` 

> 在[如何在 Bundle 内部加载服务配置](http://symfony.com/doc/current/cookbook/bundles/extension.html)这篇文章中深入学习。

> 如果一个 bundle 提供 Extension 类，那么你就*不*应该重写那个 bundle 中的任何服务容器参数。这个的思想是如果存在 Extension 类，所有的设置都应该是配置的，是这个类使得所有的选项可配置。换句话说，extension 类定义了所有的公共配置设置，这些配置应当向后兼容。  

> 在依赖注入容器中处理参数的问题可以阅读[在依赖注入类中使用参数](http://symfony.com/doc/current/cookbook/configuration/using_parameters_in_dic.html)。  

### 处理 $configs 数组

先说重要的，你必须创建一个 extension 类，就像[如何在 Bundle 内部加载服务配置](http://symfony.com/doc/current/cookbook/bundles/extension.html)这篇文章中说的那样。  

无论何时当一个用户在设置文件中包含 **acme_social** 键值（这是一个 DI 别名）时，在它之下的配置就会添加到一个配置的数组中并且传到你的 extension 的 **load()** 方法中（Symfony 会自动将 XML 和 YAML 转换到数组）。  

在以前部分配置的例子中，数组传递到你的 **load()** 方法如下所示：  

```
array(
    array(
        'twitter' => array(
            'client_id' => 123,
            'client_secret' => '$ecret',
        ),
    ),
)
```  

你会注意到这是一个*数组的数组*，不是一个简单的扁平的具有配置值的数组。这是有意为之的，实际上它允许 Symfony 分析几个配置资源。举例来说，如果 **acme_social** 出现在其它的配置文件比如 **config_dev.yml** 中并且在它下面有不同的值，进来的数组可能就如下所示：  

```
array(
    // values from config.yml
    array(
        'twitter' => array(
            'client_id' => 123,
            'client_secret' => '$secret',
        ),
    ),
    // values from config_dev.yml
    array(
        'twitter' => array(
            'client_id' => 456,
        ),
    ),
)
```  

两个数组的顺序取决于哪个先被设置。  

但是不用担心！Symfony 的设置组件将会帮助你合并这些值，提供默认的值并且对用户的错误的设置进行校验。下面介绍它是如何运行的。在 **DependencyInjection** 目录下创建一个 **Configuration** 类然后创建一个定义了你的 bundle 配置结构的树。  

**Configuration** 类处理简单的配置如下所示：  

```
// src/Acme/SocialBundle/DependencyInjection/Configuration.php
namespace Acme\SocialBundle\DependencyInjection;

use Symfony\Component\Config\Definition\Builder\TreeBuilder;
use Symfony\Component\Config\Definition\ConfigurationInterface;

class Configuration implements ConfigurationInterface
{
    public function getConfigTreeBuilder()
    {
        $treeBuilder = new TreeBuilder();
        $rootNode = $treeBuilder->root('acme_social');

        $rootNode
            ->children()
                ->arrayNode('twitter')
                    ->children()
                        ->integerNode('client_id')->end()
                        ->scalarNode('client_secret')->end()
                    ->end()
                ->end() // twitter
            ->end()
        ;

        return $treeBuilder;
    }
}
```  

> **Configuration** 类可以比上面展示的复杂很多，支持“原型”节点，高级的验证，特定 XML 的正常化以及高级合并。你可以通过阅读[组件设置文档](http://symfony.com/doc/current/components/config/definition.html)来学习更多相关知识。你也可以通过实际检查一些核心的 Configuration 类，比如 [FrameworkBundle Configuration](https://github.com/symfony/symfony/blob/master/src/Symfony/Bundle/FrameworkBundle/DependencyInjection/Configuration.php) 中的或者 [TwigBundle Configuration](https://github.com/symfony/symfony/blob/master/src/Symfony/Bundle/TwigBundle/DependencyInjection/Configuration.php) 中的。  

这个类可以在你的 **load()** 方法下应用来合并配置以及强力验证（例如如果附加选项通过，异常就会被抛出）：  

```
public function load(array $configs, ContainerBuilder $container)
{
    $configuration = new Configuration();

    $config = $this->processConfiguration($configuration, $configs);
    // ...
}
```  

**processConfiguration()** 方法使用了你已经在 **Configuration** 类中定义的配置树来验证，正常化以及将配置数组合并在一起。  

>代替每次在你提供的配置选项的扩展中调用 **processConfiguration()**，你可能想要使用 **ConfigurableExtension** 来帮你自动完成这件事：  
>```
>// src/Acme/HelloBundle/DependencyInjection/AcmeHelloExtension.php
namespace Acme\HelloBundle\DependencyInjection;

use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\HttpKernel\DependencyInjection\ConfigurableExtension;

class AcmeHelloExtension extends ConfigurableExtension
{
    // note that this method is called loadInternal and not load
    protected function loadInternal(array $mergedConfig, ContainerBuilder $container)
    {
        // ...
    }
}
>```  
>这个类使用了方法来获取 Configuration 实例，如果你的 Configuration 类不叫做 **Configuration** 或者它没有和你的扩展放在同一个命名空间内，那么你应当重写它。  

>自己处理 Configuration
>使用 Config 组件是可选择的。**load()** 方法获得了一个具有配置值的数组。你可以简单地自己分析这些数组（例如重写配置并且应用 [isset](http://php.net/manual/en/function.isset.php) 来检测值是否存在）。你要知道支持 XML 是很困难的。
>```
>public function load(array $configs, ContainerBuilder $container)
>{
    $config = array();
    // let resources override the previous set value
    foreach ($configs as $subConfig) {
        $config = array_merge($config, $subConfig);
    }

    // ... now use the flat $config array
>}
>```  

## 修正另外的 bundle 的配置

如果你有很多 bundle 它们互相依赖，允许一个 **Extension** 类去修正传递到另外一个 bundle 的 **Extension** 类的配置将会非常有用，就好像最终开发者的配置文件实际放在了 **app/config/config.yml** 文件中。这可以通过预先的扩展来完成。获取更多细节，你可以参考[如何简化多个 Bundle 的配置](http://symfony.com/doc/current/cookbook/bundles/prepend_extension.html)。  

## 转储配置

**config:dump-reference** 命令将控制台中的 bundle 的默认配置使用 Yaml 格式转储。  

只要你的 bundle 的配置位于标准的位置（**YourBundle\DependencyInjection\Configuration**）并且对于传到构造器没有争议那么这将会自动进行。如果你有一些不同，你的 **Extension** 类必须重写 [Extension::getConfiguration()](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/DependencyInjection/Extension.html#getConfiguration()) 方法并且返回 **Configuration** 的一个实例。  

## 支持 XML

Symfony 允许人们提供三种不同格式的配置：Yaml, XML 和 PHP。Yaml 和 PHP 应用的相同的语法并且当使用 Config 组件时都是默认支持的。支持 XML 需要你多做一些事情。但是当和别人共享你的 bundle 时，建议你遵循以下的步骤。  

### 使你的 Config Tree 做好支持 XML 的准备

Config 组件提供了一些默认的方法来修正编辑 XML 配置。详见“[正常化](http://symfony.com/doc/current/components/config/definition.html#component-config-normalization)”组件的文档。然而，你也可以做一些选择，这将会提升使用 XML 配置的体验。

### 选择一个 XML 命名空间

在 XML 中，[XML 命名空间](http://en.wikipedia.org/wiki/XML_namespace)是用来决定哪个元素属于特定的 bundle 的配置。命名空间是由 [Extension::getNamespace()](http://api.symfony.com/2.7/Symfony/Component/DependencyInjection/Extension/Extension.html#getNamespace()) 方法返回的。按照惯例，命名空间是一个链接(它并不必须是一个有效的链接且不一定需要存在)。默认情况下，bundle 的命名空间是 **http://example.org/dic/schema/DI_ALIAS**，这里的 **DI_ALIAS** 是扩展的 DI 别名。你可能想要将其改成更加专业的链接：  

```
// src/Acme/HelloBundle/DependencyInjection/AcmeHelloExtension.php

// ...
class AcmeHelloExtension extends Extension
{
    // ...

    public function getNamespace()
    {
        return 'http://acme_company.com/schema/dic/hello';
    }
}
```  

### 提供一个 XML 架构

XML 具有一个非常有用的特征叫做 [XML 架构](http://en.wikipedia.org/wiki/XML_schema)。这个允许在 XML Schema Definition （一个 xsd 文件）中描述所有的可能的元素和属性以及它们的值。这个 XSD 文件被 IDEs 使用用作智能完成，同时也被 Config 组件用来验证元素。  

为了使用这个架构，XML 配置文件必须提供一个 **xsi:schemaLocation** 附件指向 XSD 文件作为一个特定的 XML 命名空间。这个 XML 命名空间之后将会被从 [Extension::getXsdValidationBasePath()](http://api.symfony.com/2.7/Symfony/Component/DependencyInjection/ExtensionInterface.html#getXsdValidationBasePath()) 方法返回的 XSD 验证基本路径所取代。这个命名空间将会跟随其它的从基本路径到文件的路径。  

按照惯例，XSD 文件存在于 **Resources/config/schema**，但是你可以把它放在任何你想放的地方。你需要将这个路径作为基本路径返回：  

```
// src/Acme/HelloBundle/DependencyInjection/AcmeHelloExtension.php

// ...
class AcmeHelloExtension extends Extension
{
    // ...

    public function getXsdValidationBasePath()
    {
        return __DIR__.'/../Resources/config/schema';
    }
}
```  

假设 XSD 文件叫做 **hello-1.0.xsd**，框架的位置就会是 **http://acme_company.com/schema/dic/hello/hello-1.0.xsd**：  

```
<!-- app/config/config.xml -->
<?xml version="1.0" ?>

<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:acme-hello="http://acme_company.com/schema/dic/hello"
    xsi:schemaLocation="http://acme_company.com/schema/dic/hello
        http://acme_company.com/schema/dic/hello/hello-1.0.xsd">

    <acme-hello:config>
        <!-- ... -->
    </acme-hello:config>

    <!-- ... -->
</container>
```  