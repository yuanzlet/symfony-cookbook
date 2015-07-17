# 如何使用序列化

把一个对象序列化和反序列化成不同的格式（例如：JSON 或者 XML）是一个非常复杂的话题。在 Symfony 中有一个序列化程序组件 [Serializer Component ](http://symfony.com/doc/current/components/serializer.html "Serializer Component")  可以给您提供一些工具来帮您解决上述问题。

事实上，当您开始序列化和反序列化之前，您可以通过阅读[序列化组件](http://symfony.com/doc/current/components/serializer.html "序列化组件")来了解并熟悉序列化，正规化子组件和编码器。

## 激活序列化程序

> **2\.3** 在 Symfony 之中一直都有序列化程序，不过在 Symfony 2.3 版本之前，您需要自己去构建序列化程序服务。

YAML:

```
# app/config/config.yml
framework:
    # ...
    serializer:
        enabled: true
```

XML:

```
<!-- app/config/config.xml -->
<framework:config>
    <!-- ... -->
    <framework:serializer enabled="true" />
</framework:config>
```

PHP:

```
// app/config/config.php
$container->loadFromExtension('framework', array(
    // ...
    'serializer' => array(
        'enabled' => true,
    ),
));
```

## 使用序列化程序服务 

您一旦启动了序列化服务 **serializer**，您就可以在您需要的任何服务进程中使用它，它还可以被用在下述的控制器中：

```
// src/AppBundle/Controller/DefaultController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class DefaultController extends Controller
{
    public function indexAction()
    {
        $serializer = $this->get('serializer');

        // ...
    }
} 
```

## 添加正规化子组件和编码器

> **2\.7** [ObjectNormalizer](http://api.symfony.com/2.7/Symfony/Component/Serializer/Normalizer/ObjectNormalizer.html "ObjectNormalizer") 是在 Symfony 2.7 中默认启用的。在以前的版本，您需要加载你自己的标准组件。

您一旦启动上述组件，在容器中便可以使用序列化程序服务 **serializer**，并且还配备了两个[编码器](http://symfony.com/doc/current/components/serializer.html#component-serializer-encoders "编码器") ( [JsonEncoder](http://api.symfony.com/2.7/Symfony/Component/Serializer/Encoder/JsonEncoder.html "JsonEncoder")  和 [XmlEncoder](http://api.symfony.com/2.7/Symfony/Component/Serializer/Encoder/XmlEncoder.html "XmlEncoder")) 以及  [ObjectNormalizer 标准组件](http://symfony.com/doc/current/components/serializer.html#component-serializer-normalizers "ObjectNormalizer 标准组件"))。

您可以通过给正规化子组件和编码器添加上 [serializer.normalizer](http://symfony.com/doc/current/reference/dic_tags.html#reference-dic-tags-serializer-normalizer "serializer.normalizer") 和 [serializer.encoder](http://symfony.com/doc/current/reference/dic_tags.html#reference-dic-tags-serializer-encoder "serializer.encoder") 标记来加载他们。也可以通过设置标记的优先级顺序来决定匹配顺序。

这里有一个描述如何去加载 [GetSetMethodNormalizer](http://api.symfony.com/2.7/Symfony/Component/Serializer/Normalizer/GetSetMethodNormalizer.html"GetSetMethodNormalizer") 的示例：

YAML:

```
# app/config/services.yml
services:
    get_set_method_normalizer:
        class: Symfony\Component\Serializer\Normalizer\GetSetMethodNormalizer
        tags:
            - { name: serializer.normalizer }
```

XML:

```
<!-- app/config/services.xml -->
<services>
    <service id="get_set_method_normalizer" class="Symfony\Component\Serializer\Normalizer\GetSetMethodNormalizer">
        <tag name="serializer.normalizer" />
    </service>
</services>
```

PHP:

```
// app/config/services.php
use Symfony\Component\DependencyInjection\Definition;

$definition = new Definition(
    'Symfony\Component\Serializer\Normalizer\GetSetMethodNormalizer'
));
$definition->addTag('serializer.normalizer');
$container->setDefinition('get_set_method_normalizer', $definition);
```

## 使用序列化组注释

> **2\.7** 在 Symfony 2.7 节中，我们讲解了 Symfony 支持序列化组的应用。

使用以下配置去启动[序列化组注释](http://symfony.com/doc/current/components/serializer.html#component-serializer-attributes-groups"序列化组注释")：

YAML:

```
# app/config/config.yml
framework:
    # ...
    serializer:
        enable_annotations: true
```

XML:

```
<!-- app/config/config.xml -->
<framework:config>
    <!-- ... -->
    <framework:serializer enable-annotations="true" />
</framework:config>
```

PHP:

```
// app/config/config.php
$container->loadFromExtension('framework', array(
    // ...
    'serializer' => array(
        'enable_annotations' => true,
    ),
));
```

接下来,把  [@Groups annotations](http://symfony.com/doc/current/components/serializer.html#component-serializer-attributes-groups"@Groups annotations")  注释添加到您的类中，并且选择在序列化的时候将要使用哪个组：

```
$serializer = $this->get('serializer');
$json = $serializer->serialize(
    $someObject,
    'json', array('groups' => array('group1'))
);
```

## 启用元数据缓存

> **2\.7** 在 Symfony 2.7 节中，我们介绍了序列化程序。

就像序列化组可以提高应用程序的性能一样，我们可以使用序列化组件来使用元数据，任何在 **Doctrine\Common\Cache\Cache** 下被服务所实现的接口都能被使用。

在服务中使用的 [APCu](https://github.com/krakjoe/apcu "APCu")（PHP 中的 APC 单元 < 5.5 版本） 单元都是内置的。

YAML:

```
# app/config/config_prod.yml
framework:
    # ...
    serializer:
        cache: serializer.mapping.cache.apc
```

XML:

```
<!-- app/config/config_prod.xml -->
<framework:config>
    <!-- ... -->
    <framework:serializer cache="serializer.mapping.cache.apc" />
</framework:config>
```

PHP:

```
// app/config/config_prod.php
$container->loadFromExtension('framework', array(
    // ...
    'serializer' => array(
        'cache' => 'serializer.mapping.cache.apc',
    ),
));
```

## 进一步使用序列化组件

[DunglasApiBundle](https://github.com/dunglas/DunglasApiBundle "DunglasApiBundle") 体系提供了一个支持 [JSON-LD](http://json-ld.org/ "JSON-LD") 和 [Hydra Core Vocabulary](http://www.hydra-cg.com/ "Hydra Core Vocabulary")  等超媒体格式 的 API 系统。它是建立在 Symfony 框架和其序列化程序组件之上的。它提供了自定义子正规化、自定义编码器、 自定义元数据和缓存系统。

如果您想完全的利用 Symfony 的序列化程序组件，那么请看看它是怎样捆绑工作的。

