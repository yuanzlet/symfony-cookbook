# 如何创建一个自定义表单域类型

Symfony 具有很多核心的字段类型来建立表单。然而有些情况下为了特定的目的你可能想要创建一个自定义的表单字段类型。本指导假设你需要定义人的性别的字段，基于存在的选择字段。这一节介绍这个字段是如何定义的，你将如何自定义它的输出以及最后你将如何将它注册到你的应用程序中来使用。  

## 定义字段类型

为了创建自定义的字段类型，首先你需要建立和字段相关的类。在这种情况下字段类型的类将会调用 **GenderType** 以及文件将会默认被储存在表单字段的默认位置中，也就是 **<BundleName>\Form\Type**。确保字段扩展了 [AbstractType](http://api.symfony.com/2.7/Symfony/Component/Form/AbstractType.html)：  

```
// src/AppBundle/Form/Type/GenderType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\OptionsResolver\OptionsResolver;

class GenderType extends AbstractType
{
    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'choices' => array(
                'm' => 'Male',
                'f' => 'Female',
            )
        ));
    }

    public function getParent()
    {
        return 'choice';
    }

    public function getName()
    {
        return 'gender';
    }
}
```  

> 文件的位置并不是很重要——**Form\Type** 目录只是一个惯例。  

在这里，**getParent** 功能返回的值表示你正在扩展**选择**字段类型。这也就意味着在默认情况下你继承了所有的逻辑，同时渲染了所有字段类型。你可以到 [ChoiceType](https://github.com/symfony/symfony/blob/master/src/Symfony/Component/Form/Extension/Core/Type/ChoiceType.php) 类去看看这些逻辑。这里有三种十分重要的方法：  

**buildForm()**  
每一个字段类型都有 **buildForm()** 方法，你可以在这个方法中配置建立任何字段。注意这是和你建立*你的*表单相同的方法，同时它也可以在这里起作用。  

**buildView()**  
这个方法是用来建立在你的模板中渲染字段时你所需要的任意变量。举例来说，在 [ChoiceType](https://github.com/symfony/symfony/blob/master/src/Symfony/Component/Form/Extension/Core/Type/ChoiceType.php) 中，**multiple** 变量在模板中配置和使用来设置（或者不设置）**select** 字段的 **multiple** 属性。更多细节详见[为字段建立模板](http://symfony.com/doc/current/cookbook/form/create_custom_field_type.html#creating-a-template-for-the-field)。  

> **configureOptions()** 方法是在 Symfony 2.7 中引进的。在之前的版本中这个方法叫做 **setDefaultOptions()**。  

**configureOptions()**  
这个方法为你的表单类型定义了选项，这是可以用在 **buildForm()** 和 **buildView()** 中的选项。对于所有的字段有很多普通的选项（详见[表单字段类型](http://symfony.com/doc/current/reference/forms/types/form.html)），但是在这里你可以创建你所需要的。  

> 如果你创建了一个包含了很多字段的字段，那么记得将你的“父”类型设置成**表单**或者扩展**表单**的那些。此外，如果你想要修改你的父类型中产生的子类型的任何“视图”，就使用 **finishView()** 方法。  

**getName()** 方法返回一个标识符，这个标识符在你的应用程序中是独特的。这个可以在很多地方应用，比如当你自定义你的表单类型如何渲染时。  

这个字段的目标就是扩展选择类型来启用性别的选择。这个可以通过将**选择**设置成一个可能性别的列表来完成。  

## 为字段创建一个模板

每一个字段类型都是由模板碎片渲染的，这个是由你的 **getName()** 方法的值在某种程度上决定的。获取更多信息详见[什么是表单主题？](http://symfony.com/doc/current/cookbook/form/form_customization.html#cookbook-form-customization-form-themes)。  

在这种情况下，由于父字段是**选择**，你并不需要做任何工作由于定制的字段会自动被当做**选择**类型来渲染。但是为了这个例子考虑，假设当你的字段是“扩大的”（例如 radio button 或者复选框而不是选择字段），你想要一直在 **ul** 元素中渲染它。在你的表单主题模板中（详见上面的链接），创建一个 **gender_widget** 来处理它：  

Twig:

```Twig
{# app/Resources/views/Form/fields.html.twig #}
{% block gender_widget %}
    {% spaceless %}
        {% if expanded %}
            <ul {{ block('widget_container_attributes') }}>
            {% for child in form %}
                <li>
                    {{ form_widget(child) }}
                    {{ form_label(child) }}
                </li>
            {% endfor %}
            </ul>
        {% else %}
            {# just let the choice widget render the select tag #}
            {{ block('choice_widget') }}
        {% endif %}
    {% endspaceless %}
{% endblock %}
```

PHP:

```PHP
<!-- app/Resources/views/Form/gender_widget.html.php -->
<?php if ($expanded) : ?>
    <ul <?php $view['form']->block($form, 'widget_container_attributes') ?>>
    <?php foreach ($form as $child) : ?>
        <li>
            <?php echo $view['form']->widget($child) ?>
            <?php echo $view['form']->label($child) ?>
        </li>
    <?php endforeach ?>
    </ul>
<?php else : ?>
    <!-- just let the choice widget render the select tag -->
    <?php echo $view['form']->renderBlock('choice_widget') ?>
<?php endif ?>
```

> 确保正确的控件前缀被使用。在这个例子中的名称应该是 **gender_widget**，根据 **getName** 返回来的值。进一步来说，主配置文件应当指向定制的表单模板这样就能够在渲染所有表单时使用。  

>当使用 Twig 如下所示：  

>```YAML
># app/config/config.yml
twig:
    form_themes:
        - 'AppBundle:Form:fields.html.twig'
>```

>```XML
><!-- app/config/config.xml -->
<twig:config>
    <twig:form-theme>AppBundle:Form:fields.html.twig</twig:form-theme>
</twig:config>
>```

>```PHP
>// app/config/config.php
$container->loadFromExtension('twig', array(
    'form_themes' => array(
        'AppBundle:Form:fields.html.twig',
    ),
));
>```

>对于 PHP 模板引擎，你的配置应当如下所示：  

>```YAML
># app/config/config.yml
framework:
    templating:
        form:
            resources:
                - 'AppBundle:Form'
>```

>```XML
><!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:framework="http://symfony.com/schema/dic/symfony"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
    http://symfony.com/schema/dic/symfony http://symfony.com/schema/dic/symfony/symfony-1.0.xsd">
    <framework:config>
        <framework:templating>
            <framework:form>
                <framework:resource>AppBundle:Form</twig:resource>
            </framework:form>
        </framework:templating>
    </framework:config>
</container>
>```

>```PHP
>// app/config/config.php
$container->loadFromExtension('framework', array(
    'templating' => array(
        'form' => array(
            'resources' => array(
                'AppBundle:Form',
            ),
        ),
    ),
));
>```

## 使用字段类型

你可以马上使用你的自定义的字段类型，只需要在你的一个表单中创建一个新的类型实例：  

```
// src/AppBundle/Form/Type/AuthorType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;

class AuthorType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('gender_code', new GenderType(), array(
            'placeholder' => 'Choose a gender',
        ));
    }
}
```

但是仅仅是由于 **GenderType()** 很简单所以这个可以起作用。如果性别代码存储在配置或者在数据库中该怎么办？下一节将为你介绍如何使用复杂的字段类型来解决这个问题。  

> **placeholder** 是在 Symfony 2.6 中引入的，支持 **empty_value**，这个在 2.6 之前的版本也可用。  

## 将字段类型作为服务

目前为止，本节已经假设你已经拥有一个简单的定制的字段类型。但是如果你需要访问配置，数据库连接，或者一些其它服务，那么你就需要将你的字段类型注册为服务，举例来说，你将性别参数存储在配置中：  

YAML:

```YAML
# app/config/config.yml
parameters:
    genders:
        m: Male
        f: Female
```

XML:

```XML
<!-- app/config/config.xml -->
<parameters>
    <parameter key="genders" type="collection">
        <parameter key="m">Male</parameter>
        <parameter key="f">Female</parameter>
    </parameter>
</parameters>
```

PHP:

```PHP
// app/config/config.php
$container->setParameter('genders.m', 'Male');
$container->setParameter('genders.f', 'Female');
```

为了使用参数，定义你的自定义字段类型为服务，将**性别**参数值作为第一参数注入到它的将要创建的 **__construct** 函数：  

YAML:

```YAML
# src/AppBundle/Resources/config/services.yml
services:
    app.form.type.gender:
        class: AppBundle\Form\Type\GenderType
        arguments:
            - "%genders%"
        tags:
            - { name: form.type, alias: gender }
```

XML:

```XML
<!-- src/AppBundle/Resources/config/services.xml -->
<service id="app.form.type.gender" class="AppBundle\Form\Type\GenderType">
    <argument>%genders%</argument>
    <tag name="form.type" alias="gender" />
</service>
```

PHP:

```PHP
// src/AppBundle/Resources/config/services.php
use Symfony\Component\DependencyInjection\Definition;

$container
    ->setDefinition('app.form.type.gender', new Definition(
        'AppBundle\Form\Type\GenderType',
        array('%genders%')
    ))
    ->addTag('form.type', array(
        'alias' => 'gender',
    ))
;
```

> 确保服务文件被输入。详见[使用入口输入配置](http://symfony.com/doc/current/book/service_container.html#service-container-imports-directive)。  

确保 tag 的 **alias** 属性和由  方法定义的返回值要相一致。当你使用定制的字段的时候你就体会到这个的重要性。但是首先，向 **GenderType** 添加 **__construct** 方法，这个接收性别的配置：  

```
// src/AppBundle/Form/Type/GenderType.php
namespace AppBundle\Form\Type;

use Symfony\Component\OptionsResolver\OptionsResolver;

// ...

// ...
class GenderType extends AbstractType
{
    private $genderChoices;

    public function __construct(array $genderChoices)
    {
        $this->genderChoices = $genderChoices;
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'choices' => $this->genderChoices,
        ));
    }

    // ...
}
```

非常好！**GenderType** 现在已经拥有配置参数并且注册成为了服务。除此之外，因为你在它的配置中使用 **form.type** 别名，现在使用这个字段更容易了：  

```
// src/AppBundle/Form/Type/AuthorType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\FormBuilderInterface;

// ...

class AuthorType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('gender_code', 'gender', array(
            'placeholder' => 'Choose a gender',
        ));
    }
}
```

注意替代实例化一个新的实例，你可以仅仅通过你的服务配置中使用的别名来引用它，性别。玩的开心！