# 如何实现一个简单的注册表单

一些表单有额外的字段，其值不需要存储在数据库中。例如，您可能想创建一个带有额外字段的注册表单（像“条款接受”复选框字段）以及嵌入表单，实际上是存储账户信息。

## 简单的用户模型

您有一个简单的 **User** 实体映射到数据库中：

```
// src/Acme/AccountBundle/Entity/User.php
namespace Acme\AccountBundle\Entity;

use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Validator\Constraints as Assert;
use Symfony\Bridge\Doctrine\Validator\Constraints\UniqueEntity;

/**
 * @ORM\Entity
 * @UniqueEntity(fields="email", message="Email already taken")
 */
class User
{
    /**
     * @ORM\Id
     * @ORM\Column(type="integer")
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    protected $id;

    /**
     * @ORM\Column(type="string", length=255)
     * @Assert\NotBlank()
     * @Assert\Email()
     */
    protected $email;

    /**
     * @ORM\Column(type="string", length=255)
     * @Assert\NotBlank()
     * @Assert\Length(max = 4096)
     */
    protected $plainPassword;

    public function getId()
    {
        return $this->id;
    }

    public function getEmail()
    {
        return $this->email;
    }

    public function setEmail($email)
    {
        $this->email = $email;
    }

    public function getPlainPassword()
    {
        return $this->plainPassword;
    }

    public function setPlainPassword($password)
    {
        $this->plainPassword = $password;
    }
}
```

此 **User** 实体包含三个字段，其中两个（**email** 和 **plainPassword**）应该在表单中显示。电子邮件属性在数据库中必须是独立的，这是通过在类的顶部添加此验证执行的。

如果您想在安全系统中整合此用户，您需要实现安全组件的 [UserInterface](http://symfony.com/doc/current/book/security.html#book-security-user-entity)。

## *为什么 4096 密码限制？*

注意 **plainPassword** 字段有一个最长长度有4096个字符。出于安全目的（[CVE-2013-5750](https://symfony.com/blog/cve-2013-5750-security-issue-in-fosuserbundle-login-form)），Symfony 在编码的时候限制了明文密码的长度为4096个字符。

## 为模型创建一个表单

接下来，为 **User** 模型创建表单：

```
// src/Acme/AccountBundle/Form/Type/UserType.php
namespace Acme\AccountBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class UserType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('email', 'email');
        $builder->add('plainPassword', 'repeated', array(
           'first_name'  => 'password',
           'second_name' => 'confirm',
           'type'        => 'password',
        ));
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => 'Acme\AccountBundle\Entity\User'
        ));
    }

    public function getName()
    {
        return 'user';
    }
}
```

只有两种字段：**email** 和 **plainPassword**  （重复以确认输入的密码）。**data_class** 选择告知表单基础数据类的名称（例如，您的 **User** 实体）。

想探索更多关于表单组件的事情，阅读 [Forms](http://symfony.com/doc/current/book/forms.html)。

## 在注册表单中嵌入用户表单

您将为注册页面使用的表单与用于简单修饰 **User**（例如 **UserType**）的表单不一样。注册表单包含进一步的字段像“接受条款”，其值不会被存储在数据库中。

开始先创建一个简单类代表“注册”：

```
// src/Acme/AccountBundle/Form/Model/Registration.php
namespace Acme\AccountBundle\Form\Model;

use Symfony\Component\Validator\Constraints as Assert;

use Acme\AccountBundle\Entity\User;

class Registration
{
    /**
     * @Assert\Type(type="Acme\AccountBundle\Entity\User")
     * @Assert\Valid()
     */
    protected $user;

    /**
     * @Assert\NotBlank()
     * @Assert\True()
     */
    protected $termsAccepted;

    public function setUser(User $user)
    {
        $this->user = $user;
    }

    public function getUser()
    {
        return $this->user;
    }

    public function getTermsAccepted()
    {
        return $this->termsAccepted;
    }

    public function setTermsAccepted($termsAccepted)
    {
        $this->termsAccepted = (bool) $termsAccepted;
    }
}
```

接下来，为 **Registration** 模型创建表单：

```
// src/Acme/AccountBundle/Form/Type/RegistrationType.php
namespace Acme\AccountBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;

class RegistrationType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('user', new UserType());
        $builder->add(
            'terms',
            'checkbox',
            array('property_path' => 'termsAccepted')
        );
        $builder->add('Register', 'submit');
    }

    public function getName()
    {
        return 'registration';
    }
}
```

您不需要为嵌入 **UserType** 表单而使用一种特别的方法。一个表单也是一个字段—所以您可以像其他字段一样添加此字段，预计 **Registration.user** 属性会保存 **User** 类的一个实例。

## 提交表单

下面，您需要一个控制器来处理表单。开始先创建一个简单的控制器来展示注册表单：

```
// src/Acme/AccountBundle/Controller/AccountController.php
namespace Acme\AccountBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;

use Acme\AccountBundle\Form\Type\RegistrationType;
use Acme\AccountBundle\Form\Model\Registration;

class AccountController extends Controller
{
    public function registerAction()
    {
        $registration = new Registration();
        $form = $this->createForm(new RegistrationType(), $registration, array(
            'action' => $this->generateUrl('account_create'),
        ));

        return $this->render(
            'AcmeAccountBundle:Account:register.html.twig',
            array('form' => $form->createView())
        );
    }
}
```

还有它的模板：

```
{# src/Acme/AccountBundle/Resources/views/Account/register.html.twig #}
{{ form(form) }}
```

接下来，创建控制器处理表单提交。进行验证，并将数据保存到数据库中:

```
use Symfony\Component\HttpFoundation\Request;
// ...

public function createAction(Request $request)
{
    $em = $this->getDoctrine()->getManager();

    $form = $this->createForm(new RegistrationType(), new Registration());

    $form->handleRequest($request);

    if ($form->isValid()) {
        $registration = $form->getData();

        $em->persist($registration->getUser());
        $em->flush();

        return $this->redirectToRoute(...);
    }

    return $this->render(
        'AcmeAccountBundle:Account:register.html.twig',
        array('form' => $form->createView())
    );
}
```

## 添加新路由

接下来，更新您的路由。如果您将路由放置于您包的内部（如下所示），不要忘记确保路由文件正在被[导入](http://symfony.com/doc/current/book/routing.html#routing-include-external-resources)。

YAML

```
# src/Acme/AccountBundle/Resources/config/routing.yml
account_register:
    path:     /register
    defaults: { _controller: AcmeAccountBundle:Account:register }

account_create:
    path:     /register/create
    defaults: { _controller: AcmeAccountBundle:Account:create }
```

XML

```
<!-- src/Acme/AccountBundle/Resources/config/routing.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="account_register" path="/register">
        <default key="_controller">AcmeAccountBundle:Account:register</default>
    </route>

    <route id="account_create" path="/register/create">
        <default key="_controller">AcmeAccountBundle:Account:create</default>
    </route>
</routes>
```

PHP

```
// src/Acme/AccountBundle/Resources/config/routing.php
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;

$collection = new RouteCollection();
$collection->add('account_register', new Route('/register', array(
    '_controller' => 'AcmeAccountBundle:Account:register',
)));
$collection->add('account_create', new Route('/register/create', array(
    '_controller' => 'AcmeAccountBundle:Account:create',
)));

return $collection;
```

## 更新您的数据库模式

当然，因为您在此教程中已经添加了一个 **User** 实体，确保您的数据库模型已经正确更新：

```
$ php app/console doctrine:schema:update --force
```

就是这样！您的表单现在验证了，并且允许您保存 **User** 对象到数据库中。在 **Registration** 模型类上额外的 **terms** 复选框在验证过程中使用，但在保存 User 到数据库之后就不是真正地使用了。


