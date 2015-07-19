# 如何使用数据转换

数据转换是用于将字段数据转换成可以在表单中展示的格式（并且可以重新提交）。他们已经内部使用了很多字段类型。举例来说，[数据字段类型](http://symfony.com/doc/current/reference/forms/types/date.html)可以被渲染成 **yyyy-MM-dd** 格式的文本框。内部的，数据转换将 **DateTime** 开始的字段的值转换成 **yyyy-MM-dd** 字符串来渲染表格，并且返回到 **DateTime** 对象提交。  

>当表单字段拥有 **inherit_data** 选项设置，数转换将不会被应用到那个字段。  

## 简单的例子：在用户输入上消除 HTML ##

假设你拥有一个 **textarea** 类型描述标签的 Task 表单：  

```
// src/AppBundle/Form/TaskType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

// ...
class TaskType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('description', 'textarea');
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => 'AppBundle\Entity\Task',
        ));
    }

    // ...
}
```

但是这里有两个复杂点：  

1. 你的用户允许使用*一些* HTML 标签，但是不是其他的：在表单提交之后你需要一种调用 [striptags](http://php.net/manual/en/function.striptags.php) 的方法；
2. 为了友好性，在渲染表单之前你想要将 **\<br/\>** 标签转换成换行符（**\n**）这样文本就更容易编辑了。

这是一个将定制的数据转换附到 **description** 字段的*最好*时机。最简单的方法就是使用 [CallbackTransformer](http://api.symfony.com/2.7/Symfony/Component/Form/CallbackTransformer.html) 类：  

```
// src/AppBundle/Form/TaskType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\CallbackTransformer;
use Symfony\Component\Form\FormBuilderInterface;
// ...

class TaskType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('description', 'textarea');

        $builder->get('description')
            ->addModelTransformer(new CallbackTransformer(
                // transform <br/> to \n so the textarea reads easier
                function ($originalDescription) {
                    return preg_replace('#<br\s*/?>#i', "\n", $originalDescription);
                },
                function ($submittedDescription) {
                    // remove most HTML tags (but not br,p)
                    $cleaned = strip_tags($submittedDescription, '<br><br/><p>');

                    // transform any \n to real <br/>
                    return str_replace("\n", '<br/>', $cleaned);
                }
            ))
        ;
    }

    // ...
}
```

**CallbackTransformer** 类使用了两个召回功能作为变元。第一个将原始值转换成为能够在渲染字段时使用的格式。第二个做了相反的事情：它将提交的值转换回你在代码中将要用的格式。  

>**addModelTransformer()** 方法接受任何实施 [DataTransformerInterface](http://api.symfony.com/2.7/Symfony/Component/Form/DataTransformerInterface.html) 的对象这样你就可以创建你自己的类，而不是将所有的逻辑都放入表单（详见下一节）。

你也可以添加转换器，你可以通过稍微改变格式的方法添加文件：  

```
$builder->add(
    $builder->create('description', 'textarea')
        ->addModelTransformer(...)
);
```
## 更难的例子：将问题数字转化为问题实体 ##

比如说你的 Task 实体到 Issue 实体有多对一的关系（例如每一个 Task 都有一个可选的外部关键字对应与之相关的 Issue）。添加包含所有问题的列表框可能最终会变得*很*长并且需要很长时间加载出来。作为替代，你可以在用户可以简单地输入问题数字的地方决定你想要添加一个列表框。  

由像往常一样设置文本字段开始：  

```
// src/AppBundle/Form/TaskType.php
namespace AppBundle\Form\Type;

// ...
class TaskType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('description', 'textarea')
            ->add('issue', 'text')
        ;
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => 'AppBundle\Entity\Task'
        ));
    }

    // ...
}
```

好的开始！但是如果你在这里停止并且提交表单，Task 的 **issue** 属性就会是一个字符串（例如 “55”）。你如在提交时将这个转换成为 **Issue** 实体？  

### 创建转换器 ###

你可以像之前那样使用 **CallbackTransformer**。但是由于这个有一丢丢复杂，创建一个新的转换类将会使得 **TaskType** 表单类更简单。  

创建一个 **IssueToNumberTransformer** 类：它将会负责转化问题数字和 **Issue** 实体：  

```
// src/AppBundle/Form/DataTransformer/IssueToNumberTransformer.php
namespace AppBundle\Form\DataTransformer;

use AppBundle\Entity\Issue;
use Doctrine\Common\Persistence\EntityManager;
use Symfony\Component\Form\DataTransformerInterface;
use Symfony\Component\Form\Exception\TransformationFailedException;

class IssueToNumberTransformer implements DataTransformerInterface
{
    private $entityManager;

    public function __construct(EntityManager $entityManager)
    {
        $this->entityManager = $entityManager;
    }

    /**
     * Transforms an object (issue) to a string (number).
     *
     * @param  Issue|null $issue
     * @return string
     */
    public function transform($issue)
    {
        if (null === $issue) {
            return '';
        }

        return $issue->getId();
    }

    /**
     * Transforms a string (number) to an object (issue).
     *
     * @param  string $issueNumber
     * @return Issue|null
     * @throws TransformationFailedException if object (issue) is not found.
     */
    public function reverseTransform($issueNumber)
    {
        // no issue number? It's optional, so that's ok
        if (!$issueNumber) {
            return;
        }

        $issue = $this->entityManager
            ->getRepository('AppBundle:Issue')
            // query for the issue with this id
            ->find($issueNumber)
        ;

        if (null === $issue) {
            // causes a validation error
            // this message is not shown to the user
            // see the invalid_message option
            throw new TransformationFailedException(sprintf(
                'An issue with number "%s" does not exist!',
                $issueNumber
            ));
        }

        return $issue;
    }
}
```

就好像第一个例子，转换器有两个方向。**transform()** 方法负责将你代码中的数据转换成可以在你的表单中渲染的格式（例如 **Issue** 对象到它的 **id** 一个字符串）。**reverseTransform()** 方法负责相反的工作：它将提交的值转回你想要的格式（例如将 **id** 转换成为 **Issue** 对象）。  

为了引起校验错误，使用 [TransformationFailedException](http://api.symfony.com/2.7/Symfony/Component/Form/Exception/TransformationFailedException.html)。但是你向这个例外传递的信息不回向用户展示。你可以使用 **invalid_message** 选项来设置消息（详见下面）。  

>当 **null** 被传递到 **transform()** 方法时。你的转换器将会返回和它正在转化的类型相等的值（例如一个空的字符串，整型的 0 或者浮点型的 0.0）。  

### 使用转换器 ###

接下来，你需要从 **TaskType** 内部将 **IssueToNumberTransformer** 类实例化并且将其添加到 **issue** 字段。但是为了完成这个，你将会需要实体管理的实例（由于 **IssueToNumberTransformer** 需要这个）。  

没问题！仅仅添加 **__construct()** 功能到 **TaskType** 中并且使得这个被传入。然后，你就能轻而易举地创建以及添加转换器了：  

```
// src/AppBundle/Form/TaskType.php
namespace AppBundle\Form\Type;

use AppBundle\Form\DataTransformer\IssueToNumberTransformer;
use Doctrine\Common\Persistence\EntityManager;

// ...
class TaskType extends AbstractType
{
    private $entityManager;

    public function __construct(EntityManager $entityManager)
    {
        $this->entityManager = $entityManager;
    }

    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('description', 'textarea')
            ->add('issue', 'text', array(
                // validation message if the data transformer fails
                'invalid_message' => 'That is not a valid issue number',
            ));

        // ...

        $builder->get('issue')
            ->addModelTransformer(new IssueToNumberTransformer($this->entityManager));
    }

    // ...
}
```

现在，当你创建你的 **TaskType** 时，你需要在实体管理器中传递：  

```
// e.g. in a controller somewhere
$entityManager = $this->getDoctrine()->getManager();
$form = $this->createForm(new TaskType($entityManager), $task);

// ...
```  

>为了使得这一步更加简单（尤其如果 **TaskType** 嵌入在其他的表单类型类中），你可以选择[将你的表单类型注册成为服务](http://symfony.com/doc/current/book/forms.html#form-as-services)。  

棒棒的，你已经完成了！你的用户将可以很容易地在文本字段中输入问题数字并且这将会转化成问题对象。这就意味着，在成功的提交之后，表单组件将会向 **Task::setIssue()** 传递一个真正的 **Issue** 对象而不是问题数字。  

如果问题没有被发现的话，那个字段的表单错误将会产生同时这个错误消息可以被 **invalid_message** 字段选项控制。  

>在添加你的转换器的时候需要注意。举例来说，下列代码就是**错误**的，由于转换器将会被用于真个表单而不是仅仅这个字段：  

>```
>// THIS IS WRONG - TRANSFORMER WILL BE APPLIED TO THE ENTIRE FORM
// see above example for correct code
$builder->add('issue', 'text')
    ->addModelTransformer($transformer);
>```

## 创建一个可以重复使用的 issue_selector 字段 ##

在上面的例子中，你对正常的 **text** 字段应用了转换器。但是如果做很多这样的转换，最好[创建一个定制的字段类型](http://symfony.com/doc/current/cookbook/form/create_custom_field_type.html)那样就可以自动完成这些了。  

首先，创建定制字段类型类：  

```
// src/AppBundle/Form/IssueSelectorType.php
namespace AppBundle\Form;

use AppBundle\Form\DataTransformer\IssueToNumberTransformer;
use Doctrine\ORM\EntityManager;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class IssueSelectorType extends AbstractType
{
    private $entityManager;

    public function __construct(EntityManager $entityManager)
    {
        $this->entityManager = $entityManager;
    }

    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $transformer = new IssueToNumberTransformer($this->entityManager);
        $builder->addModelTransformer($transformer);
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'invalid_message' => 'The selected issue does not exist',
        ));
    }

    public function getParent()
    {
        return 'text';
    }

    public function getName()
    {
        return 'issue_selector';
    }
}
```  

很好！这个像文本字段（**getParent()**）一样的运行和渲染，但是将会自动具有数据转换*并且* **invalid_message** 选项将会有一个很好的默认值。  

接下来，将你的类型注册为服务并且给它加上 **form.type** 的标签这样他就可以被认为是定制的字段类型了：  

```YAML
# app/config/services.yml
services:
    app.type.issue_selector:
        class: AppBundle\Form\IssueSelectorType
        arguments: ["@doctrine.orm.entity_manager"]
        tags:
            - { name: form.type, alias: issue_selector }
```

```XML
<!-- app/config/services.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">

    <services>
        <service id="app.type.issue_selector"
            class="AppBundle\Form\IssueSelectorType">
            <argument type="service" id="doctrine.orm.entity_manager"/>
            <tag name="form.type" alias="issue_selector" />
        </service>
    </services>
</container>
```

```PHP
// app/config/services.php
use Symfony\Component\DependencyInjection\Definition;
use Symfony\Component\DependencyInjection\Reference;
// ...

$container
    ->setDefinition('app.type.issue_selector', new Definition(
            'AppBundle\Form\IssueSelectorType'
        ),
        array(
            new Reference('doctrine.orm.entity_manager'),
        )
    )
    ->addTag('form.type', array(
        'alias' => 'issue_selector',
    ))
;
```

现在，不论何时你想要使用你的特殊的 **issue_selector** 字段类型都是十分容易的：  

```
// src/AppBundle/Form/TaskType.php
namespace AppBundle\Form\Type;

use AppBundle\Form\DataTransformer\IssueToNumberTransformer;
// ...

class TaskType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('description', 'textarea')
            ->add('issue', 'issue_selector')
        ;
    }

    // ...
}
```

## 关于模型和视图转换 ##

在上面的例子中，转换器曾经作为一种“模型”转换器。实际上，有两种类型的转化器同时又有三种不同类型的基础数据。  

![](http://symfony.com/doc/current/_images/DataTransformersTypes.png)  

在任何表单中，这三种不同的类型数据是：  

1. **模型数据**——这是你的应用程序中使用的数据格式（例如一个 **Issue** 对象）。如果你调用 **Form::getData()** 或者 **Form::setData()**，你就会处理模型数据。
2. **普通数据**——这是你的数据的普通版本并且这个和你的“模型”数据一样常见（尽管不是在我们的例子中）。它经常不会被直接应用。
3. **视图数据**——这是表单字段自动填充的数据格式。用户也很有可能提交这种格式的数据。当你调用 **Form::submit($data)** 时，**$data** 就是“视图”格式的数据。  

这两种不同类型的转换器帮助来回转换这些类型的数据：  

**模型转换器：**  
- **转换**：“模型数据”=>“普通数据”
- **反转换**：“普通数据”=>“模型数据”  

**视图转换器：**
- **转换**：“普通数据”=>“视图数据”
- **反转换**：“视图数据”=>“普通数据”  

你需要那种转换器取决于你的实际情况。  

为了使用视图转换器，你需要调用 **addViewTransformer**。  

### 那么为什么使用模型转换器？ ###

在这个例子中，字段是 **文本** 字段，同时文本字段一直被认为是一种简单，纯量的格式在“普通”和“视图”格式中。由于这个原因最合适的转换器就是“模型”转换器（这个转换器来回转换*普通*格式）——字符串的 issue 数字——到*模型*格式——Issue 对象）。  

转换器的不同之处在于副标题以及你需要一直想着字段的“普通”数据会是什么样子。举例来说，**文本**字段的普通数据就是一个字符串，但是对于**日期**字段就是 **DateTime** 对象。  

>一条最普通的原则，正常化的数据应当包含尽可能多的信息。  


