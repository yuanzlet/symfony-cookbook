# 如何安装第三方 Bundles

大多数的 bundles 都提供自己的安装指南，然而，基本的 bundles 安装步骤都是一样的：  

- A)添加 Composer 依赖
- B)启用 Bundle  
- C)设置 Bundle  

## A)添加 Composer 依赖

依赖是由 Composer 管理的，所以如果你不太了解 Composer，你可以在[它的相关文档](https://getcomposer.org/doc/00-intro.md)中学习一些基本理论。这包括了两步：  

### 1) 找出 Packagist 上的 Bundle 的名字

bundle 的 README 文件（例如 [FOSUserBundle](https://github.com/FriendsOfSymfony/FOSUserBundle)通常会告诉你它的名字（例如 **friendsofsymfony/user-bundle**）。如果不行，你可以在 [Packagist.org](https://packagist.org/) 网站上搜索 bundle。  

>寻找 bundle？试一试在 [KnpBundles.com](http://knpbundles.com/) 网站上搜索：非官方的 Symfony Bundles 的档案馆。  

### 2)通过 Composer 安装 Bundle

既然你知道了包的名字，你可以通过 Composer 安装它：  

```
$ composer require friendsofsymfony/user-bundle
``` 

这个将会为你的工程选择最佳的版本，添加到 **composer.json** 并且下载它的代码到 **vendor/** 目录下。如果你需要特定的版本，将它作为 [composer require](https://getcomposer.org/doc/03-cli.md#require) 命令的第二个参数：  

```
$ composer require friendsofsymfony/user-bundle "~2.0"
```  

## B)启用 Bundle

这时候，bundle 已经安装在你的 Symfony 工程(在 **vendor/friendsofsymfony/** 之中) 并且自动装载识别出了它的类。现在你需要做的唯一一件事就是在 **AppKernel** 中注册  bundle：  

```
// app/AppKernel.php

// ...
class AppKernel extends Kernel
{
    // ...

    public function registerBundles()
    {
        $bundles = array(
            // ...
            new FOS\UserBundle\FOSUserBundle(),
        );

        // ...
    }
}
```  

在一些特别的案例中，你可能想让 bundle *仅仅*在开发[环境](http://symfony.com/doc/current/cookbook/configuration/environments.html)下可用。举例来说，DoctrineFixturesBundle 帮助你来加载虚拟数据——一些你可能只想在开发的时候做的。只是在 **dev** 和 **test** 环境下装载 bundle，按照下面的方法来注册 bundle：  

```
// app/AppKernel.php

// ...
class AppKernel extends Kernel
{
    // ...

    public function registerBundles()
    {
        $bundles = array(
            // ...
        );

        if (in_array($this->getEnvironment(), array('dev', 'test'))) {
            $bundles[] = new Doctrine\Bundle\FixturesBundle\DoctrineFixturesBundle();
        }

        // ...
    }
}
```  

## C)设置 Bundle

对于 bundle 来说需要一些额外的设置或者在 **app/config/config.yml** 的调整是很正常的事。bundle 的说明文档会告诉你有关的设置，但是你也需要通过 **config:dump-reference** 命令来获取一些 bundle 的设置指导：  

```
$ app/console config:dump-reference AsseticBundle
```  

代替 bundle 的全名，你也可以用使用过的短名字作为 bundle 的设置的根：  

```
$ app/console config:dump-reference assetic
```  

输出结果如下所示：  

```
assetic:
    debug:                %kernel.debug%
    use_controller:
        enabled:              %kernel.debug%
        profiler:             false
    read_from:            %kernel.root_dir%/../web
    write_to:             %assetic.read_from%
    java:                 /usr/bin/java
    node:                 /usr/local/bin/node
    node_paths:           []
    # ...
```  

## 其它的安装

在这里，阅读你的全新的 bundle 的 **README** 文件来看看接下来做什么。玩的开心！