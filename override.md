# 如何重写部分 Bundle

这篇文档是教你如何重写第三方 bundle 的不同部分的快速指南。  

## 模板

获取有关于重写模板的信息参见  

- [重写 Bundle 模板](http://symfony.com/doc/current/book/templating.html#overriding-bundle-templates)  
- [如何使用 Bundle 的继承来重写部分 Bundle](http://symfony.com/doc/current/cookbook/bundles/inheritance.html)  

## 路由

在 Symfony 中路由是不会自动输入的，如果你想要从任何的 bundle 中包含路由，那么它们必须从你的应用程序的某个地方人工输入（例如 **app/config/routing.yml**）。 

最简单的“重写”路由的方法就是根本就不要输入。代替输入一个第三方 bundle 的路由，只需简单的将路由文件复制到你的应用程序中修正，然后输入。  

## 控制器

假设第三方的 bundle 使用无服务的控制器（这是大多数的情况），你可以使用 bundle 的继承来很容易地重写控制器。获取更多信息，你可以参考[如何使用 Bundle 的继承来重写部分 Bundle](http://symfony.com/doc/current/cookbook/bundles/inheritance.html)。如果控制器是一个服务，下一节我们将教你如何进行重写。  

## 服务和配置

为了重写、扩展服务，这里有两个选项。第一个，你可以通过在 **app/config/config.yml** 中设置参数的方式将服务的类的名称保留到你自己的类的名称中。如果类的名称是作为包含服务的 bundle 服务设置的一个参数被定义的，这就是唯一可能的方法。举例来说，为了重写 Symfony 的 **translator** 服务所使用的类，你需要重写 **translator.class** 变量。找到哪个参数需要重写可能需要花费一些精力。对于翻译器，参数被定义与使用于核心的 FrameworkBundle 的 **Resources/config/translation.xml** 文件中：　　

YAML:

```YAML
# app/config/config.yml
parameters:
    translator.class: Acme\HelloBundle\Translation\Translator
```  

XML:

```XML
<!-- app/config/config.xml -->
<parameters>
    <parameter key="translator.class">Acme\HelloBundle\Translation\Translator</parameter>
</parameters>
```  

PHP:

```PHP
// app/config/config.php
$container->setParameter('translator.class', 'Acme\HelloBundle\Translation\Translator');
```  

第二个，如果类不是作为一个参数，当你的 bundle 使用时或者你需要修正除了类的名称之外的东西时你需要确保类一直被重写，你可以应用一个编译器通过：  

```
// src/Acme/DemoBundle/DependencyInjection/Compiler/OverrideServiceCompilerPass.php
namespace Acme\DemoBundle\DependencyInjection\Compiler;

use Symfony\Component\DependencyInjection\Compiler\CompilerPassInterface;
use Symfony\Component\DependencyInjection\ContainerBuilder;

class OverrideServiceCompilerPass implements CompilerPassInterface
{
    public function process(ContainerBuilder $container)
    {
        $definition = $container->getDefinition('original-service-id');
        $definition->setClass('Acme\DemoBundle\YourService');
    }
}
```  

本例中你从原始的服务中取得服务定义，并且将它的名称设置成你的类的名称。  

阅读[如何在 Bundle 中使用 Compiler Passes](http://symfony.com/doc/current/cookbook/service_container/compiler_passes.html) 来获取关于如何使用 compiler passes 的信息。如果你想要进行一些重写类之外的事情，比如说添加方法调用，那么你就能应用 compiler passe 方法了。  

## 实体和实体映射

由于 Doctrine 工作的原理，重写 bundle 的实体映射是不可能的。然而，如果一个 bundle 提供了映射的超级类（就好像 FOSUserBundle 中的 **User** 实体）我们就能重写属性和关联。你可以在 [Doctrine 的文档](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/inheritance-mapping.html#overrides)中学习有关这个的特征以及限制。  

## 表单

为了重写表单类型，它必须作为服务注册（就是意味着它要有 **form.type** 标签）。然后你就可以重写像[服务和配置](http://symfony.com/doc/current/cookbook/bundles/override.html#services-configuration)中介绍的那样重写任何服务了。当然，这个只有在用别名的时候才会起作用而不是被实例化，例如：  

```
$builder->add('name', 'custom_type');
```  

而不是：

```
$builder->add('name', new CustomType());
```  

## 校验元数据

Symfony 从每一个 bundle 中加载所有的校验配置文件然后将它们合并成一个校验树。这就意味着你可以向属性中添加新的限制，但是你不能重写它们。  

为了重写这个，第三方 bundle 需要有[校验组](http://symfony.com/doc/current/book/validation.html#book-validation-validation-groups)的配置。例如，FOSUserBundle 有这个设置。创建你自己的校验，向新的校验组中添加限制：  

YAML:

```YAML
# src/Acme/UserBundle/Resources/config/validation.yml
FOS\UserBundle\Model\User:
    properties:
        plainPassword:
            - NotBlank:
                groups: [AcmeValidation]
            - Length:
                min: 6
                minMessage: fos_user.password.short
                groups: [AcmeValidation]
```  

XML:

```XML
<!-- src/Acme/UserBundle/Resources/config/validation.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<constraint-mapping xmlns="http://symfony.com/schema/dic/constraint-mapping"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/constraint-mapping
        http://symfony.com/schema/dic/constraint-mapping/constraint-mapping-1.0.xsd">

    <class name="FOS\UserBundle\Model\User">
        <property name="plainPassword">
            <constraint name="NotBlank">
                <option name="groups">
                    <value>AcmeValidation</value>
                </option>
            </constraint>

            <constraint name="Length">
                <option name="min">6</option>
                <option name="minMessage">fos_user.password.short</option>
                <option name="groups">
                    <value>AcmeValidation</value>
                </option>
            </constraint>
        </property>
    </class>
</constraint-mapping>
```  

现在，更新 FOSUserBundle 的配置，因此它就会应用你的校验组来替代原始的那个。  

## 翻译

翻译和 bundle 并不相关，但是和域相关。这就意味着你可以从任何的翻译文件中重写翻译，只要它在[正确的域](http://symfony.com/doc/current/components/translation/introduction.html#using-message-domains)中。  

> 最终的翻译文件总是会成功。这就意味着你需要确保包含*你的*翻译的 bundle 在你重写了翻译的那些 bundle 之后被加载。这个在 **AppKernel** 中完成。  
> 翻译文件也不会知道 [bundle 继承](http://symfony.com/doc/current/cookbook/bundles/inheritance.html)。如果你想从父 bundle 重写翻译，要确保父 bundle 在 **AppKernel** 类中在子 bundle 之前加载。  
> 经常成功的文件位于 **app/Resources/translations**，因为那些文件总是最后加载。  