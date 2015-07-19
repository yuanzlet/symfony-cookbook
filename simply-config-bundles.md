# 如何简化多个 Bundle 的配置

当创建可重复利用以及可扩展的应用程序时，开发者经常面临一个选择：创建一个简单的大的 bundle 还是创建多个小的 bundle。创建一个简单的 bundle 的缺点就是不能让用户选择移除他们不需要的功能。创建多个 bundle 的缺点就是配置会变得很繁杂无聊，很多设置都需要对不同的 bundle 重复设置。  

使用下列方法，通过一个单一的扩展来预先设置所有的 bundle，可以移除多个 bundle 的缺点。可以使用 **app/config/config.yml** 中定义的设置来预先设置就好像它们被用户在应用程序配置中明确写出来。  

举例来说，这个可以用来设置在多个 bundle 中应用的实体管理器的名称。或者它也可以用来启用依赖于另一个 bundle 加载的可选特征。  

为了有扩展能力来完成这个工作，你需要实现 [PrependExtensionInterface](http://api.symfony.com/2.7/Symfony/Component/DependencyInjection/Extension/PrependExtensionInterface.html)：  

```
// src/Acme/HelloBundle/DependencyInjection/AcmeHelloExtension.php
namespace Acme\HelloBundle\DependencyInjection;

use Symfony\Component\HttpKernel\DependencyInjection\Extension;
use Symfony\Component\DependencyInjection\Extension\PrependExtensionInterface;
use Symfony\Component\DependencyInjection\ContainerBuilder;

class AcmeHelloExtension extends Extension implements PrependExtensionInterface
{
    // ...

    public function prepend(ContainerBuilder $container)
    {
        // ...
    }
}
```  

在 [prepend()](http://api.symfony.com/2.7/Symfony/Component/DependencyInjection/Extension/PrependExtensionInterface.html#prepend()) 方法中，开发者可以在每个注册的 bundle 的扩展调用 [load()](http://api.symfony.com/2.7/Symfony/Component/DependencyInjection/Extension/ExtensionInterface.html#load()) 方法之前完全访问 [ContainerBuilder](http://api.symfony.com/2.7/Symfony/Component/DependencyInjection/ContainerBuilder.html) 实例。为了预先设置 bundle 的扩展开发者可以在 [ContainerBuilder](http://api.symfony.com/2.7/Symfony/Component/DependencyInjection/ContainerBuilder.html) 实例上使用 [prependExtensionConfig()](http://api.symfony.com/2.7/Symfony/Component/DependencyInjection/ContainerBuilder.html#prependExtensionConfig()) 方法。由于这个方法仅仅预先设置，其它的在 **app/config/config.yml** 中的设置会重写这些以前的设置。  

下面的例子说明了如何在多个 bundle 中预先配置，同时如何在一个特定的其它 bundle 没有被注册时禁用多个 bundle 的标志：  

```
public function prepend(ContainerBuilder $container)
{
    // get all bundles
    $bundles = $container->getParameter('kernel.bundles');
    // determine if AcmeGoodbyeBundle is registered
    if (!isset($bundles['AcmeGoodbyeBundle'])) {
        // disable AcmeGoodbyeBundle in bundles
        $config = array('use_acme_goodbye' => false);
        foreach ($container->getExtensions() as $name => $extension) {
            switch ($name) {
                case 'acme_something':
                case 'acme_other':
                    // set use_acme_goodbye to false in the config of
                    // acme_something and acme_other note that if the user manually
                    // configured use_acme_goodbye to true in the app/config/config.yml
                    // then the setting would in the end be true and not false
                    $container->prependExtensionConfig($name, $config);
                    break;
            }
        }
    }

    // process the configuration of AcmeHelloExtension
    $configs = $container->getExtensionConfig($this->getAlias());
    // use the Configuration class to generate a config array with
    // the settings "acme_hello"
    $config = $this->processConfiguration(new Configuration(), $configs);

    // check if entity_manager_name is set in the "acme_hello" configuration
    if (isset($config['entity_manager_name'])) {
        // prepend the acme_something settings with the entity_manager_name
        $config = array('entity_manager_name' => $config['entity_manager_name']);
        $container->prependExtensionConfig('acme_something', $config);
    }
}
```  

上面所说的将会和在 AcmeGoodbyeBundle 没有被注册以及 **acme_hello** 的 **entity_manager_name** 设置成了 **non_default** 的情况下将以下代码写入 **app/config/config.yml** 等同：  

YAML:

```YAML
# app/config/config.yml
acme_something:
    # ...
    use_acme_goodbye: false
    entity_manager_name: non_default

acme_other:
    # ...
    use_acme_goodbye: false
```  

XML:

```XML
<!-- app/config/config.xml -->
<acme-something:config use-acme-goodbye="false">
    <acme-something:entity-manager-name>non_default</acme-something:entity-manager-name>
</acme-something:config>

<acme-other:config use-acme-goodbye="false" />
```  

PHP:

```PHP
// app/config/config.php
$container->loadFromExtension('acme_something', array(
    // ...
    'use_acme_goodbye' => false,
    'entity_manager_name' => 'non_default',
));
$container->loadFromExtension('acme_other', array(
    // ...
    'use_acme_goodbye' => false,
));
```  