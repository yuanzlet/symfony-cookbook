# 如何在安全，路由，服务和验证中使用表达式

在 Symfony2.4 中，一个强有力的 [ExpressionLanguage](http://symfony.com/doc/current/components/expression_language/introduction.html) 组件添加到 Symfony 中。这允许我们在配置中添加高度自定义的逻辑。

Symfony 框架有以下几种方式利用原装的表达式：

•	[配置服务](http://symfony.com/doc/current/book/service_container.html#book-services-expressions)；

•	[路由匹配条件](http://symfony.com/doc/current/book/routing.html#book-routing-conditions)；

•	[检查安全](http://symfony.com/doc/current/cookbook/expression/expressions.html#book-security-expressions)（解释如下）并且[获取 allow_if 的控件](http://symfony.com/doc/current/cookbook/security/access_control.html#book-security-allow-if)；

•	[验证](http://symfony.com/doc/current/reference/constraints/Expression.html)。

想要知道更多关于如何创建和使用表达式，参见[表达式句法](http://symfony.com/doc/current/components/expression_language/syntax.html)。

## 安全：复杂的访问控件表达式

除了像 **ROLE_ADMIN** 一样的角色，**isGranted** 方法也接受 [Expression](http://api.symfony.com/2.7/Symfony/Component/ExpressionLanguage/Expression.html) 对象：

```
use Symfony\Component\ExpressionLanguage\Expression;
// ...

public function indexAction()
{
    $this->denyAccessUnlessGranted(new Expression(
        '"ROLE_ADMIN" in roles or (user and user.isSuperAdmin())'
    ));

    // ...
}
```

在这个例子中，如果当前用户有 **ROLE_ADMIN**，或者如果当前用户对象是 **isSuperAdmin()**，返回到 **true**，将被授予访问（注意：您的用户对象可能没有 **isSuperAdmin** 方法，该方法专为此例而发明的）。

这使用一个表达式，您可以学习更多的表达式语言句法，参见[表达式句法](http://symfony.com/doc/current/components/expression_language/syntax.html)。

在表达式中，您可能访问到大部分变量：

**user**

   用户对象（或者您未验证的时候字符串 **anon**）

**roles**

   用户所有的一组角色，包括来自[角色等级](http://symfony.com/doc/current/book/security.html#security-role-hierarchy)但不包括 **IS_AUTHENTICATED_*/** 属性（见以下的功能）。

**object**

   对象（若任何）作为第二个参数传递给 **isGranted**。

**token**

   标识对象。

**trust_resolver**

   [AuthenticationTrustResolverInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Authentication/AuthenticationTrustResolverInterface.html) 对象：您可能使用会使用 **is_*** 的功能。

另外，您可以访问表达式里很多的功能：

**is_authenticated**

   返回 **true** 如果用户通过“记住我”或者“完全”验证—例如，返回 true 如果用户“注册”。

**is_anonymous**

   等同于使用 **IS_AUTHENTICATED_ANONYMOUSLY** 的 **isGranted** 功能。

**is_remember_me**

   相似但不等同于 **IS_AUTHENTICATED_REMEMBERED**，见下。

**is_fully_authenticated**

   相似但不等同于 **IS_AUTHENTICATED_FULLY**，见下。

**has_role**

   检查看是否用户有了所给的角色—等同于表达式 **'ROLE_ADMIN' in roles**。

> ### 比起检查 *IS_AUTHENTICATED_REMEMBERED*，*is_remember_me* 是不同的。

> **is_remember_me** 和 **is_authenticated_fully** 功能对于使用 **IS_AUTHENTICATED_REMEMBERED** 和 **IS_AUTHENTICATED_FULLY**  的 **isGranted** 功能是相似的—但是它们**不**一样。以下体现了区别：

```
use Symfony\Component\ExpressionLanguage\Expression;
// ...

$ac = $this->get('security.authorization_checker');
$access1 = $ac->isGranted('IS_AUTHENTICATED_REMEMBERED');

$access2 = $ac->isGranted(new Expression(
    'is_remember_me() or is_fully_authenticated()'
));
```

> 这里，**$access1** 和 **$access2** 将会有相同的值。不像 **IS_AUTHENTICATED_REMEMBERED** 和 **IS_AUTHENTICATED_FULLY** 的行为，如果用户通过 cookie 验证，**is_remember_me** 的功能*只是*返回 true 并且如果用户在这个部分确实注册了（例如，完备的），**is_fully_authenticated** *只是*返回 true。
