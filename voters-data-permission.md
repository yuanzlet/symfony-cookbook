# 如何使用 Voter 检查用户权限

在Symfony里，你可以用[ACL模块](http://symfony.com/doc/current/cookbook/security/acl.html)来检测用户的数据使用权限，即使这个模块显得有些过于强势。一个更简单的解决方法就

是与顾客接触，利用他们的Voter来作为判定条件。

> Voters 也能用其他方式被用到，比如，用户IP黑名单就来自整个应用程序：[How to Use Voters to Check User Permissions](http://symfony.com/doc/current/cookbook/security/).

> 看看 [authorization](http://symfony.com/doc/current/components/security/authorization.html) 这一章可以对 voter 有更深刻的理解

## Symfony 怎么使用 voter

为了合理使用 voter，我们必须了解 Symfony 与 voter 的互动机制。所有的 voter 都被 **isGranted()** 函数申请调用，同时检查他们的权限（比如**security.authorization_checker** 服务）。Voter 的每一次的决定，都会接触到相应的资源。

从根本上，Symfony 从所有的 voter 里收集响应，通过在应用程序里制定的相对一致可行且合理的策略，同时结合它们做出最后的判决（允许或者拒绝访问申请）。

如果要获得更充分的信息，可以参考 [访问决策管理器相关章节](http://symfony.com/doc/current/components/security/authorization.html#components-security-access-decision-manager).

## Voter 接口

一个自定义 voter 需要使用 [VoterInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Authorization/Voter/VoterInterface.html) 接口，或者其子接口[AbstractVoter](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Authorization/Voter/AbstractVoter.html) ，这个接口可以让创建一个 voter 对象更简单便捷一点。

```
abstract class AbstractVoter implements VoterInterface
{
    abstract protected function getSupportedClasses();
    abstract protected function getSupportedAttributes();
    abstract protected function isGranted($attribute, $object, $user = null);
}
```

在这个例子里，voter会检查他们自己是否可以得到一个特定的针对他们自定义条件的对象（比如他们必须是一个是这个对象的所有者）。如果条件检测失败，那么就会返回**VoterInterface::ACCESS_DENIED**,否则就会返回 **VoterInterface::ACCESS_GRANTED** .如果投票者完全没有处理权限的话，就会返回 **VoterInterface::ACCESS_ABSTAIN**。

## 创建voter

我们的目标是创建一个voter，来检测用户能否读入或者编辑一个特定的对象，下面是一个实现特例：

```
// src/AppBundle/Security/Authorization/Voter/PostVoter.php
namespace AppBundle\Security\Authorization\Voter;

use Symfony\Component\Security\Core\Authorization\Voter\AbstractVoter;
use Symfony\Component\Security\Core\User\UserInterface;

class PostVoter extends AbstractVoter
{
    const VIEW = 'view';
    const EDIT = 'edit';

    protected function getSupportedAttributes()
    {
        return array(self::VIEW, self::EDIT);
    }

    protected function getSupportedClasses()
    {
        return array('AppBundle\Entity\Post');
    }

    protected function isGranted($attribute, $post, $user = null)
    {
        // make sure there is a user object (i.e. that the user is logged in)
        if (!$user instanceof UserInterface) {
            return false;
        }

        switch($attribute) {
            case self::VIEW:
                // the data object could have for example a method isPrivate()
                // which checks the Boolean attribute $private
                if (!$post->isPrivate()) {
                    return true;
                }

                break;
            case self::EDIT:
                // we assume that our data object has a method getOwner() to
                // get the current owner user entity for this data object
                if ($user->getId() === $post->getOwner()->getId()) {
                    return true;
                }

                break;
        }

        return false;
    }
}
```

情况就是如此。Voter 已经创建完成了，下一步是把voter移动到安全层里。

简明扼要的说，这里我们用三个抽象方法来实现我们的目标：

[getSupportedClasses()](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Authorization/Voter/AbstractVoter.html#getSupportedClasses())

它告知Symfony，每当一个被给定类的对象传递给 **isGranted()** 方法时，你的 voter 就应该被调用。比如说，如果你返回了 **array('AppBundle\Model\Product')**, 当一个 

**Product** 对象传递给 **isGranted()** 方法时，Symfony 就可以调用 voter。

[getSupportedAttributes()](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Authorization/Voter/AbstractVoter.html#getSupportedAttributes())

它告知 Symfony，每当一些给定字符串作为第一个参数传递给 **isGranted()** 方法时，应当调用 voter。比如，如果你返回了 **array('CREATE','READ')**,当它们中的一个被发送到 **isGranted()** 时，Symfony 就会调用 voter。

[isGranted()](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Authorization/Voter/AbstractVoter.html#isGranted())

这个方法采用了商业逻辑，来核实是否允许未被给定的用户访问给定对象的给定属性（例如，**create** 或 **read**），这个方法必须返回 boolean 类型的值。

> 目前，使用 **AbstractVoter** 基类，您必须创建一个对象总是传递给 **isGranted()** 的 voter。

## 声明voter是一项服务

将 voter 归入安全层，你就必须把它声明为一项服务，然后贴上 **security.voter:** 的标签：

YAML:
```
# src/AppBundle/Resources/config/services.yml
services:
    security.access.post_voter:
        class:      AppBundle\Security\Authorization\Voter\PostVoter
        public:     false
        tags:
           - { name: security.voter }
```

XML:
```
<!-- src/AppBundle/Resources/config/services.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">
    <services>
        <service id="security.access.post_voter"
            class="AppBundle\Security\Authorization\Voter\PostVoter"
            public="false">
            <tag name="security.voter" />
        </service>
    </services>
</container>
```

PHP:
```
// src/AppBundle/Resources/config/services.php
$container
    ->register(
            'security.access.post_voter',
            'AppBundle\Security\Authorization\Voter\PostVoter'
    )
    ->setPublic(false)
    ->addTag('security.voter')
;
```

## 如何在一个控制器里使用voter

一个已注册的 voter 会在 **isGranted()** 函数从授权检查中被调用的时候调用。

```
// src/AppBundle/Controller/PostController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Response;

class PostController extends Controller
{
    public function showAction($id)
    {
        // get a Post instance
        $post = ...;

        $authChecker = $this->get('security.authorization_checker');

        if (false === $authChecker->isGranted('view', $post)) {
            throw $this->createAccessDeniedException('Unauthorized access!');
        }

        return new Response('<h1>'.$post->getName().'</h1>');
    }
}
```

> 2.6 **security.authorization_checker** 服务在 Symfony2.6 里被介绍，在 Symfony2.6 之前的版本里，你不得不用 **security.context** 服务里面的 **isGranted()** 方法。

就是这么简单！

## 改变访问决策策略。

想象一下你对于每一个对象的每一种行为有多种多样的 voter。比如说，你有一个 voter 用来检测一个站点的使用成员是否已经超过了18岁。

为了应对这些情况，访问决策管理者用一种决策管理策略。你可以根据你的需求安装一组。这里有三种可使用的策略：

**affirmative**（default）

给予授权当一个 voter 允许授权的时候

**consensus**

当有很多 voter 允许授权而不是被拒绝的时候给予授权

**unanimous**

只有所有 voters 都允许授权的时候给予授权

在上述情形下，所有的 voters 都应该允许访问以便授予用户读取 post 的访问权限。在这种情况下，默认策略可能会不再有效，而且 **unanimous** 应当被取代。你可以在安全配置里设置这些参数。

YAML：
```
# app/config/security.yml
security:
    access_decision_manager:
        strategy: unanimous
```

XML：
```
<!-- app/config/security.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<srv:container xmlns="http://symfony.com/schema/dic/security"
    xmlns:srv="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/security
        http://symfony.com/schema/dic/security/security-1.0.xsd"
>
    <config>
        <access-decision-manager strategy="unanimous">
    </config>
</srv:container>
```

PHP：
```
// app/config/security.php
$container->loadFromExtension('security', array(
    'access_decision_manager' => array(
        'strategy' => 'unanimous',
    ),
));
```
	
	
