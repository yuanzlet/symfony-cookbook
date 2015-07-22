# 如何使用高级的访问控制列表

本章的目的是给出一个更深入的 ACL 系统的观点,并解释其背后的一些设计决策。

## 设计概念

Symfony 的对象实例的安全功能是基于一个访问控制列表的概念。每一个域对象**实例**都有自己的 ACL。ACL 实例有一个详细的列表,访问控制项(ACEs),用于访问决策。Symfony 的 ACL 系统集中在两个主要目标：

- 为您的域对象提供一种方式来有效地检索大量的 ACLs / ACEs，并修改它们；

- 提供一种方式来容易地决定一个人是否被允许在域对象上执行操作。

正如第一点中所暗示的,Symfony 的 ACL 系统的主要功能之一是以高性能的方式检索 ACLs / ACEs。这是非常重要的,因为每个 ACL 可能有一些 ACEs,并以树形方式继承另一个 ACL。因此,ORM 是没有作用的,代替使用 Doctrine 的 DBAL 直接连接的默认实现。

### 对象身份

ACL 系统是完全脱离您的域对象。他们甚至不需要存储在同一个数据库中,或在同一台服务器上。为了实现这种分离,ACL 系统对象通过对象标识对象表示。每次你想为一个域对象检索 ACL,ACL 系统将首先从你的域对象创建一个对象的身份,然后通过这个对象身份 ACL 提供者进行进一步处理。

### 安全身份

这是模拟对象身份标识,但在您的应用程序中代表一个用户或角色。每个角色或用户有自己的安全标识。

## 数据库表结构

默认实现使用五个数据库表如下所示。这些表按行数递增的顺序排在一个典型的应用程序中：

- *acl_security_identities：*此表记录所有持有 ACEs 的安全身份。默认实现附带两个安全身份：[RoleSecurityIdentity](http://api.symfony.com/2.7/Symfony/Component/Security/Acl/Domain/RoleSecurityIdentity.html) 和 [UserSecurityIdentity](http://api.symfony.com/2.7/Symfony/Component/Security/Acl/Domain/UserSecurityIdentity.html)。
- *acl_classes:*这个表类名映射到一个唯一的 ID,可以引用其他表。　　
- *acl_object_identities:*该表中每一行代表一个单独的域对象实例。
- *acl_object_identity_ancestors:*这个表允许所有的祖先 ACL 在一个非常有效的方法下决定。
- *acl_entries:*这个表包含所有 ACEs。这通常是行数最多的表。它可以包含数千万没有显著影响性能的 ACEs。

## 访问控制条目的范围

访问控制条目在他们的应用中可以有不同的适用范围。在 Symfony 中,基本上有两个不同的范围：

- 类范围:这些条目适用于所有从属于相同类的对象。

- 对象范围:这个使用范围只用在前一章,它只适用于一个特定的对象。

有时,你会发现只需要为对象的一个特定字段申请一个 ACE。假设您想要一个只对管理员而不是您的客户服务可见的 ID。为了解决这个普遍的问题,增加了两个 sub-scopes：

- Class-Field-Scope：这些条目适用于所有从属于相同类的对象,但只有特定字段的对象。

- Object-Field-Scope：这些条目适用于一个特定的对象,并且只适用于那个对象的特定字段。　　

## 预先授权决策

预先授权决策,即做出决定之前任何安全方法被调用(或安全措施),已证实 AccessDecisionManager 服务是被使用的。AccessDecisionManager 也被用于实现基于角色的授权决策。就像角色,ACL 系统添加了一些新的属性,可以用来检查不同的权限。

### 内置权限映射

<table  border="1">
<colgroup>
<col width="11%">
<col width="9%">
<col width="9%">
<col width="8%">
<col width="22%">
<col width="41%">
</colgroup>
<thead valign="bottom">
<tr><th><strong>属性</th>
<th><strong>含义</th>
<th><strong>整数位掩码</th>
</tr>
</thead>
<tbody valign="top">
<tr><td>VIEW</td>
<td>是否有人可以查看域对象。</td>
<td>VIEW, EDIT, OPERATOR, MASTER, or OWNER</td>
</tr>
<tr><td>EDIT</td>
<td>一个人是否可以更改域对象。</td>
<td>EDIT, OPERATOR, MASTER, or OWNER</td>
</tr>
<tr><td>CREATE</td>
<td>一个人是否被允许创建域对象。</td>
<td>CREATE, OPERATOR, MASTER, or OWNER</td>
</tr>
<tr><td>DELETE</td>
<td>一个人是否被允许删除域对象。</td>
<td>DELETE, OPERATOR, MASTER, or OWNER</td>
</tr>
<tr><td>UNDELETE</td>
<td>删除的人是否可以恢复以前删除的域对象。</td>
<td>UNDELETE, OPERATOR, MASTER, or OWNER</td>
</tr>
<tr><td>OPERATOR</td>
<td>是否有人允许执行所有上述操作。</td>
<td>OPERATOR, MASTER, or OWNER</td>
</tr>
<tr><td>MASTER</td>
<td>是否有人允许执行所有上述操作，并且允许任何上述权限授予其他人。</td>
<td>MASTER, or OWNER</td>
</tr>
<tr><td>OWNER</td>
<td>是否有人拥有域对象。OWNER 可以执行上述任何操作并授予 MASTER 和 OWNER 权限。</td>
<td>OWNER</td>
</tr>
</tbody>
</table>

### 权限属性 vs. 权限位掩码 　　

AccessDecisionManager 属性的使用就像角色一样。通常,这些属性事实上代表一个聚合的整数位掩码。使用整数位掩码另一方面,ACL 系统内部在数据库中有效存储用户的权限,并使用极快的位掩码操作执行访问检查。

### 可扩展性

上述许可映射绝不是静态的，并且理论上可以完全被取代。然而,它应该囊括大多数您遇到的问题,以及与其他包的互操作性。鼓励您坚持它们原本被设想的意义。

## 后授权决策

后授权决策在一个安全的方法被调用后做出,并且通常涉及被这样的方法返回的域对象。在调用完成后，提供者也允许修改,或在返回之前过滤域对象之前返回。

由于当前的 PHP 语言的局限性,没有 post-authorization 功能构建为核心的安全组件。然而,有一个实验 [JMSSecurityExtraBundle](https://github.com/schmittjoh/JMSSecurityExtraBundle) 增加这些功能。如果您想了解更多关于这是如何完成的的信息，请看它的相关文档。　

## 达到授权决策的过程

ACL 类提供了两个方法来确定是否一个安全标识需要位掩码,**isGranted** 和 **isFieldGranted**。当 ACL 通过这些方法之一收到授权请求,它代表这个请求 [PermissionGrantingStrategy](http://api.symfony.com/2.7/Symfony/Component/Security/Acl/Domain/PermissionGrantingStrategy.html) 的实现。这允许您替换访问决策的方式而不修改 ACL 类本身。

**PermissionGrantingStrategy** 首先检查你所有的 object-scope ACEs。如果没有适用的,class-scope ACEs 将被检查。如果没有适用的,那么这个过程会重复地对父 ACL 的 ACEs 进行。如果没有父亲 ACL 存在,将会抛出一个异常。
