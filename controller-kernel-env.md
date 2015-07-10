# 理解前端控制器、内核及环境如何协同工作

在[如何掌握并创建新的环境](http://symfony.com/doc/current/cookbook/configuration/environments.html)这一节中我们解释了 Symfony 如何在不同的配置之下使用环境来运行的应用程序的基本思想。这一节我们将会更加深入地介绍当你的应用程序自我启动时会发生什么。为了理解这一过程，你必须理解一起运行的三个部分：  

- [前端控制器](http://symfony.com/doc/current/cookbook/configuration/front_controllers_and_kernel.html#the-front-controller)
- [Kernel 类](http://symfony.com/doc/current/cookbook/configuration/front_controllers_and_kernel.html#the-kernel-class)
- [环境](http://symfony.com/doc/current/cookbook/configuration/front_controllers_and_kernel.html#the-environments)

>通常情况下，你不需要定义你自己的前端控制器或者 **AppKernel** 类由于 [Symfony 的标准版本](https://github.com/symfony/symfony-standard)提供了默认的安装设置。  
>这一节的文档就是来解释表象之后发生的事情的。  

## 前端控制器 ##

[前端控制器](http://en.wikipedia.org/wiki/Front_Controller_pattern)是人们所熟知的设计样式；它是由一些应用程序运行的代码的*所有*请求形成的代码组成的。  

在 [Symfony 的标准版本](https://github.com/symfony/symfony-standard)中，这个角色被 **web/** 目录下的  [app.php](https://github.com/symfony/symfony-standard/blob/master/web/app.php) 和 [app_dev.php](https://github.com/symfony/symfony-standard/blob/master/web/app_dev.php) 文件所取代。这些是当一个请求被处理时最先执行的 PHP 脚本。  

前端控制器的主要目的就是创建一个 **AppKernel** 的实例（稍后再作更多的介绍），使得它可以处理请求并且向浏览器返回处理结果。  

由于每一个请求都是经过它按路线发送的，前端控制器可以被用来全局的优先初始化来建立 kernel 或者使用附加特征[装饰](http://en.wikipedia.org/wiki/Decorator_pattern) kernel。例子包括：  

- 设置自动加载或者添加自动加载机制；
- 添加 HTTP 层的缓存通过使用 [AppCache](http://symfony.com/doc/current/book/http_cache.html#symfony-gateway-cache) 的实例封装；
- 启用（或者跳过）[ClassCache](http://symfony.com/doc/current/cookbook/debugging.html)；
- 启用 [Debug Component](http://symfony.com/doc/current/components/debug/introduction.html)。

前端控制器可能被请求链接地址选择就像下面所示：  

```
http://localhost/app_dev.php/some/path/...
```  

正如你所见，这个链接包含了用来作为前端控制器的 PHP 脚本。你可以用它轻松地切换前端控制器或者使用一个放置在 **web/** 目录中的定制的控制器（例如 **app_cache.php**）。  

当使用 Apache 以及[由 Symfony 的标准版本所定义的 RewriteRule](https://github.com/symfony/symfony-standard/blob/master/web/.htaccess) 时，你可以忽略链接中的文件名并且 RewriteRule 会将 **app.php** 作为默认的。  

>基本上每一个其他的网页服务器都应该可以表现出一种和上面描述的 RewriteRule 相似的行为。阅读你的服务器文档查看更多细节或者可以看[配置网页服务器](http://symfony.com/doc/current/cookbook/configuration/web_server_configuration.html)。  

>确保你的前端控制器不被未授权访问。举例来说，你不想对于任意的你的生产环境的用户都开放调试模式。  

从技术层面来讲，在命令行运行 Symfony 时使用的 [app/console](https://github.com/symfony/symfony-standard/blob/master/app/console) 脚本也是前端控制器，只有不是为网页应用，但是是命令行请求。  

## Kernel 类 ##

[Kernel](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/Kernel.html) 类是 Symfony 的核心。它负责建立组成你的应用程序的所有 bundle，并且向他们提供应用程序的设置。然后它用它的 [handle()](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/HttpKernelInterface.html#handle()) 方法在服务请求之前创建服务容器。  

[KernelInterface](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/KernelInterface.html) 中声明了在 [Kernel](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/Kernel.html) 没有应用的两种方法并且因此将他们作为[模板方法](http://en.wikipedia.org/wiki/Template_method_pattern)提供：  

[registerBundles()](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/KernelInterface.html#registerBundles())  
它必须返回一批运行应用程序所需要的所有 bundle。  

[registerContainerConfiguration()](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/KernelInterface.html#registerContainerConfiguration())  
它负责加载应用程序配置。  

为了填补这些（小的）空白，你的应用程序需要把 Kernel 划为子类并且实施这些方法。按照惯例结果类叫做 **AppKernel**。  

除此之外，Symfony 标准版本在 **app/** 目录下提供了 [AppKernel](https://github.com/symfony/symfony-standard/blob/master/app/AppKernel.php)。这个类是使用了环境的名称——环境的名称是传给 Kernel 的 [constructor](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/Kernel.html#__construct()) 方法以及可以使用 [getEnvironment()](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/Kernel.html#getEnvironment()) 方法获得的——来决定创建哪个 bundle。这个的逻辑是在 **registerBundles()** 中，当你向你的应用程序中添加 bundle 时方法意味着你扩展的。  

当然，你也可以自己创建，有选择性的或者附加的 **AppKernel** 变体。你所需要的就是适应（或者添加新的）你的前端控制器并且会使用心得 Kernel。  

>**AppKernel** 的名称和位置不能修复。当将多个 Kernel 放入一个简单的应用程序中，这也许会使得添加附加的子目录变得有意义，举例来说 **app/admin/AdminKernel.php** 和 **app/api/ApiKernel.php**。所有这些重要的就是你的前端控制器可以创建适当的 kernel 实例。  

拥有不同的 **AppKernels** 可能会对于启用不同的前端控制器（可能在不同的服务器上）来独立运行你的应用程序的部分很有用（举例来说，admin UI, front-end UI 和数据库迁移）。  

>**AppKernel** 还可以干很多事，例如[重写默认目录结构](http://symfony.com/doc/current/cookbook/configuration/override_dir_structure.html)。但是你你实施几个 **AppKernel** 不需要像这样忙忙碌碌改变的几率是很高的。  

## 环境 ##

就像刚才提到的，**AppKernel** 必须要调用其他方法——[registerContainerConfiguration()](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/KernelInterface.html#registerContainerConfiguration())。这个方法是从正确的*环境*中加载应用程序的设置的。  

环境已经在[前面的章节中](http://symfony.com/doc/current/cookbook/configuration/environments.html)详细介绍过了，你可能也记得 Standard 标准版本有三种环境——**dev**, **prod** 和 **test**。  

更技术的讲，这些名称只不过是从前端控制器传递到 **AppKernel** 的构造器的字符串。这个名称之后可以应用在 [registerContainerConfiguration()](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/KernelInterface.html#registerContainerConfiguration()) 方法中来决定加载哪个文件。  

Symfony 标准版本中的 [AppKernel](https://github.com/symfony/symfony-standard/blob/master/app/AppKernel.php) 类只是简单地通过加载 **app/config/config_*environment*.yml** 文件来实施这个方法。当然，你如果需要更加精致的加载你的应用程序的方法的话也可以区别地使用这个方法。  


