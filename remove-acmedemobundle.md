# 如何移除 AcmeDemoBundle

Symfony 的标准版本来自于一个完整的 demo,这个 demo 位于一个名叫 AcmeDemoBundle 的 bundle 中。在开始一个工程的时候它是一个很好的可以引用的样板文件，但是最终你可能会想要移除它。  

> 这篇文章使用 AcmeDemoBundle 作为例子，但是你可以使用这些步骤移除任何的 bundle。  

## 1.在 AppKernel 中移除 bundle 的注册

为了从框架中解除 bundle 的联系，你应当从 **AppKernel::registerBundles()** 方法中移除 bundle。bundle 通常会出现在 **$bundles** 数组之中但是 AcmeDemoBundle 是唯一的在开发环境中注册的并且你可以在下面的声明中找到它：  

```
// app/AppKernel.php

// ...
class AppKernel extends Kernel
{
    public function registerBundles()
    {
        $bundles = array(...);

        if (in_array($this->getEnvironment(), array('dev', 'test'))) {
            // comment or remove this line:
            // $bundles[] = new Acme\DemoBundle\AcmeDemoBundle();
            // ...
        }
    }
}
```  

## 2.移除 Bundle 的配置

既然 Symfony 不知道 bundle，那么你需要移除在 **app/config** 之中的有关 bundle 的任何配置以及路由配置。  

### 2.1 移除 bundle 的路由配置

AcmeDemoBundle 的路由配置可以在 **app/config/routing_dev.yml** 中找到。移除位于文件根部的 **_acme_demo** 入口。

### 2.2 移除 bundle 的配置

一些 bundle 的配置包含在 **app/config/config*.yml** 文件的其中之一。你要确保从这些文件中移除了相关的配置。你可以通过寻找配置文件中的 **acme_demo** 字符串（或者不论 bundle 的名称是什么，例如 FOSUserBundle 的 **fos_user**）来发现 bundle 的配置。  

AcmeDemoBundle 没有配置。然而，bundle 被用于 **app/config/security.yml** 文件的配置。你可以用它作为你的样板文件为了你自己的安全，但是你也**可以**移除所有的东西：对于 Symfony 来说你是否移除都是无所谓的。  

## 3.从文件系统中移除 bundle

现在你已经从你的应用程序中移除了所有有关的 bundle 的东西，你还需要从文件系统中移除它。bundle 位于 **src/Acme/DemoBundle** 的目录之下。你应当移除这个目录，同时你也可以移除 **Acme** 目录。  

>如果你不知道 bundle 的目录，你可以使用 [getPath()](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/Bundle/BundleInterface.html#getPath()) 方法来获取 bundle 的目录：  
```
dump($this->container->get('kernel')->getBundle('AcmeDemoBundle')->getPath());
die();
```  

### 3.1 移除 bundle 资产

移除位于网页或目录（例如 AcmeDemoBundle 的 **web/bundles/acmedemo**）中的资产。  

## 4.移除与其他 Bundle 的集合

> 这个并不适用于 AcmeDemoBundle 因为没有 bundle 依赖于它，所以你可以跳过这一步。  

一些 bundle 依赖于其它的 bundle，如果你移除了它们其中之一，那么另外一个很可能也就不可用了。确保没有其它的第三方的或者自己做的 bundle 依赖于你所要移除的 bundle。  

> 如果一个 bundle 依赖于另一个 bundle 的话，在大多数情况下这就意味着它从 bundle 中应用服务。寻找 bundle 的别名有助于你发现这种关系（例如 bundle 的 **acme_demo** 别名就依赖于 AcmeDemoBundle）。
> 如果一个第三方的 bundle 依赖于另一个 bundle 的话，你可以在 bundle 目录的 **composer.json** 文件中找到所提及的 bundle。