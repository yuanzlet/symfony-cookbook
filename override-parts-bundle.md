# 如何使用 Bundle 的继承来重写部分 Bundle

在使用第三方 bundle 的时候，你很可能有这样一个想法，就是你想用你自己的 bundle 文件重写那个第三方的 bundle 文件。Symfony 为你提供了一个非常方便的方法来重写诸如 controllers, templates 以及其它的在 bundle 的 **Resources/** 目录中的文件。  

举例来说，假如你正在安装 [FOSUserBundle](https://github.com/friendsofsymfony/fosuserbundle)，但是你想重写它的基础的 **layout.html.twig** 模板，同时也是它的一个控制台，假设你也拥有自己的 UserBundle，在那里你想要让重写文件存在。这就要通过将你的 FOSUserBundle 注册成你的 bundle 的“父类”开始：  

```  
// src/UserBundle/UserBundle.php
namespace UserBundle;

use Symfony\Component\HttpKernel\Bundle\Bundle;

class UserBundle extends Bundle
{
    public function getParent()
    {
        return 'FOSUserBundle';
    }
}
```  

通过这个细小的改动，你可以通过简单的创建同名文件的方式来重写 FOSUserBundle 的一些部分。  

> 不管方法的名称，bundle 之间没有父子关系，这就是一种扩展和重写已经存在的 bundle 的一种方法。  

## 重写 Controllers

假设你想向 FOSUserBundle 内部的 **RegistrationController** 中的 **registerAction** 添加一些功能。这样做，就创建你自己的 RegistrationController.php 文件，重写 bundle 的原始方法并且改变它的功能：  

```
// src/UserBundle/Controller/RegistrationController.php
namespace UserBundle\Controller;

use FOS\UserBundle\Controller\RegistrationController as BaseController;

class RegistrationController extends BaseController
{
    public function registerAction()
    {
        $response = parent::registerAction();

        // ... do custom stuff
        return $response;
    }
}
```  

> 基于你需要如何改变行为的程度，你可能调用 **parent::registerAction()** 或者完全用你自己的逻辑替换它。  

> 以这种方式重写 controller 只是在 bundle 引用了在路由和模板中使用标准 **FOSUserBundle:Registration:register** 语法的 controller 的情况下是有效的。这是最好的实践案例。  

## 重写资源：模板, 路由等等

大多数的资源也可以被重写，只需要简单地在你的父 bundle 相同的位置创建一个文件。  

举例来说，通常都需要重写 FOSUserBundle 的 **layout.html.twig** 模板这样你就可以应用你自己的应用程序的基础布局。由于这个文件在 FOSUserBundle 的 **Resources/views/layout.html.twig** 目录下，你可以在 UserBundle 的相同位置创建你自己的文件。Symfony 将会完全忽视 FOSUserBundle 内的文件，并且用你的文件作为替代。  

路由文件以及一些其它的资源也是如此。  

> 重写资源文件只是会在应用 **@FOSUserBundle/Resources/config/routing/security.xml** 方法的资源上起作用。如果你的资源没有应用 **@BundleName** 快捷键，那么将不能应用这种方法重写。  

> Translation 和 validation 文件并不是像上述描述的一样工作。如果你想重写 translations，请阅读"[Translations](http://symfony.com/doc/current/cookbook/bundles/override.html#override-translations)"，阅读"[Validation Metadata](http://symfony.com/doc/current/cookbook/bundles/override.html#override-validation)" 获取关于重写 validation 的方法。