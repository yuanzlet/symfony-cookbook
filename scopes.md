# 如何使用作用域

这篇文章讲述的内容是关于作用域的，作用域是有关[服务容器](http://symfony.com/doc/current/book/service_container.html)的一个比较高级的主题。如果您曾经在创建一个服务的时候被提示有关于“作用域”的错误，那么这篇文章很值得您去阅读。

> 如果您想要注入**请求**服务，一个简单的解决方案就是反向注入 **request_stack** 服务，并通过调用 [getCurrentRequest()](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/RequestStack.html#getCurrentRequest()) 方法 (请参阅[注入请求](http://symfony.com/doc/current/book/service_container.html#book-container-request-stack)) 来访问当前请求。剩下的这些篇幅从理论和更先进的方式来谈论作用域。如果您正在因为请求服务而处理作用域问题，那么您只需注入 request_stack。

## 理解作用域

一个服务的作用域决定着服务的实例在容器中的使用范围。DependencyInjection 组件提供了两个通用的作用域:

**容器** (默认):

您每次请求相同的实例都会通过这个容器来调用它。

**原型**:

当您每次请求服务时都会创建新的实例。

[ContainerAwareHttpKernel](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/DependencyInjection/ContainerAwareHttpKernel.html) 还定义了第三种作用域:请求。这个作用域和请求捆绑在了一起，这意味着每个子请求都会创建一个新的实例，并且这个实例在请求之外就不能被使用了 (例如在 CLI 请求中) 。

## 示例：客户端作用域

与**请求**服务（请求服务具有一个简单的注释，请参照上面的介绍）不同的是，在默认的 Symfony 容器中不存在属于除了**容器**和**原型**以外的其它作用域。但是对于本文的意图来讲，我们想象着这里有另一个作用域**客户**端和一个属于它的 **client_configuration**（客户端布局） 服务。这并不是一个很常见的情况，不过在一个请求中，您可以使用这个方法多次进入或者退出**客户端**作用域，并且每个客户端都具有它自身的 **client_configuration** 服务。

作用域在一个服务的依赖项上添加了约束：一个服务不能依赖于作用域比较狭窄的服务组。比如，如果您创建一个通用的 my_foo 服务，但是您试图去注入 **client_configuration** 服务，当对这个容器进行编译的时候，您将会得到一个 [ScopeWideningInjectionException](http://api.symfony.com/2.7/Symfony/Component/DependencyInjection/Exception/ScopeWideningInjectionException.html) 异常提示。请阅读下方文本框的内容以获得更多信息。

> ## 作用域和依赖项

> 想象一下你已经配置了 **my_mailer** 服务。但是您还没有配置服务的作用域，因此容器的作用域是默认的。换句话说，每次您请求容器的 **my_mailer** 服务，您将会得到相同的对象。这通常就是您想要您的服务的工作方式。

> 然而，想象一下，您想让您的 **my_mailer** 服务中具有 **client_configuration** 服务，或许是因为您想在该服务中了解一些细节，比如“发件人"地址，并且把它作为构造函数的参数，下面有几个为什么会出现这个问题的原因:

> - 当请求 **my_mailer** 服务时，一个 **my_mailer**（在这里叫做 MailerA）的服务实例被创建了。同时 **client_configuration**（在这里叫做 ConfigurationA） 被传递给了它。

> - 当您的一个应用程序需要使用另外的客户端来处理一些事情。您可以使用让您的应用程序进入一个新的 **client_configuration** 作用域，并且给容器设置一个新的 **client_configuration** 服务的方法。我们称这个服务为 **ConfigurationB**.

> - 在您应用程序的某个位置，您再次请求 **my_mailer** 服务。 因为您的服务处于容器的范围内，所以只是相同的实例 (MailerA) 被重用了。但是这里有一个问题：MailerA 实例仍包含旧的 ConfigurationA 对象，但它现在**不是**具有（当前的 client_configuration 服务是 ConfigurationB）正确配置对象 。虽然这是一个微小且不易察觉的错误，但是，这个错误的组合可能会造成很大的错误，这就是为什么它不被允许的原因。

> 所以，这就是为什么会存在作用域的原因以及它们可能会导致的问题。请继续阅读本文去寻找通用的解决办法。

> 当然，一个服务如果依赖于具有很广泛的作用域的服务时完全没有问题的。

## 在一个比较狭窄的作用域内使用服务 

> 对于作用域问题，下述有两个解决办法：

- A）把您的服务作为依赖项放在同一个作用域下（或者更窄的作用域）。如果您依赖于 **client_configuration** 服务，那么这就意味着把您新的服务放在客户端作用域中（请参阅 [A）改变您服务的作用域](http://symfony.com/doc/current/cookbook/service_container/scopes.html#changing-service-scope)）

- B）将整个容器传递给您的服务，当您每次想确定您是否得到了正确的实例，您就可以从容器中检索您的依赖项--您的服务时在默认的容器依赖项中的，（请参见 [B）把容器作为您的服务的依赖项进行传递](http://symfony.com/doc/current/cookbook/service_container/scopes.html#changing-service-scope)））

> 在 Symfony 2.7 以前，有另一种基于同步服务的选择。然而，在 从 Symfony 2.7 版本开始，这种服务就被取消了。

## A）改变您的服务的范围

应该按照下面的定义来修改服务的作用域。在这个例子中假定在 **Mailer** 类中的构造函数的第一个参数是 **ClientConfiguration** 对象。

YMAL:

```
# app/config/services.yml
services:
    my_mailer:
        class: AppBundle\Mail\Mailer
        scope: client
        arguments: ["@client_configuration"]
```

XML:

```

<!-- app/config/services.xml -->
<services>
    <service id="my_mailer"
            class="AppBundle\Mail\Mailer"
            scope="client">
            <argument type="service" id="client_configuration" />
    </service>
</services>
```

PHP:

```
// app/config/services.php
use Symfony\Component\DependencyInjection\Definition;

$definition = $container->setDefinition(
    'my_mailer',
    new Definition(
        'AppBundle\Mail\Mailer',
        array(new Reference('client_configuration'),
    ))
)->setScope('client');
```

## B）把容器作为您服务的依赖项进行传递

并不是所有时候都可以把作用域设置得比较狭窄（比如：当树枝环境要把它作为一个依赖项的时候，一个树枝扩展必须在容器作用域内），在这些情况下，你可以向你的服务传递整个容器:

```
// src/AppBundle/Mail/Mailer.php
namespace AppBundle\Mail;

use Symfony\Component\DependencyInjection\ContainerInterface;

class Mailer
{
    protected $container;

    public function __construct(ContainerInterface $container)
    {
        $this->container = $container;
    }

    public function sendEmail()
    {
        $request = $this->container->get('client_configuration');
        // ... do something using the client configuration here
    }
}
```

> 请注意，不要把一个客户端的配置存储在一个对象的属性中，因为可能在将来调用这个对象的时候会出现在第一节中阐述的那种问题（除非 Symfony 没有检查到您出现了错误）。

该类的服务配置就如同下面的代码所示：

YAML:

```
# app/config/services.yml
services:
    my_mailer:
        class:     AppBundle\Mail\Mailer
        arguments: ["@service_container"]
        # scope: container can be omitted as it is the default
```

XML:

```
<!-- app/config/services.xml -->
<services>
    <service id="my_mailer" class="AppBundle\Mail\Mailer">
         <argument type="service" id="service_container" />
    </service>
</services>
```

PHP:

```
// app/config/services.php
use Symfony\Component\DependencyInjection\Definition;
use Symfony\Component\DependencyInjection\Reference;

$container->setDefinition('my_mailer', new Definition(
    'AppBundle\Mail\Mailer',
    array(new Reference('service_container'))
));
```

> 把整个容器注入服务通常不是一个很好的办法（您只用注入您需要的部分即可）。