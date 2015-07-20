# 如何利用表单事件动态修改表单

时常地，表单不能静态地被创建。在这一节，你将学习基于三种常见的用例如何自定义你的表单：  

1. [基于基础数据自定义你的表单](http://symfony.com/doc/current/cookbook/form/dynamic_form_modification.html#cookbook-form-events-underlying-data)  
   例子：你有一个“产品”表单并且你需要修正、添加、移除一个字段基于基础的被编辑的产品数据。  
2. [如何基于用户数据动态生成表单](http://symfony.com/doc/current/cookbook/form/dynamic_form_modification.html#cookbook-form-events-user-data)  
   例子：你创建了一个 “Friend Message” 的表单并且需要建立一个包含和现有的授权的用户友好的唯一用户的下拉菜单。  
3. [动态生成提交的表单](http://symfony.com/doc/current/cookbook/form/dynamic_form_modification.html#cookbook-form-events-submitted-data)  
   例子：在一个注册表单中，你有一个“镇”字段并且还有一个“州”字段，这个字段应该是动态的取决于“镇”字段的值。  

如果你想要学习更多表单事件之后的基本知识，你可以看看[表单事件](http://symfony.com/doc/current/components/form/form_events.html)的文档。  

## 基于基础数据自定义你的表单

在跳到动态表格产生之前，想象一下空的表单类什么样：  

```
// src/AppBundle/Form/Type/ProductType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class ProductType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('name');
        $builder->add('price');
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => 'AppBundle\Entity\Product'
        ));
    }

    public function getName()
    {
        return 'product';
    }
}
```  

> 如果这段特定的代码你已经不熟悉了，你可能需要在进行下一步之前复习一下[表单章节](http://symfony.com/doc/current/book/forms.html)。  

假设这样一种情况：这个表单使用了一个虚构的“产品”类这个类只有两个属性(“名称”和“价格”）。由这个类产生的表单将会看起来都一样不管是否有一个新的产品被创建或者已经存在的产品被编辑（例如从数据库中取出产品）。  

现在假设，一旦对象被创建的话你就不希望用户改变**名称**的值了。为了完成这个，你可以使用 Symfony 的 [EventDispatcher component](http://symfony.com/doc/current/components/event_dispatcher/introduction.html) 系统来分析数据并且基于产品的对象信息修正表单。在这一节，你将要学习如何在这个层次向你的表单添加灵活性。  

### 向表单类添加事件监听器

那么，代替直接添加**名称**控件，创建那个特定的字段的任务就委托给事件监听器了：  

```
// src/AppBundle/Form/Type/ProductType.php
namespace AppBundle\Form\Type;

// ...
use Symfony\Component\Form\FormEvent;
use Symfony\Component\Form\FormEvents;

class ProductType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('price');

        $builder->addEventListener(FormEvents::PRE_SET_DATA, function (FormEvent $event) {
            // ... adding the name field if needed
        });
    }

    // ...
}
```

目标就是只创建**名称**字段如果基础的**产品**对象是新的（例如没有被放到数据库中）。基于这一点，事件监听器可能如下所示：  

```
// ...
public function buildForm(FormBuilderInterface $builder, array $options)
{
    // ...
    $builder->addEventListener(FormEvents::PRE_SET_DATA, function (FormEvent $event) {
        $product = $event->getData();
        $form = $event->getForm();

        // check if the Product object is "new"
        // If no data is passed to the form, the data is "null".
        // This should be considered a new "Product"
        if (!$product || null === $product->getId()) {
            $form->add('name', 'text');
        }
    });
}
```  

> **FormEvents::PRE_SET_DATA** 行其实是分解了 **form.pre_set_data** 字符串。[FormEvents](http://api.symfony.com/2.7/Symfony/Component/Form/FormEvents.html) 是具有组织功能的。它是一个中心地带，在这里你可以找到所有的不同的表单事件。你可以通过 [FormEvents](http://api.symfony.com/2.7/Symfony/Component/Form/FormEvents.html) 类来看完整的表单事件列表。  

### 向表单类添加事件预订管理

为了更好的重复利用或者如果你的事件监听器里有复杂的逻辑，你也可以通过向[事件预定管理](http://symfony.com/doc/current/components/event_dispatcher/introduction.html#event-dispatcher-using-event-subscribers)中添加**名称**字段来转移逻辑：  

```
// src/AppBundle/Form/Type/ProductType.php
namespace AppBundle\Form\Type;

// ...
use AppBundle\Form\EventListener\AddNameFieldSubscriber;

class ProductType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('price');

        $builder->addEventSubscriber(new AddNameFieldSubscriber());
    }

    // ...
}
```  

现在创建**名称**的逻辑存在于它自己的预定类之中了：  

```
// src/AppBundle/Form/EventListener/AddNameFieldSubscriber.php
namespace AppBundle\Form\EventListener;

use Symfony\Component\Form\FormEvent;
use Symfony\Component\Form\FormEvents;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;

class AddNameFieldSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents()
    {
        // Tells the dispatcher that you want to listen on the form.pre_set_data
        // event and that the preSetData method should be called.
        return array(FormEvents::PRE_SET_DATA => 'preSetData');
    }

    public function preSetData(FormEvent $event)
    {
        $product = $event->getData();
        $form = $event->getForm();

        if (!$product || null === $product->getId()) {
            $form->add('name', 'text');
        }
    }
}
```  

## 如何基于用户数据动态创建表单

有些时候你希望你的表单不只是基于其它表单数据动态创建而是基于其它的数据——例如当前的用户的一些数据。假设你有一个社交网站，网站中的人们只能和被标记成朋友的人进行聊天。在这种情况下，和谁聊天的“选择列表”应当只包括目前用户的朋友的用户名。  

### 创建表单样式类型

使用了事件监听器，你的表单可能如下所示：  

```
// src/AppBundle/Form/Type/FriendMessageFormType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Form\FormEvents;
use Symfony\Component\Form\FormEvent;
use Symfony\Component\Security\Core\Authentication\Token\Storage\TokenStorageInterface;

class FriendMessageFormType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('subject', 'text')
            ->add('body', 'textarea')
        ;
        $builder->addEventListener(FormEvents::PRE_SET_DATA, function (FormEvent $event) {
            // ... add a choice list of friends of the current application user
        });
    }

    public function getName()
    {
        return 'friend_message';
    }
}
```  

现在的问题就是获取当前用户的用户名，同时创建一个只包含用户朋友的选择字段。  

幸运的是向表单中注入服务这个十分简单。这个可以在构造器中完成：  

```
private $tokenStorage;

public function __construct(TokenStorageInterface $tokenStorage)
{
    $this->tokenStorage = $tokenStorage;
}
```  

> 你可能会奇怪，既然你已经可以访问用户（通过存储），为什么不直接在 **buildForm** 使用并且忽略事件监听器呢？这是因为在 **buildForm** 方法中这样做的话就会导致整个表单类型被修正而仅仅是一个表单实例。这个可能不是一个常见问题，但是技术层面来讲的话一个单一的表单类型可以使用单一的请求来创建很多表单或者字段。  

### 自定义表单类型

既然你已经有了基础你就可以利用 **TokenStorageInterface** 并且向监听器添加逻辑了：  

```
// src/AppBundle/FormType/FriendMessageFormType.php

use Symfony\Component\Security\Core\Authentication\Token\Storage\TokenStorageInterface;
use Doctrine\ORM\EntityRepository;
// ...

class FriendMessageFormType extends AbstractType
{
    private $tokenStorage;

    public function __construct(TokenStorageInterface $tokenStorage)
    {
        $this->tokenStorage = $tokenStorage;
    }

    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('subject', 'text')
            ->add('body', 'textarea')
        ;

        // grab the user, do a quick sanity check that one exists
        $user = $this->tokenStorage->getToken()->getUser();
        if (!$user) {
            throw new \LogicException(
                'The FriendMessageFormType cannot be used without an authenticated user!'
            );
        }

        $builder->addEventListener(
            FormEvents::PRE_SET_DATA,
            function (FormEvent $event) use ($user) {
                $form = $event->getForm();

                $formOptions = array(
                    'class' => 'AppBundle\Entity\User',
                    'property' => 'fullName',
                    'query_builder' => function (EntityRepository $er) use ($user) {
                        // build a custom query
                        // return $er->createQueryBuilder('u')->addOrderBy('fullName', 'DESC');

                        // or call a method on your repository that returns the query builder
                        // the $er is an instance of your UserRepository
                        // return $er->createOrderByFullNameQueryBuilder();
                    },
                );

                // create the field, this is similar the $builder->add()
                // field name, field type, data, options
                $form->add('friend', 'entity', $formOptions);
            }
        );
    }

    // ...
}
```

> [TokenStorageInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Core/Authentication/Token/Storage/TokenStorageInterface.html) 是在 Symfony 2.6 中被引入的。以前的版本，你需要使用 [SecurityContextInterface](http://api.symfony.com/2.7/Symfony/Component/Security/Core/SecurityContextInterface.html) 的 **getToken()** 方法。  

> **multiple** 和 **expanded** 表单选项都是默认设置成 false 这是因为邻近字段是**实体**。  

### 使用表单

现在我们的表单准备使用了，这里还有两种可能的方式在控制器中使用它：  

1. 手动创建并且记住将 token storage 传递给它；  

或者  

2. 将它定义为服务。  

#### a) 手动创建表单

这个非常简单，并且这可能是更好的方法，除非你正在很多地方使用你的新的表单类型或者将其放置在其它表单中：  

```
class FriendMessageController extends Controller
{
    public function newAction(Request $request)
    {
        $tokenStorage = $this->container->get('security.token_storage');
        $form = $this->createForm(
            new FriendMessageFormType($tokenStorage)
        );

        // ...
    }
}
```  

#### b)将表单定义为服务

将你的表单定义为服务，仅仅创建一个正常的服务然后添加 [form.type](http://symfony.com/doc/current/reference/dic_tags.html#dic-tags-form-type) 标签。  

YAML:

```YAML
# app/config/config.yml
services:
    app.form.friend_message:
        class: AppBundle\Form\Type\FriendMessageFormType
        arguments: ["@security.token_storage"]
        tags:
            - { name: form.type, alias: friend_message }
```

XML:

```XML
<!-- app/config/config.xml -->
<services>
    <service id="app.form.friend_message" class="AppBundle\Form\Type\FriendMessageFormType">
        <argument type="service" id="security.context" />
        <tag name="form.type" alias="friend_message" />
    </service>
</services>
```

PHP:

```PHP
// app/config/config.php
$definition = new Definition('AppBundle\Form\Type\FriendMessageFormType');
$definition->addTag('form.type', array('alias' => 'friend_message'));
$container->setDefinition(
    'app.form.friend_message',
    $definition,
    array('security.token_storage')
);
```  

如果你想要从控制器中或者其它的有权访问表单工厂的服务创建表单，那么你可以使用：  

```
use Symfony\Component\DependencyInjection\ContainerAware;

class FriendMessageController extends ContainerAware
{
    public function newAction(Request $request)
    {
        $form = $this->get('form.factory')->create('friend_message');

        // ...
    }
}
```  

如果你扩展 **Symfony\Bundle\FrameworkBundle\Controller\Controller** 类，你可以简单地调用：  

```
$form = $this->createForm('friend_message');
```  

你也可以简单的将表单类型嵌入到其它表单：  

```
// inside some other "form type" class
public function buildForm(FormBuilderInterface $builder, array $options)
{
    $builder->add('message', 'friend_message');
}
```  

## 动态创建提交表单

另外一种可能出现的情况就是你想要根据用户提交的数据特定的自定义表单。举例来说，假设你有一个收集运动的注册表单。一些时间将会允许你指定你的字段的喜欢的位置。这将会是一个**选择**字段的例子。然而可能的选择将会依赖于运动。足球就会有前锋，后卫，守门员等等……棒球就会有投手但是不会有守门员。你需要正确的选项来保证验证通过。  

这个将作为一个实体字段传递到表单。所以我们可以向下面这样访问每一项运动：  

```
// src/AppBundle/Form/Type/SportMeetupType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Form\FormEvent;
use Symfony\Component\Form\FormEvents;
// ...

class SportMeetupType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('sport', 'entity', array(
                'class'       => 'AppBundle:Sport',
                'placeholder' => '',
            ))
        ;

        $builder->addEventListener(
            FormEvents::PRE_SET_DATA,
            function (FormEvent $event) {
                $form = $event->getForm();

                // this would be your entity, i.e. SportMeetup
                $data = $event->getData();

                $sport = $data->getSport();
                $positions = null === $sport ? array() : $sport->getAvailablePositions();

                $form->add('position', 'entity', array(
                    'class'       => 'AppBundle:Position',
                    'placeholder' => '',
                    'choices'     => $positions,
                ));
            }
        );
    }

    // ...
}
```  

> 为了支持 **empty_value**，**placeholder** 选项是在 Symfony 2.6 中引进的，这个在 2.6 之前版本也可以用。  

当你第一次创建表单来展示用户时，那么这个例子会很好的帮助你。  

然而，当你处理表单提交的时候事情就变得复杂了。这是因为 **PRE_SET_DATA** 事件告诉我们你开始的数据（例如一个空的 **SportMeetup** 对象）而*不是*提交的数据。  

在表单上，我们经常会听到下列事件：  

- **PRE_SET_DATA**
- **POST_SET_DATA**
- **PRE_SUBMIT**
- **SUBMIT**
- **POST_SUBMIT**

> **PRE_SUBMIT**, **SUBMIT** 和 **POST_SUBMIT** 事件是在 Symfony 2.3 中引进的。在之前，它们叫做 **PRE_BIND**, **BIND** 和 **POST_BIND**。

关键就是将 **POST_SUBMIT** 监听器添加到你依赖的字段中。如果你将一个 **POST_SUBMIT** 监听器添加到子表单中（例如**运动**），并且向父表单添加一个新的子表单，表单组件就会自动侦测出新的字段并且将它映射到提交的客户的数据。  

表单的形式将会如下所示：  

```
// src/AppBundle/Form/Type/SportMeetupType.php
namespace AppBundle\Form\Type;

// ...
use Symfony\Component\Form\FormInterface;
use AppBundle\Entity\Sport;

class SportMeetupType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('sport', 'entity', array(
                'class'       => 'AppBundle:Sport',
                'placeholder' => '',
            ));
        ;

        $formModifier = function (FormInterface $form, Sport $sport = null) {
            $positions = null === $sport ? array() : $sport->getAvailablePositions();

            $form->add('position', 'entity', array(
                'class'       => 'AppBundle:Position',
                'placeholder' => '',
                'choices'     => $positions,
            ));
        };

        $builder->addEventListener(
            FormEvents::PRE_SET_DATA,
            function (FormEvent $event) use ($formModifier) {
                // this would be your entity, i.e. SportMeetup
                $data = $event->getData();

                $formModifier($event->getForm(), $data->getSport());
            }
        );

        $builder->get('sport')->addEventListener(
            FormEvents::POST_SUBMIT,
            function (FormEvent $event) use ($formModifier) {
                // It's important here to fetch $event->getForm()->getData(), as
                // $event->getData() will get you the client data (that is, the ID)
                $sport = $event->getForm()->getData();

                // since we've added the listener to the child, we'll have to pass on
                // the parent to the callback functions!
                $formModifier($event->getForm()->getParent(), $sport);
            }
        );
    }

    // ...
}
```  

你可以看到你需要监听这两个事件并且需要有不同的回调，只是因为在两个不同的情形，你可以应用的数据在不同的事件中。如果不是那样，监听器将会一直在给定的表单执行相同的任务。  

还差一件事就是在选定运动后的客户端升级。这将会通过向你的应用程序进行 AJAX 回调完成。假设你拥有运动集合创建控制器：  

```
// src/AppBundle/Controller/MeetupController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;
use AppBundle\Entity\SportMeetup;
use AppBundle\Form\Type\SportMeetupType;
// ...

class MeetupController extends Controller
{
    public function createAction(Request $request)
    {
        $meetup = new SportMeetup();
        $form = $this->createForm(new SportMeetupType(), $meetup);
        $form->handleRequest($request);
        if ($form->isValid()) {
            // ... save the meetup, redirect etc.
        }

        return $this->render(
            'AppBundle:Meetup:create.html.twig',
            array('form' => $form->createView())
        );
    }

    // ...
}
```  

根据当前在**运动**字段的选择，关联的模板使用了一些 JavaScript 更新**位置**表单字段：  

Twig:

```Twig
{# app/Resources/views/Meetup/create.html.twig #}
{{ form_start(form) }}
    {{ form_row(form.sport) }}    {# <select id="meetup_sport" ... #}
    {{ form_row(form.position) }} {# <select id="meetup_position" ... #}
    {# ... #}
{{ form_end(form) }}

<script>
var $sport = $('#meetup_sport');
// When sport gets selected ...
$sport.change(function() {
  // ... retrieve the corresponding form.
  var $form = $(this).closest('form');
  // Simulate form data, but only include the selected sport value.
  var data = {};
  data[$sport.attr('name')] = $sport.val();
  // Submit data via AJAX to the form's action path.
  $.ajax({
    url : $form.attr('action'),
    type: $form.attr('method'),
    data : data,
    success: function(html) {
      // Replace current position field ...
      $('#meetup_position').replaceWith(
        // ... with the returned one from the AJAX response.
        $(html).find('#meetup_position')
      );
      // Position field now displays the appropriate positions.
    }
  });
});
</script>
```

PHP:

```PHP
<!-- app/Resources/views/Meetup/create.html.php -->
<?php echo $view['form']->start($form) ?>
    <?php echo $view['form']->row($form['sport']) ?>    <!-- <select id="meetup_sport" ... -->
    <?php echo $view['form']->row($form['position']) ?> <!-- <select id="meetup_position" ... -->
    <!-- ... -->
<?php echo $view['form']->end($form) ?>

<script>
var $sport = $('#meetup_sport');
// When sport gets selected ...
$sport.change(function() {
  // ... retrieve the corresponding form.
  var $form = $(this).closest('form');
  // Simulate form data, but only include the selected sport value.
  var data = {};
  data[$sport.attr('name')] = $sport.val();
  // Submit data via AJAX to the form's action path.
  $.ajax({
    url : $form.attr('action'),
    type: $form.attr('method'),
    data : data,
    success: function(html) {
      // Replace current position field ...
      $('#meetup_position').replaceWith(
        // ... with the returned one from the AJAX response.
        $(html).find('#meetup_position')
      );
      // Position field now displays the appropriate positions.
    }
  });
});
</script>
```  

向确切的更新过的**位置**字段提交整个表单的主要的好处就是不需要附加的服务端代码；所有的上述代码产生的提交的表单都能再利用。  

## 禁止表单验证

使用 **POST_SUBMIT** 事件来禁止表单验证并且阻止 [ValidationListener](http://api.symfony.com/2.7/Symfony/Component/Form/Extension/Validator/EventListener/ValidationListener.html) 被调用。  

需要做这个的原因就是即使你设置 **validation_groups** 为 **false** 也会依然有完整性的检查执行。举例来说，一个上传的文件将会被检查是否太大,表单将会检查是否有不存在的字段被提交。使用监听器禁用所有这些：  

```
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Form\FormEvents;
use Symfony\Component\Form\FormEvent;

public function buildForm(FormBuilderInterface $builder, array $options)
{
    $builder->addEventListener(FormEvents::POST_SUBMIT, function (FormEvent $event) {
        $event->stopPropagation();
    }, 900); // Always set a higher priority than ValidationListener

    // ...
}
```

> 通过这样做。你可以故意的禁用一些东西不仅仅是表单验证，因为 **POST_SUBMIT** 事件可能还有其它的监听器。