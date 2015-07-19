# 如何掌握并创建新的环境

每一个应用程序都是代码和一系列规定了代码如何执行相应功能的设置的集合。设置可能定义了使用的数据库或者有些东西是否应该被缓存又或者冗长的日志应该如何处理。在 Symfony 中，“环境”的思想就是可以使用很多不同的设置运行相同的代码库。举例来说，**dev** 环境应当使用从而使得开发简单并且有好的设置，然而 **prod** 环境就应当使用优化速度的设置。  

## 不同的环境，不同的配置文件

一个典型的 Symfony 应用程序都是从以下三种环境开始的：**dev**，**prod** 以及 **test**。就像上面提到的，每一个环境就是简单的代表着用不同的配置执行相同的代码库的方式。那么也不用奇怪每一个环境都要加载自己独立的配置文件了。如果你使用的是 YAML 配置格式，将会用到下列文件：  

- 在 **dev** 环境下：**app/config/config_dev.yml**
- 在 **prod** 环境下：**app/config/config_prod.yml**
- 在 **test** 环境下：**app/config/config_test.yml**  

这个工作通过 AppKernel 类中默认使用的一个简单的标准起作用：  

```
// app/AppKernel.php

// ...

class AppKernel extends Kernel
{
    // ...

    public function registerContainerConfiguration(LoaderInterface $loader)
    {
        $loader->load($this->getRootDir().'/config/config_'.$this->getEnvironment().'.yml');
    }
}
```  

正如你所见的那样，当 Symfony 被加载时，它就会使用给定的环境来决定加载哪一个配置文件。它通过一种简洁，有效并且通俗易懂的方式达到了目标。  

当然，在现实情况下，每一个环境都会和其它的有一些不同。一般来讲，所有的环境都会共享很多共同的配置。打开 **config_dev.yml** 配置文件，你可以看到这是如何轻而易举地完成的：  

YAML:

```YAML
imports:
    - { resource: config.yml }

# ...
```  

XML:

```XML
<imports>
    <import resource="config.xml" />
</imports>

<!-- ... -->
```  

PHP:

```PHP
$loader->import('config.php');

// ...
```  

为了共享相同的配置每一个环境的配置文件首先会从核心配置文件（**config.yml**）输入。文件的 remainder 可以通过重写个别的参数的方式分离默认配置。举例来说，默认情况下，web_profiler 工具栏是不可用的。然而，在 **dev** 环境下，这个工具栏通过修正 **config_dev.yml** 配置文件的 **toolbar** 选项的值来激活：  

YAML:

```YAML
# app/config/config_dev.yml
imports:
    - { resource: config.yml }

web_profiler:
    toolbar: true
    # ...
```  

XML:

```XML
<!-- app/config/config_dev.xml -->
<imports>
    <import resource="config.xml" />
</imports>

<webprofiler:config toolbar="true" />
```  

PHP:

```PHP
// app/config/config_dev.php
$loader->import('config.php');

$container->loadFromExtension('web_profiler', array(
    'toolbar' => true,

    // ...
));
```  

## 在不同的环境下执行应用程序

在每一种环境下执行应用程序，使用前端控制器的 **app.php**（对于 **prod** 环境） 或者 **app_dev.php**（对于 **dev** 环境） 来加载应用程序：  

```
http://localhost/app.php      -> *prod* environment
http://localhost/app_dev.php  -> *dev* environment
```  

> 上面给定的链接地址是假设你的网页服务器设置使用应用程序的 **web/** 目录作为其根目录。获取更多信息详见[安装 Symfony](http://symfony.com/doc/current/book/installation.html)。  

如果你打开了这些目录中的一些文件，你将会看到每个使用过的环境被分别设置：  

```
// web/app.php
// ...

$kernel = new AppKernel('prod', false);

// ...
```  

**prod** 主键表明这个应用程序需要在 **prod** 环境下运行。Symfony 的应用程序可以通过这个代码在任何环境下运行并且可以改变环境字符串。  

> **test** 环境是用来编写功能性测试的并且不能直接通过前端的浏览器直接访问。换句话说，不像其它的环境那样，这里的前端控制器没有 **app_test.php** 文件。  

> ##调试模式  
> 重要的，但是和*环境*的问题并不相关的是 **false** 的争论作为 **AppKernel** constructor 的第二个争论。这个争论指出应用程序是否应当在“调试模式”下运行。不管环境，Symfony 应用程序可以通过调试模式设置成 **true** 或者 **false** 的情况下运行。这会影响到程序的很多东西，比如错误是否应该被展示，或者缓存文件需要在每一个请求上动态重新建立。尽管不是要求，调试模式通常只在 **dev** 和 **test** 环境下被设置成 **true**，在 **prod** 环境下被设置成 **false**。  
> 内在的，调试模式的值变成了 [service container](http://symfony.com/doc/current/book/service_container.html) 中使用过的 **kernel.debug** 参数。如果你看应用程序内部的配置文件，你就会看到过使用过的参数，举例来说，使用 Doctrine DBAL 打开或者关闭日志：  

>```YAML
>doctrine:
   dbal:
       logging: "%kernel.debug%"
       # ...
>```  

>```XML
><doctrine:dbal logging="%kernel.debug%" />
>```  

>```PHP
>$container->loadFromExtension('doctrine', array(
    'dbal' => array(
        'logging'  => '%kernel.debug%',
        // ...
    ),
    // ...
));
>```  

> Symfony 2.3 之中，是否展示错误不再依赖于调试模式。你需要在前端控制器中调用 [enable()](http://api.symfony.com/2.7/Symfony/Component/Debug/Debug.html#enable())。

### 为控制台命令选择环境

默认情况下，Symfony 的命令在 **dev** 环境下执行并且在调试模式下可用。使用 **--env** 和 **--no-debug** 选项可以修正这一行为：  

```
# 'dev' environment and debug enabled
$ php app/console command_name

# 'prod' environment (debug is always disabled for 'prod')
$ php app/console command_name --env=prod

# 'test' environment and debug disabled
$ php app/console command_name --env=test --no-debug
```  

除了 **--env** 和 **--no-debug** 选项之外，Symfony 命令的行为也可以通过环境变量控制。Symfony 控制应用程序在执行任何命令之前都会检查环境变量的存在以及值：  

**SYMFONY_ENV**
将命令的执行环境设置成这个变量的值（dev, prod, test,等等）；

**SYMFONY_DEBUG**
如果是 **0**，调试模式就是不可用。否则，调试模式就是可用。

这些环境变量对于服务器的产品有很大用处因为他们能保证在不添加任何命令选项的情况下命令一直运行在 **prod** 环境下。  

## 创建新的环境

在默认情况下，Symfony 应用程序拥有三个处理大多数情况的环境。当然，自从环境只不过是一个配置的字符串时，创建一个新的环境就变得很容易。  

举例来说，假设在开发之前，你需要用基准问题测试你的应用程序。一种方法是用基准问题测试应用程序使用接近的产品设置，但是要启用 Symfony 的 **web_profiler**。这就会允许 Symfony 记录关于你的应用程序在用基准问题测试时的信息。

通过调用新的环境是完成这件事的最好的方法，举例来说，**benchmark**。以创建一个新的配置文件开始：  

YAML:

```YAML
# app/config/config_benchmark.yml
imports:
    - { resource: config_prod.yml }

framework:
    profiler: { only_exceptions: false }
```  

XML:

```XML
<!-- app/config/config_benchmark.xml -->
<imports>
    <import resource="config_prod.xml" />
</imports>

<framework:config>
    <framework:profiler only-exceptions="false" />
</framework:config>
```  

PHP:

```PHP
// app/config/config_benchmark.php
$loader->import('config_prod.php')

$container->loadFromExtension('framework', array(
    'profiler' => array('only-exceptions' => false),
));
```  

>由于参数被分解的方式，你不能使用它们来动态建立输入路径。这也就意味着类似于下列的东西不会工作：  
>```YAML
># app/config/config.yml
imports:
    - { resource: "%kernel.root_dir%/parameters.yml" }
>```
>```XML
><!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">

    <imports>
        <import resource="%kernel.root_dir%/parameters.yml" />
    </imports>
</container>
>```
>```PHP
>// app/config/config.php
$loader->import('%kernel.root_dir%/parameters.yml');
>```

这个简单的添加，应用程序现在支持了一个名叫 **benchmark** 的新环境。  

这个新的配置文件从 **prod** 环境输入配置并且修正了它。这就保证了新的环境和 **prod** 环境是完全一致的，除了一些这里做出的一些明确的改变。  

因为你将想要通过浏览器访问这个环境，你还需要为它创建一个前端控制器。复制 **web/app.php** 文件到 **web/app_benchmark.php** 并且编辑环境成为 **benchmark**：  

```
// web/app_benchmark.php
// ...

// change just this line
$kernel = new AppKernel('benchmark', false);

// ...
```  

这个新的环境现在可以通过下面的代码访问：  

```
http://localhost/app_benchmark.php
```  

>一些环境，像 **dev** 环境，从来都不是为了任何公共部署的服务器的访问。这是因为确定的环境，为了调试的目的，可能给出了太多的关于应用程序或者基础的结构的信息。为了确保这些环境不能被访问，前端控制器通常通过控制器顶端的下列代码来保护它不受外部 IP 地址的伤害：  
>```
>if (!in_array(@$_SERVER['REMOTE_ADDR'], array('127.0.0.1', '::1'))) {
    die('You are not allowed to access this file. Check '.basename(__FILE__).' for more information.');
}
>```

## 环境和缓存目录

Symfony 利用缓存的方法有很多种：应用程序配置，路由配置，Twig 模板以及很多都缓存到文件系统中储存的 PHP 对象文件中。  

默认设置下，这些缓存文件大多储存在 **app/cache** 目录下。然而，每个环境缓存有它自己的文件集合：  

```
<your-project>/
├─ app/
│  ├─ cache/
│  │  ├─ dev/   # cache directory for the *dev* environment
│  │  └─ prod/  # cache directory for the *prod* environment
│  ├─ ...
```  

有时候，当在调试的时候，看懂缓存文件如何工作很有帮助。当这么做的时候，记住要看看正在使用的环境的目录（大多数的开发与调试都是在 **dev** 环境下）。然而也可能不是， **app/cache/dev** 目录下包含了以下文件：  

**appDevDebugProjectContainer.php**  
缓存的"service container"代表了缓存的应用程序配置。

**appDevUrlGenerator.php**  
从路由配置中产生的 PHP 类并且在产生链接时使用。  

**appDevUrlMatcher.php**  
路由匹配使用的 PHP 类——看这里来了解编译的普通的匹配收到的链接的不同路由的逻辑。  

**twig/**  
这个目录包含了所有缓存的 Twig 模板。  

>你可以很方便的改变目录位置和名称。获取更多信息可以阅读名为[如何重写 Symfony 的默认目录结构](http://symfony.com/doc/current/cookbook/configuration/override_dir_structure.html)的文章。  

## 更深入的学习

阅读[如何在服务容器内设置外部参数](http://symfony.com/doc/current/cookbook/configuration/external_parameters.html)。  