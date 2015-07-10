# 在独立注入类中使用参数

你已经在 [Symfony 服务容器](http://symfony.com/doc/current/book/service_container.html#book-service-container-parameters)中看到如何使用配置参数了。举例来说，存在特殊的情形比如当你想使用 **%kernel.debug%** 参数使得你的 bundle 中的服务进入调试模式。对于这种情况来说为了使系统理解参数的值你需要做很多工作。默认情况下你的 **%kernel.debug%** 参数将会被认为是简单的字符串。看看下面这个例子：  

```
// inside Configuration class
$rootNode
    ->children()
        ->booleanNode('logging')->defaultValue('%kernel.debug%')->end()
        // ...
    ->end()
;

// inside the Extension class
$config = $this->processConfiguration($configuration, $configs);
var_dump($config['logging']);
```  

现在检查结果进一步看看：  

```YAML
my_bundle:
    logging: true
    # true, as expected

my_bundle:
    logging: "%kernel.debug%"
    # true/false (depends on 2nd parameter of AppKernel),
    # as expected, because %kernel.debug% inside configuration
    # gets evaluated before being passed to the extension

my_bundle: ~
# passes the string "%kernel.debug%".
# Which is always considered as true.
# The Configurator does not know anything about
# "%kernel.debug%" being a parameter.
```  

```XML
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:my-bundle="http://example.org/schema/dic/my_bundle">

    <my-bundle:config logging="true" />
    <!-- true, as expected -->

     <my-bundle:config logging="%kernel.debug%" />
     <!-- true/false (depends on 2nd parameter of AppKernel),
          as expected, because %kernel.debug% inside configuration
          gets evaluated before being passed to the extension -->

    <my-bundle:config />
    <!-- passes the string "%kernel.debug%".
         Which is always considered as true.
         The Configurator does not know anything about
         "%kernel.debug%" being a parameter. -->
</container>
```  

```PHP
$container->loadFromExtension('my_bundle', array(
        'logging' => true,
        // true, as expected
    )
);

$container->loadFromExtension('my_bundle', array(
        'logging' => "%kernel.debug%",
        // true/false (depends on 2nd parameter of AppKernel),
        // as expected, because %kernel.debug% inside configuration
        // gets evaluated before being passed to the extension
    )
);

$container->loadFromExtension('my_bundle');
// passes the string "%kernel.debug%".
// Which is always considered as true.
// The Configurator does not know anything about
// "%kernel.debug%" being a parameter.
```  

为了支持这个用例，**Configuration** 类必须通过以下的扩展注入参数：  

```
namespace AppBundle\DependencyInjection;

use Symfony\Component\Config\Definition\Builder\TreeBuilder;
use Symfony\Component\Config\Definition\ConfigurationInterface;

class Configuration implements ConfigurationInterface
{
    private $debug;

    public function  __construct($debug)
    {
        $this->debug = (bool) $debug;
    }

    public function getConfigTreeBuilder()
    {
        $treeBuilder = new TreeBuilder();
        $rootNode = $treeBuilder->root('my_bundle');

        $rootNode
            ->children()
                // ...
                ->booleanNode('logging')->defaultValue($this->debug)->end()
                // ...
            ->end()
        ;

        return $treeBuilder;
    }
}
```  

并且通过 **Extension** 类在 **Configuration** 的构造器中设置：  

```
namespace AppBundle\DependencyInjection;

use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\HttpKernel\DependencyInjection\Extension;

class AppExtension extends Extension
{
    // ...

    public function getConfiguration(array $config, ContainerBuilder $container)
    {
        return new Configuration($container->getParameter('kernel.debug'));
    }
}
```  

>在 Extension 中设置默认

>在 TwigBundle 和 AsseticBundle 中的 **Configurator** 类中有一些 **%kernel.debug%** 的应用实例。然而这是因为默认的参数值是由 Extension 类设置的。例如在 AsseticBundle 中，你可以找到：  

>```
>$container->setParameter('assetic.debug', $config['debug']);
>```

>**%kernel.debug%** 字符串作为争论处理的解释者通过向进行评估的容器进行解释。这两种方法达到同一个目标。AsseticBundle 不会使用 **%kernel.debug%** 相反的会使用新的 **%assetic.debug%** 参数。  


