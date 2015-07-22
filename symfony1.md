# Symfony2 与 Symfony1 的区别

Symfony2 框架和第一代框架相比，包含了一个重要的改变。幸运的是，由于核心的 MVC 架构，曾经精通一个 Symfony1 项目的技能与使用 Symfony2 的开发中用到的技能相关性很大。当然，**app.ymi** 没了，但是路由（routing）、控件（controllers）和模板（templates）还在。

本章大略的讲述 Symfony2 与 Symfony1 的区别。正如您所看到的，许多任务以稍稍不同的方式处理。您将会欣赏这些细微的差别，因为它们提升稳定性、可预测性和解耦（decoupled）您在 Symfony2 中应用程序的代码。

所以，靠坐在椅子上，放松，就像您从“过去”旅行到“现在”。

## 目录结构

当您看着一个 Symfony2 项目——例如 [Symfony Standard Edition](https://github.com/symfony/symfony-standard)——您将会注意到一个与 Symfony1 非常不同的目录结构。然而，这些不同某种程度上讲仅仅是表面上的。

### app/目录

像 Symfony1 里面一样，您的工程有一个或多个应用程序，每一个都存在于 **apps/**目录之下（例如：**apps/frontend**）。在 Symfony2 默认情况下，您只有一个由 **app/**目录代表的应用程序。在 Symfony1 中，**app/**目录包含应用程序的特定配置。它还包含应用程序特定的缓存、日志、模板目录和一个 **Kernel** 类（**AppKernel**），是代表应用程序的基本对象。

和 Symfony1 不同的是，**app/**目录下几乎没有 PHP 代码。这个目录不是要像 Symfony1 一样存储模块（modules）或者库文件。相反，它只是储存配置和其他资源（模板，翻译文件（translation files））。

### src/目录

简单地说，您的所有实际代码都在这里。在Symfony2 中，所有的应用程序代码都在一个包（bundle，大致相当于一个 Symfony1 插件）里，并且默认情况下每个包（bundle）都在 src 目录之下。那么，**src** 目录有点像 Symfony1 中的 **plugins** 目录，但是更加灵活。此外，您的包（bundle）会存在于 **src/**目录之下，而第三方的包（bundle）会存在于 **vendor/**目录之下的某个地方。

为了得到一个更好的关于 **src/**目录的概念，首先想一想一个 Symfony1 应用程序的结构。首先，一部分代码可能存在于一个或多个程序中。最常见的那些包括模块，但也可能包括您写在程序中的其他 PHP 类。您可能也在您的项目的 **config** 目录下创建了一个 **schema.yml** 文件并建立了几个模型（model）文件。最后，为了帮助一些常见功能，您使用了一些存在 **plugins/**目录下的第三方插件。换句话说，驱动您的应用程序的代码储存在不同的地方。

在 Symfony2 中，生活简单得多，因为所有的 Symfony2 代码都必须存在于一个包（bundle）里。pretend symfony1 项目中，所有的都可以移入一个或多个插件中（事实上，这是一个非常好的尝试）。假设所有的模块、PHP 类、架构、路由配置等都转移到一个插件里，Symfony1 **plugins/**目录会和 Symfony2 **src/**目录非常相似。

再一次简而言之，**src/**目录就是您的代码、资产（assets）、模板和大多数特定于您的项目的东西存在的地方。

### vendor/目录

**vendor/**目录基本上相当于 Symfony1 里的 **lib/vendor/**目录，这是所有的 vendor 库和包（bundle）的传统目录。默认情况下，您会在这个目录里发现 Symfony2 的库文件以及一些其他的相关的库，例如 Doctrine2，Twig 和 Swift Mailer。第三方 Symfony2 包（bundle）存在于**vendor/**目录下的某个地方。

### web/目录

**web/**目录没有太多改变。最明显的不同是没有 **css/**，**js/** 和 **images/**目录。这是有意的。像您的 PHP 代码，所有的资产（assets）也都在一个包（bundle）里。在一个控制台命令的帮助下，每个包（bundle）的 **Resources/public/**目录被复制或者象征性的连接到 **web/bundles/**目录。这允许您保持包（bundle）内资源的组织，但是仍使它们能够被人们使用。要确保所有的包（bundle）都是可用的，运行以下命令：

```
$ php app/console assets:install web
```

> 这个命令是 Symfony2 的，等同于 Symfony1 的 **plugin:publish-assets** 命令。

## 自动加载

现代框架的优点之一是永远不需要担心需求文件。通过使用一个自动加载器，您可以指定项目中的任何类，并且相信它可用。在 Symfony2 中，自动加载变得更普遍，更快，并且独立的需求清除缓存。

在 Symfony1 中，自动加载是在整个项目中搜索需要的 PHP 类文件并将这些信息储存在一个巨大的数组（array）中。那数组（array）准确的告诉 Symfony1 哪个文件包含哪些类。在生产环境中，这就导致当类增加或者移走时需要清除缓存。

在 Symfony2 中，一个名为 [Composer](https://getcomposer.org/) 的工具处理这一过程。自动加载器幕后的想法很简单：您的类的名称（包括命名空间）必须与包含这个类的文件的路径相匹配。以 Symfony2 Standard Edition 中的 FrameworkExtraBundle 作为例子：

```PHP
namespace Sensio\Bundle\FrameworkExtraBundle;

use Symfony\Component\HttpKernel\Bundle\Bundle;
// ...

class SensioFrameworkExtraBundle extends Bundle
{
    // ...
}
```

这个文件本身存在于 **vendor/sensio/framework-extra-bundle/Sensio/Bundle/FrameworkExtraBundle/SensioFrameworkExtraBundle.php**。正如您所见，路径的第二部分采用类的命名空间。第一部分等同于 SensioFrameworkExtraBundle 的包（package）的名称。

命名空间 **Sensio\Bundle\FrameworkExtraBundle** 和包（package）名称 **sensio/framework-extra-bundle** 清楚地说明了文件应该存在的目录是（**vendor/sensio/framework-extra-bundle/Sensio/Bundle/FrameworkExtraBundle/**）。然后 Composer 就可以在这个特定空间中寻找文件并快速地加载它。

如果文件不在这个准确的位置，您会收到这样一个错误提示 **Class "Sensio\Bundle\FrameworkExtraBundle\SensioFrameworkExtraBundle" does not exist.**。在 Symfony2 中，一个“class does not exist”错误意味着类的命名空间和物理位置没有匹配。基本上，Symfony2 是在确切位置找一个类，但这个位置不存在（或包含不同的类）。为了让类能够自动加载，您**永远不需要清除 Symfony2 的缓存**。

就像之前提到的那样，为了让自动加载器能够工作，它需要知道命名空间 **Sensio** 存在于 **vendor/sensio/framework-extra-bundle** 目录下，并且，举例来讲，命名空间 **Doctrine** 存在于 **vendor/doctrine/orm/lib/**目录下。这个映射完全由 Composer 控制。每一个通过 Composer 加载的第三方库都有它明确的设置，并且 Composer 为您处理一切事情。

为了让这能够工作，您的项目适用的所有的第三方库都必须定义在 **composer.json** 文件中。

如果您看一看 Symfony Standard Edition 中的 **HelloController**，您会发现它存在在 **Acme\DemoBundle\Controller** 命名空间下。然而 AcmeDemoBundle 并没有定义在您的 **composer.json** 文件中。尽管如此，但是这些文件还是自动加载了。这是因为您可以告诉 Composer 在没有定义一个依赖的情况下从特定的目录加载文件：

```
"autoload": {
    "psr-0": { "": "src/" }
}
```

这意味着如果一个类没有在 **vendor** 目录中找到，Composer 会在抛出“class does not exist”错误之前，在 **src** 目录中搜索它。在 [the Composer documentation](https://getcomposer.org/doc/04-schema.md#autoload) 中阅读更多的关于 Composer 配置的知识。

## 使用控制台

在 Symfony1 中，控制台是在项目的根目录下被称为 **symfony** 的。

```
$ php symfony
```

在 Symfony2 中，控制台是在 app 子目录下被称为 **console** 的。

```
$ php app/console
```

## 应用程式

在 Symfony1 项目中，通常有几个应用程序：例如一个用于前端和一个用于后端。

在 Symfony2 项目中，您只需要创建一个应用程序（一个 blog 应用程序，一个 intranet 应用程序，...）。多数时候，如果您想要创建第二个应用程序，您可以创建另一个项目并且在它们之间共享一些包（bundles）。

然后如果您需要将一些包（bundles）的前端后端功能区分开，您可以给控件创建子命名空间，给模板创建子目录，区分语义结构，分离路由设置，等等。

当然，您的项目中有多个应用程序并没有错误，这完全取决于您。第二个应用程序意味着一个新的目录，例如 **my_app/**，具有相同的基本设置，比如 **app/**目录。

> 在词汇表中查看一个[工程（Project）](http://symfony.com/doc/current/glossary.html#term-project)、一个[应用程序（Application）](http://symfony.com/doc/current/glossary.html#term-application)和一个[包（bundle）](http://symfony.com/doc/current/glossary.html#term-bundle)的定义。

## 包（bundles）和插件

在一个 Symfony1 项目中，一个插件可能包含配置，模块，PHP库，资产（assets）和其他与项目相关的东西。在 Symfony2 中，插件的思想被“包（bundle）”代替了。一个包甚至比插件更加强大，因为核心 Symfony2 框架可以通过一系列包带入。在 Symfony2 中，包是一等公民，它非常灵活以至于核心代码本身就是一个包。

在 Symfony1 中，一个插件必须在 **ProjectConfiguration** 类中启用：

```
// config/ProjectConfiguration.class.php
public function setup()
{
    // some plugins here
    $this->enableAllPluginsExcept(array(...));
}
```

在 Symfony2 中，包在应用程序内核（application kernel）中激活：

```
// app/AppKernel.php
public function registerBundles()
{
    $bundles = array(
        new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
        new Symfony\Bundle\TwigBundle\TwigBundle(),
        ...,
        new Acme\DemoBundle\AcmeDemoBundle(),
    );

    return $bundles;
}
```

## 路由（routing.yml）和配置（config.yml）

在 Symfony1 中，**routing.yml** 和 **app.yml** 配置文件会自动加载在任何插件中。在 Symfony2 中，一个包中的路由和应用配置必须手动包含（include）。举个例子，要从一个名为 AcmeDemoBundle 的包里包括一个路由资源，您可以按照下面例子做：

```YAML
# app/config/routing.yml
_hello:
    resource: "@AcmeDemoBundle/Resources/config/routing.yml"
```

```XML
<!-- app/config/routing.yml -->
<?xml version="1.0" encoding="UTF-8" ?>

<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

    <import resource="@AcmeDemoBundle/Resources/config/routing.xml" />
</routes>
```

```PHP
// app/config/routing.php
use Symfony\Component\Routing\RouteCollection;

$collection = new RouteCollection();
$collection->addCollection($loader->import("@AcmeHelloBundle/Resources/config/routing.php"));

return $collection;
```

这将会加载从 AcmeDemoBundle 中的 **Resources/config/routing.yml** 文件中发现的路由。特殊的 **@AcmeDemoBundle** 是一个简写语法，它会在内部解决包的完整的路径。

您可以用同样的策略从一个包中带入设置:

```YAML
# app/config/config.yml
imports:
    - { resource: "@AcmeDemoBundle/Resources/config/config.yml" }

```

```XML
<!-- app/config/config.xml -->
<imports>
    <import resource="@AcmeDemoBundle/Resources/config/config.xml" />
</imports>
```

```PHP
// app/config/config.php
$this->import('@AcmeDemoBundle/Resources/config/config.php')
```

Symfony2 中的配置有点像在 Symfony1 中的 **app.yml**，除了更加系统化。您可以使用 **app.yml** 简单地创建您想要的任何键（key）。默认情况下，这些条目（entries）是没有意义的，完全取决于您在应用程序中如何使用它们：

```
# some app.yml file from symfony1
all:
  email:
    from_address:  foo.bar@example.com
```

在 Symfony2 中，您也可以在 **parameters** 键（parameters key）下创建任意条目（entries）：

```YAML
parameters:
    email.from_address: foo.bar@example.com
```

```XML
<parameters>
    <parameter key="email.from_address">foo.bar@example.com</parameter>
</parameters>
```

```PHP
$container->setParameter('email.from_address', 'foo.bar@example.com');
```

现在您可以用一个控件来获得它，例如：

```PHP
public function helloAction($name)
{
    $fromAddress = $this->container->getParameter('email.from_address');
}
```

现实中，Symfony2 的配置更为强大，主要用于配置您可以使用的对象。更多信息参见“[Service Container](http://symfony.com/doc/current/book/service_container.html)”章节。
