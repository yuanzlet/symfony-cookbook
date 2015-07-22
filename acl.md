# 如何使用访问控制列表（ACLs）

在复杂的申请中，你可能经常面临访问权限决策不仅仅取决于请求者本人（**Token**）还牵涉到了被申请的对象的问题。这也是 ACLs 系统被制作出来的原因所在。

> ## ACLs的替代选择

> 使用 ACL 并不是细微之事，它的的确确对使用有简化功能。当然，它可能会滥杀无辜。如果你的判定逻辑只是被简单的代码来描述（比如检查这个博客是否被一个现在的使用者所拥有> ），那么就考虑使用 voters。一个 voter 可以通过表决来传递对象，通过这些，你就可以做出复杂的决定和更高效地执行你的 ACL。此外，强制批准（比如 isGranted 部分）就会看起来和你所看到的这个条目极其相似，但是你的 voter 类就会在幕后控制判定逻辑了，而不是 ACL 系统。

想象你在设计一个博客系统，而你的使用者可以评论你的工作。现在，如果你希望一个使用者能够修改编辑他们自己的评论，但并不是所有的用户；此时，你可以修改所有的评论。在这种情况下，**comment** 就会处理域名对象，同时你会获得权限。你可以采用多种方式来完成这个 Symfony，两个基本的方式如下：

- *Enforce security in your business methods:*基本上讲，这个方法意味着在每一个 **Comment** 之间制作一个参照，比较这些使用者提供的**令牌**，就可以做出决策。

- *Enforce security with roles*:在这种方法中，你可以为每一个 **Comment** 对象添加一些角色。比如 **ROLE_COMMENT_1**, **ROLE_COMMENT_2** 等等。

每种方法都非常有效。然而，它们结合了你的授权逻辑来负责你的商业代码，会限制你的代码的可通用型，因此这就增加了单元调试难度。此外，你可能会撞上一些问题，如果用户只有一个简单的域名对象的话。

幸运的是，这里有一种更好的方式，你在下面就可以看到。

## 引导指令

现在，在你能够采取行动之前，你需要做一些引导指令。首先，你需要安装你要实用的 ACL 系统的连接。

YAML

```
# app/config/security.yml
security:
    acl:
        connection: default
```

XML

```
<!-- app/config/security.xml -->
<acl>
    <connection>default</connection>
</acl>
```

PHP

```
// app/config/security.php
$container->loadFromExtension('security', 'acl', array(
    'connection' => 'default',
));
```

> ACL 体系要求一种连接关系，这种连接关系可以由 DBAL 来提供，也可以由 MongoDB（使用 [MongoDBAclBundle](https://github.com/IamPersistent/MongoDBAclBundle)）来提供。然而，那并不意味着你必须用 DoctrineORM 或者 ODM 来组织你的域对象。你可以用任意组织对象的方法和手段，比如 DoctrineORM，MongoDB ODM，Propel，rawSQL 等等。选择权在你手里。

在连接方式确定好之后，你就需要来输入基础的数据结构了。幸运的是，有一项任务专门处理这种情况，只需要运行一下面的指令就可以了：

```
$ php app/console init:acl
```

## 开始工作

回到最开始的小例子上去，现在你可以对它运用 ACL 技术了。

一旦 ACL 被建立起来，你就可以通过建立一个 Access Control Entry，来向你的用户提供信息获取通道。当然同时就可以稳固你的使用者和你的工作实体之间的联系了。

### 建立一个 ACL，添加一个 ACE

```
// src/AppBundle/Controller/BlogController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\Security\Core\Exception\AccessDeniedException;
use Symfony\Component\Security\Acl\Domain\ObjectIdentity;
use Symfony\Component\Security\Acl\Domain\UserSecurityIdentity;
use Symfony\Component\Security\Acl\Permission\MaskBuilder;

class BlogController extends Controller
{
    // ...

    public function addCommentAction(Post $post)
    {
        $comment = new Comment();

        // ... setup $form, and submit data

        if ($form->isValid()) {
            $entityManager = $this->getDoctrine()->getManager();
            $entityManager->persist($comment);
            $entityManager->flush();

            // creating the ACL
            $aclProvider = $this->get('security.acl.provider');
            $objectIdentity = ObjectIdentity::fromDomainObject($comment);
            $acl = $aclProvider->createAcl($objectIdentity);

            // retrieving the security identity of the currently logged-in user
            $tokenStorage = $this->get('security.token_storage');
            $user = $tokenStorage->getToken()->getUser();
            $securityIdentity = UserSecurityIdentity::fromAccount($user);

            // grant owner access
            $acl->insertObjectAce($securityIdentity, MaskBuilder::MASK_OWNER);
            $aclProvider->updateAcl($acl);
        }
    }
}
```

在上面的代码段中有一些重要的实施策略。现在，我仅仅希望能强调两点：

首先，你可能注意到 **->createAcl()** 不直接接受域对象，而只接受 **ObjectIdentityInterface** 的启用。当你手里没有实际的区域对象实例的时候，这个附加步骤间接地允许你能够和 ACLs 交互。这在你想要检查一大批对象的权限的时候，将会起到很大的作用。

另一个有趣的部分在于 **->insertObjectAce()** 的调用。在例子里，我们可以授予直接联机的用户以修改评论的权限。而 **MaskBuilder::MASK_OWNER** 是一个提前定义的整型位掩码；不必担心掩码生成器会抽象大部分细节，但是这种技术会帮助你从数据库里得到一系列不同的权限数据，从而让你的表现能够显得更漂亮一些。

> ACEs 的检查顺序是很有意义的，作为一项通用的准则，你应该在最开始设立更多的入口。

## 检测通道

```
// src/AppBundle/Controller/BlogController.php

// ...

class BlogController
{
    // ...

    public function editCommentAction(Comment $comment)
    {
        $authorizationChecker = $this->get('security.authorization_checker');

        // check for edit access
        if (false === $authorizationChecker->isGranted('EDIT', $comment)) {
            throw new AccessDeniedException();
        }

        // ... retrieve actual comment object, and do your editing here
    }
}
```

在这个例子里，你可以检测用户是否有**编辑**权限。在 Symfony 内部，Symfony 会分配权限给几个整型的位掩码，然后检测用户是否拥有这些码。

> 你可以建立起 32 位的权限码（取决于你的 OS，PHP 的码位可以从 30 到 32 不等）。此外，你也可以定义累加权限。

## 累加权限

在上面的第一个例子中，你只能保证用户得到 **owner** 的基本权限。尽管这样可以有效地管理用户的基础操作权限，比如可视，编辑等等。但是有时候我们希望用户得到的权限更加明确清晰。

通过结合几个基本权限，**MaskBuilder** 能够被用于建立位掩码。

```
$builder = new MaskBuilder();
$builder
    ->add('view')
    ->add('edit')
    ->add('delete')
    ->add('undelete')
;
$mask = $builder->get(); // int(29)
```

这个整型位掩码能够被用于保证用户权限添加的成功。

```
$identity = new UserSecurityIdentity('johannes', 'Acme\UserBundle\Entity\User');
$acl->insertObjectAce($identity, $mask);
```

现在用户可以使用可视，编辑，删除以及取消删除对象了。
