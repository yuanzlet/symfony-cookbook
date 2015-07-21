# 如何在 Bundle 中使用 Compiler Passes

Compiler Passes 让您有机会去操作已经注册到其它服务容器中的服务定义。您可以阅读组件部分的文章“[编制容器]( http://symfony.com/doc/current/components/dependency_injection/compilation.html)” 来了解怎么去创建它们。如果您想从一个 bundle 类中注册编译器，那么您需要把它添加到 bundle 类定义中的构造方法中：

```
// src/Acme/MailerBundle/AcmeMailerBundle.php
namespace Acme\MailerBundle;

use Symfony\Component\HttpKernel\Bundle\Bundle;
use Symfony\Component\DependencyInjection\ContainerBuilder;

use Acme\MailerBundle\DependencyInjection\Compiler\CustomCompilerPass;

class AcmeMailerBundle extends Bundle
{
    public function build(ContainerBuilder $container)
    {
        parent::build($container);

        $container->addCompilerPass(new CustomCompilerPass());
    }
}
```

最常见的关于 Compiler Passes 的用例是标记服务 (想要了解更多关于标签的内容请参阅组件部分的内容"[标记服务的运用](http://symfony.com/doc/current/components/dependency_injection/tags.html)") 。如果您要使用 bundle 类中的自定义标签名，随后使用标记名来包含 bundle 的名称（使用小写字母，下划线作为分隔符），然后加一个点，最后再跟着真正的名称。比如：如果您希望在您的 AcmeMailerBundle 类中引入某种形式的"传输"标记，可以称之为 **acme_mailer.transport**。