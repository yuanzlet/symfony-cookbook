# 可复用 Bundles 的最佳实践

这里有两种类型的 bundle：  

- 特定应用程序的 bundle：只应用于建立你的应用程序；  
- 可复用的 bundle：意味着可以在多个工程中共享。  

这篇文章就是要告诉你如何构建你的**可复用 bundle** 从而它们可以很方便的进行设置和扩展。这些建议的大多数都不会应用在应用程序的 bundle 因为你总是希望让他们保持尽可能的简单。对于应用程序的 bundle，只要跟着书中和本手册中的实践案例就好。  

>特定应用程序的 bundle 的最佳实践在 [Symfony 框架最佳实践](http://symfony.com/doc/current/best_practices/introduction.html)中讨论。  

## Bundle 名称 ##

bundle 也是 PHP 命名空间。命名空间必须遵循 PHP 命名空间和类名称的 [PSR-0](http://www.php-fig.org/psr/psr-0/) 或 [PSR-4](http://www.php-fig.org/psr/psr-4/) 互通标准：它是由芯片供应商的字段开始，跟着是零或者更多的分类字段，并且以命名空间的缩写结束，缩写必须以 **Bundle** 的后缀结尾。  

命名空间在你向其添加了一个 bundle 类之后就会变成 bundle。bundle 类名称必须遵循以下的这些简单的规则：  

- 只能使用字母数字以及下划线；
- 使用骆驼拼写形式的名称；
- 使用描述性的且短的名称（不能超过两个单词）；
- 名字的前缀要用供应商关联的（并且随意选择种类命名空间）；
- 名字后缀为 **Bundle**。  

这里有一些有效地 bundle 命名空间和类名称：  

|**命名空间**   | **Bundle 类名称**         |  
|---------|:------------:|  
|**Acme\Bundle\BlogBundle**|**AcmeBlogBundle**|  
|**Acme\BlogBundle**|**AcmeBlogBundle**|  

按照惯例，bundle 的 **getName()** 方法应该返回类的名称。  

>如果你公开分享你的 bundle，你必须使用 bundle 的类名称作为库的名称**（AcmeBlogBundle** 和非 **BlogBundle** 的实例）。  


>Symfony 的核心 Bundle 不会以 **Symfony** 作为 Bundle 类的前缀并且经常添加 **Bundle** 附属命名空间；例如： [FrameworkBundle](http://api.symfony.com/2.7/Symfony/Bundle/FrameworkBundle/FrameworkBundle.html)。  

每一个 bundle 都有一个别名，这个别名是小写字母的 bundle 名称的使用下划线的短的版本（**acme_blog** 是 **AcmeBlogBundle** 的别名）。这个别名是为了加强工程中的独特性并且用来定义 bundle 的设置选项（下文将会有一些有用的例子）。  

## 目录结构 ##

AcmeBlogBundle 的基本的目录结构必须如下所示：  

```
<your-bundle>/
├─ AcmeBlogBundle.php
├─ Controller/
├─ README.md
├─ Resources/
│   ├─ meta/
│   │  └─ LICENSE
│   ├─ config/
│   ├─ doc/
│   │  └─ index.rst
│   ├─ translations/
│   ├─ views/
│   └─ public/
└─ Tests/
```  

**下列文件是强制的**，因为他们确保了自动化的工具可以依赖的结构约定性：  

- **AcmeBlogBundle.php**：这是一个将无格式的目录转换成 Symfony bundle 的类（在你的 bundle 的名称中更改这个）；  
- **README.md**：这个文件包含了基本的 bundle 的描述并且它经常展示一些基本的例子和它的完整的文档的链接（它可以使用 GitHub 支持的任何格式标记，例如 README.rst）；  
- **Resources/meta/LICENSE**：代码的完整许可。这个许可文件可以储存在 bundle  的根目录从而为了包的通用的方便；  
- **Resources/doc/index.rst**：Bundle 的文档根文件。  

子目录的深度应当保持在最常用的类和文件的最小数量（最大两层）。  

bundle 的目录是只读的。如果你需要写临时文件，把他们存放在主应用程序的 **cache/** 或者 **log/** 目录下。工具可以在 bundle 目录结构中产生文件，但是只有当产生的文件将要成为库的一部分时。  

下列的类和文件有特定的位置：  

|**类型**  | **目录**      | 
|---------|:------------:|
|Commands|**Command/**|  
|Controllers|**Controller/**|  
|Service Container Extensions|D**ependencyInjection/**|  
|Event Listeners|**EventListener/**|  
|Model classes [1]|**Model/**|  
|Configuration|**Resources/config/**|  
|Web Resources (CSS, JS, images)|**Resources/public/**|  
|Translation files|**Resources/translations/**|  
|Templates|**Resources/views/**|  
|Unit and Functional Tests|**Tests/**|  

[1] 你可以从[如何提供为多个Doctrine的实现提供模型类](http://symfony.com/doc/current/cookbook/doctrine/mapping_model_classes.html)之中找到如何使用 compiler pass 处理映射。  

## 类 ##

bundle 的目录结构使用的是命名空间等级。目前来说，**ContentController** 的控制器储存在 **Acme/BlogBundle/Controller/ContentController.php** 并且完整有效地类名称是 **Acme\BlogBundle\Controller\ContentController**。  

所有的类和文件必须遵循 [Symfony 编码标准](http://symfony.com/doc/current/contributing/code/standards.html)。  

一些类应当被看做是外观并且应当尽可能的短，就像 Commands, Helpers, Listeners, 和 Controllers 一样。  

和事件调度器相连接的类必须以 **Listener** 作为后缀。  

例外的类必须存储在 **Exception** 为名称的子命名空间中。  

## 厂商 ##

bundle 必须不能嵌在第三方的 PHP 函数库中。它需要依赖标准的 Symfony 自动装载作为替代。  

bundle 必须不能嵌在第三方的用 JavaScript, CSS, 或者其他语言编写的函数库中。  

## 测试 ##

bundle 应当携带一个由 PHPUnit 编写的存储在 **Tests/** 目录下的测试组件。测试必须遵循以下的原则：  

- 测试组件必须可以用应用程序中的简单的 **phpunit** 命令来执行；
- 功能测试必须只能够测试反应输出和一些性能分析信息如果你有的话；
- 测试必须覆盖至少 95 % 的代码基础。  

>测试组件必须不能够包含 **AllTests.php** 脚本，但是必须依靠 **phpunit.xml.dist** 文件的存在。  

## 文档 ##

所有的类和功能都必须由完整的 PHPDoc 产生。  

大量的文档应该在 **Resources/doc/** 目录下以 [reStructuredText](http://symfony.com/doc/current/contributing/documentation/format.html) 格式文件的形式提供，**Resources/doc/index.rst** 文件是唯一的强制性文件并且必须是文档的入口。  

### 安装须知 ###

为了简化安装第三方的 bundle。你可以考虑在你的 **README.md** 应用下列标准的须知。  

 
```Markdown 
Installation
============

Step 1: Download the Bundle
---------------------------

Open a command console, enter your project directory and execute the
following command to download the latest stable version of this bundle:

\`\`\`bash
$ composer require <package-name> "~1"
\`\`\`

This command requires you to have Composer installed globally, as explained
in the [installation chapter](https://getcomposer.org/doc/00-intro.md)
of the Composer documentation.

Step 2: Enable the Bundle
-------------------------

Then, enable the bundle by adding it to the list of registered bundles
in the `app/AppKernel.php` file of your project:

\`\`\`php
<?php
// app/AppKernel.php

// ...
class AppKernel extends Kernel
{
    public function registerBundles()
    {
        $bundles = array(
            // ...

            new <vendor>\<bundle-name>\<bundle-long-name>(),
        );

        // ...
    }

    // ...
}
\`\`\`
```  

  
```reStructuredText
Installation
============

Step 1: Download the Bundle
---------------------------

Open a command console, enter your project directory and execute the
following command to download the latest stable version of this bundle:

.. code-block:: bash

    $ composer require <package-name> "~1"

This command requires you to have Composer installed globally, as explained
in the `installation chapter`_ of the Composer documentation.

Step 2: Enable the Bundle
-------------------------

Then, enable the bundle by adding it to the list of registered bundles
in the ``app/AppKernel.php`` file of your project:

.. code-block:: php

    <?php
    // app/AppKernel.php

    // ...
    class AppKernel extends Kernel
    {
        public function registerBundles()
        {
            $bundles = array(
                // ...

                new <vendor>\<bundle-name>\<bundle-long-name>(),
            );

            // ...
        }

        // ...
    }

.. _`installation chapter`: https://getcomposer.org/doc/00-intro.md
```  

上述的例子假设你至少安装了 bundle 的稳定版本，这样你就不用提供包的版本号（例如 **composer require friendsofsymfony/user-bundle**）。如果安装须知提及到一些过去的 bundle 版本或者不稳定的版本，就要包含这个版本的约束条件（例如 **composer require friendsofsymfony/user-bundle "~2.0@dev"**）。  

视情况而定，你可以添加更多的安装步骤（*第三步，第四步*等等）来解释其他需要解释的任务，如注册路由或者转存资产。  

## 路由选择 ##
如果 bundle 提供路由，那么它必须在 bundle 的别名前缀。举例来说，如果你的 bundle 叫做 AcmeBlogBundle，它的所有的前缀必须是 **acme_blog_**。  

## 模板 ##

如果 bundle 提供了模板，他们一定是用的 Twig。bundle 不能提供主要设计，如果它提供了完全运行的应用程序除外。  

## 翻译文件 ##

如果 bundle 提供了信息翻译，他们必须用 XLIFF 格式定义；域必须是以 bundle 的名称命名（**acme_blog**）。  

bundle 必须不能推翻另外一个 bundle 的已经存在的信息。  

## 设置 ##

为了提供更多的灵活性，bundle 能够提供可配置的设置通过使用 Symfony  的built-in 机制。  

对于简单的配置的设置，依赖于 Symfony 的设置的默认的 **parameters** 入口。Symfony 参数是简单的键、值的组合；一个有效地 PHP 的值。每一个参数的名称都应该用 bundle 的别名开始，虽然这只是一个好的实践的建议。剩余参量的名称将会使用句号（.）来分开不同的部分（例如 acme_blog.author.email）。  

最终使用者可以在任意的设置文件中提供值：  


```YAML
# app/config/config.yml
parameters:
    acme_blog.author.email: fabien@example.com
```  

```XML 
<!-- app/config/config.xml -->
<parameters>
    <parameter key="acme_blog.author.email">fabien@example.com</parameter>
</parameters>
```  

```PHP
// app/config/config.php
$container->setParameter('acme_blog.author.email', 'fabien@example.com');
```  

从容器中取回你的代码的设置参数：  

```
$container->getParameter('acme_blog.author.email');
```  

即使这个机制足够简单，你也需要考虑使用更先进的 [semantic bundle configuration](http://symfony.com/doc/current/cookbook/bundles/configuration.html)。  

## 版本化 ##

Bundle 必须根据 [Semantic Versioning Standard](http://semver.org/) 进行版本化。  

## 服务 ##

如果 bundle 定义了服务，他们必须用 bundle 的别名作为前缀。举例来说，AcmeBlogBundle 服务必须用 **acme_blog** 作为前缀。  

除此之外，服务不一定就是意味着应用程序直接用的，应当[被定义为私有的](http://symfony.com/doc/current/components/dependency_injection/advanced.html#container-private-services)。  

>你可以通过阅读这篇名为[如何在 Bundle 内部加载服务配置](http://symfony.com/doc/current/cookbook/bundles/extension.html)的文章来学习更多有关于在 bundle 中加载服务的知识。  

## Composer 元数据 ##

**composer.json** 文件应当至少包括以下的元数据：  

**name**  
由供应商和短的 bundle 名组成。如果你自己发布了一个 bundle 而不是代表一个公司的话，使用你自己的名字（例如 **johnsmith/blog-bundle**）。bundle 的短名字包括了供应商的名字并且用连字符分开每一个单词。举例来说：**AcmeBlogBundle** 被转换成 **blog-bundle**，**AcmeSocialConnectBundle** 被转换成 **social-connect-bundle**。  

**description**  
一个对于 bundle 的目的的简短的解释。  

**type**  
使用 **symfony-bundle** 的值。  

**license**  
**MIT** 是 Symfony 的最受欢迎的证书，但是你也可以应用其他的证书。  

**autoload**  
这个信息是 Symfony 用来加载 bundle 的类的。[PSR-4](http://www.php-fig.org/psr/psr-4/) 自动加载标准推荐用于现代的 bundle，但是 [PSR-0](http://www.php-fig.org/psr/psr-0/) 标准也是推荐的。  

为了让开发者更加容易的找到自己的 bundle，在 [Packagist](https://packagist.org/) 上注册，它是 Composer 包的仓库。  

## 自定义验证约束条件 ##

从 Symfony 2.5 版本开始，一个新的验证 API 被引进。实际上，一共有三个使用者可以在自己的工程中进行设置的模型：  

- 2.4：最原始的 2.4 和早期的验证 API；
- 2.5：新的 2.5 和接下来的验证 API；
- 2.5-BC：具有向后兼容性的新的 2.5 这样 2.4 的 API 依旧能用。这个只是在 PHP 5.3.9+ 上可用。  

>从 Symfony 2.7 开始，对于 2.4 API 的支持被舍弃并且最小的 Symfony PHP 版本也已经增加到 5.3.9。如果你的 bundle 需要 Symfony 的版本在 2.7 及以上，你就不用再关心 2.4 API 了。  

作为一个 bundle 作者，你想要支持*所有的* API,因为有的用户依旧在使用 2.4 API。特别的，如果你的 bundle 直接违反了 [ExecutionContext](http://api.symfony.com/2.7/Symfony/Component/Validator/Context/ExecutionContext.html) （例如就像自定义验证约束条件），你需要检查哪个 API 被应用了。下面的代码，将会对*所有的*用户有用：  

```
use Symfony\Component\Validator\ConstraintValidator;
use Symfony\Component\Validator\Constraint;
use Symfony\Component\Validator\Context\ExecutionContextInterface;
// ...

class ContainsAlphanumericValidator extends ConstraintValidator
{
    public function validate($value, Constraint $constraint)
    {
        if ($this->context instanceof ExecutionContextInterface) {
            // the 2.5 API
            $this->context->buildViolation($constraint->message)
                ->setParameter('%string%', $value)
                ->addViolation()
            ;
        } else {
            // the 2.4 API
            $this->context->addViolation(
                $constraint->message,
                array('%string%' => $value)
            );
        }
    }
}
```  

## 从指导书中学到更多 ##

- [如何在 Bundle 内部加载服务配置](http://symfony.com/doc/current/cookbook/bundles/extension.html)  


