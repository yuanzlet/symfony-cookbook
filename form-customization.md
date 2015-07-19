# 如何自定义表单渲染

Symfony 提供了很多种来渲染你的表达的方法。在本指导中，你将学习如何自定义你的表单的每一个可能的部分使用尽可能少的步骤不论你在你的模板引擎中使用的是 Twig 还是 PHP。  

## 表单渲染基础 ##

召回表单区域的标签，错误以及 HTML 插件可以很容易地通过使用 Twig 的 **form_row** 功能或者 PHP 的帮助方法来进行渲染：  

```Twig
{{ form_row(form.age) }}
```  

```PHP
<?php echo $view['form']->row($form['age']); ?>
```  

你也可以分别渲染这三个部分：  

```Twig
<div>
    {{ form_label(form.age) }}
    {{ form_errors(form.age) }}
    {{ form_widget(form.age) }}
</div>
```  

```PHP
<div>
    <?php echo $view['form']->label($form['age']); ?>
    <?php echo $view['form']->errors($form['age']); ?>
    <?php echo $view['form']->widget($form['age']); ?>
</div>
```  

在这两种情况下，表单的标签，错误以及 HTML 插件通过使用一系列的 Symfony 包含的标记进行了渲染。举例来说，上述两个模板都会被渲染：  

```
<div>
    <label for="form_age">Age</label>
    <ul>
        <li>This field is required</li>
    </ul>
    <input type="number" id="form_age" name="form[age]" />
</div>
```  

为了快速使用原型并且测试表单，你可以只使用一行来渲染整个表单：  

```	Twig
{# renders all fields #}
{{ form_widget(form) }}

{# renders all fields *and* the form start and end tags #}
{{ form(form) }}
```

```PHP
<!-- renders all fields -->
<?php echo $view['form']->widget($form) ?>

<!-- renders all fields *and* the form start and end tags -->
<?php echo $view['form']->form($form) ?>
```  

本指导接下来的部分将解释表单的每一部分的标记如何在几个不同的水平来调整。获取更多关于表单渲染的一般信息详见在[模板中渲染表单](http://symfony.com/doc/current/book/forms.html#form-rendering-template)。  

## 什么是表单的主题？ ##

Symfony 使用表单碎片——一小块模板用来渲染一小块表单——来渲染表单的每一部分——区域标签，错误，**输入**文本框，**选择**标签等等。  

这个碎片在 Twig 中被定义为区域同时在 PHP 中被定义为模板文件。  

*主题*只不过是一些碎片的集合，这些碎片就是你在渲染表单时想要使用的。换句话说，如果你想要个性化一部分表单是如何被渲染的，你需要输入一个*主题*这个主题包含了适合的个性化的表单碎片。  

Symfony 中包含四个**内建的表单主题**这些主题定义了需要渲染的表单的每一个部分的每一个碎片：  

- [form_div_layout.html.twig](https://github.com/symfony/symfony/blob/master/src/Symfony/Bridge/Twig/Resources/views/Form/form_div_layout.html.twig)，将每一个表单的字段绕在 **\<div\>** 元素中。
- [form_table_layout.html.twig](https://github.com/symfony/symfony/blob/master/src/Symfony/Bridge/Twig/Resources/views/Form/form_table_layout.html.twig)，将整个表单绕在 **\<table\>** 元素中以及将每一个表单字段绕在 **\<tr\>** 元素中。
- [bootstrap_3_layout.html.twig](https://github.com/symfony/symfony/blob/master/src/Symfony/Bridge/Twig/Resources/views/Form/bootstrap_3_layout.html.twig)，使用合适的 CSS 类将整个表单绕在 **\<div\>** 元素中并且应用默认的 [Bootstrap 3 CSS framework](http://getbootstrap.com/) 风格。
- [bootstrap_3_horizontal_layout.html.twig](https://github.com/symfony/symfony/blob/master/src/Symfony/Bridge/Twig/Resources/views/Form/bootstrap_3_horizontal_layout.html.twig)，这个和前面的主题很相似，但是 CSS 类应用的是那些用于水平展示表单的（例如标签以及在同一行的控件）。  

>当你使用 Bootstrap 表单主题时并且手动渲染表单时，对  字段调用 **form_label()** 并不会意味着什么。由于 Bootstrap 内部，标签已经在 **form_widget()** 中显示。  

在下一节中，你将学习到如何个性化主题通过重写或者部分重写它的碎片。  

举例来说，当**整型**字段的控件被渲染时，**input number** 字段就会产生：  

```Twig
{{ form_widget(form.age) }}
```

```PHP
<?php echo $view['form']->widget($form['age']) ?>
```  

渲染：

```
<input type="number" id="form_age" name="form[age]" required="required" value="33" />
```  

内部的，Symfony 使用 **integer_widget** 碎片来渲染字段。这是因为字段类型是**整型**并且你正在渲染它的**控件**（这个和它的**标签**或者**错误**截然相反）。  

在 Twig 中这个将会从 [form_div_layout.html.twig](https://github.com/symfony/symfony/blob/master/src/Symfony/Bridge/Twig/Resources/views/Form/form_div_layout.html.twig) 模板中默认到 **integer_widget** 区域。  

在 PHP 中将会是位于 **FrameworkBundle/Resources/views/Form** 文件夹的 **integer_widget.html.php** 文件。  

默认的 **integer_widget** 的安装启用如下所示：  

```Twig
{# form_div_layout.html.twig #}
{% block integer_widget %}
    {% set type = type|default('number') %}
    {{ block('form_widget_simple') }}
{% endblock integer_widget %}
```

```PHP
<!-- integer_widget.html.php -->
<?php echo $view['form']->block($form, 'form_widget_simple', array('type' => isset($type) ? $type : "number")) ?>
```  

正如你所见，这个碎片自己渲染另一个碎片——**form_widget_simple**：  

```Twig
{# form_div_layout.html.twig #}
{% block form_widget_simple %}
    {% set type = type|default('text') %}
    <input type="{{ type }}" {{ block('widget_attributes') }} {% if value is not empty %}value="{{ value }}" {% endif %}/>
{% endblock form_widget_simple %}
```

```PHP
<!-- FrameworkBundle/Resources/views/Form/form_widget_simple.html.php -->
<input
    type="<?php echo isset($type) ? $view->escape($type) : 'text' ?>"
    <?php if (!empty($value)): ?>value="<?php echo $view->escape($value) ?>"<?php endif ?>
    <?php echo $view['form']->block($form, 'widget_attributes') ?>
/>
```  

重点是，碎片指示了表单的每一个部分的 HTML 输出。为了个性化表单输出，你只需要识别并且重写正确的碎片。一系列的表单碎片的结合就是大家所熟知的表单“主题”。当渲染表单的时候，你可以选择应用哪个或者哪些主题。  

在 Twig 中主题是简单的模板文件并且碎片是定义在这些文件中的区域。  

在 PHP 中主题是一个文件夹并且碎片是这个文件夹中的独立的模板文件。  

>**识别哪个区域来个性化**  
>在本例中，个性化的碎片名为 **integer_widget** 因为你想要为所有的**整型**字段类型重写 HTML **控件**，如果你需要个性化 **textarea** 字段，你就要个性化 **textarea_widget**。  

>正如你所见，碎片的名称是由字段类型以及字段的哪一个部分（例如 **widget, label, errors, row**）被渲染组合而成的。同样的，为了个性化错误如何仅使用输入**文本**字段渲染，你需要个性化 **text_errors** 碎片。  

>然而，更常见的是你想要个性化*所有的*字段的错误显示方式。你可以通过个性化 **form_errors** 碎片来达到这个目的。利用字段类型继承的有点。特别的，由于**文本**类型是由**表单**类型扩展而来的，在回到他的父碎片名称之前如果它不存在（例如 **form_errors**），表单的组件将会首先寻找特定类型的碎片（例如 **text_errors**）。  

>获取有关这个话题的更多信息，参见[表单碎片命名](http://symfony.com/doc/current/book/forms.html#form-template-blocks)。  

## 表单主题化 ##

为了看看表单主题化的威力，假设你想要将每一个输入**数字**字段捆绑一个 **div** 标签。做这个的关键就是将 **integer_widget** 碎片个性化。  

## 在 Twig 中进行表单主题化 ##

当在 Twig 中个性化表单字段区域时，在个性化的表单区域可能存在的地方你有两种选择：  

|**方法**|**优点** | **缺点**|  
|---------|:------------:|--------|  
|将相同的模板作为表单插入|快速简单|不能在其他的模板中应用|  
|插入单独分开的模板|可以在很多模板中重复使用|需要创建额外的模板|  

这两种方法都会产生相同的结果但是在不同的情况下各有各的优点。  

### 方法 1：将相同的模板作为表单插入 ###

最简单的个性化 **integer_widget** 区域的方法是直接在实际渲染表单的模板中进行。  

```
{% extends '::base.html.twig' %}

{% form_theme form _self %}

{% block integer_widget %}
    <div class="integer_widget">
        {% set type = type|default('number') %}
        {{ block('form_widget_simple') }}
    </div>
{% endblock %}

{% block content %}
    {# ... render the form #}

    {{ form_row(form.age) }}
{% endblock %}
```  

通过使用特殊的 **{% form_theme form _self %}** 标签，Twig 在相同的模板中寻找任何的重写的表单区域。假设 **form.age** 区域是**整型**类型的字段，当他的控件被渲染时，个性化的 **integer_widget** 区域将会被使用。  

这个方法的缺点就是个性化的表单区域当在其他的模板中渲染其他表单时不能重复利用。换句话说，这个方法在当对你的应用程序中的单一的表单进行表单个性化时最有用。如果你想在你的应用程序的多个（或者所有）表单中重复利用表单个性化，那么请阅读下一节。  

### 方法 2：插入分开的模板 ###

你也可以选择将个性化的 **integer_widget** 模板区域放置于完全分开的模板中。代码以及最终结果是一样的，但是你现在可以重复在多个模板中使用表单个性化：  

```
{# app/Resources/views/Form/fields.html.twig #}
{% block integer_widget %}
    <div class="integer_widget">
        {% set type = type|default('number') %}
        {{ block('form_widget_simple') }}
    </div>
{% endblock %}
```

既然你已经创建了个性化的表单区域，那么你就需要告诉 Symfony 来使用它。在你实际渲染表单的地方插入模板，通过 **form_theme** 标签告诉 Symfony 来使用它：  

```
{% form_theme form 'AppBundle:Form:fields.html.twig' %}

{{ form_widget(form.age) }}
```  

当 **form.age** 标签被渲染的时候，Symfony 将会从新的模板中使用 **integer_widget** 区域同时 **input** 标签将会被捆绑在个性化区域的特定的 **div** 元素上。  

#### 多重模板 ####

一个表单可以通过应用多个模板来进行个性化。为了完成这个，需要使用 **with** 关键字将所有模板的名称作为一个数组传递：  

```
{% form_theme form with ['::common.html.twig', ':Form:fields.html.twig',
                         'AppBundle:Form:fields.html.twig'] %}

{# ... #}
```  

模板可以位于不同的 bundle 同时它们甚至可以储存在全局目录 **app/Resources/views/** 中。  

#### 子表单 ####

你也可以应用表单主题来区分你的子表单：  

```
{% form_theme form.child 'AppBundle:Form:fields.html.twig' %}
```  

当你想要为不同于你的主表单的嵌套的表单定制主题时这个就会很有用。只要区分你的主题就好：  

```
{% form_theme form 'AppBundle:Form:fields.html.twig' %}

{% form_theme form.child 'AppBundle:Form:fields_child.html.twig' %}
```  

## PHP 环境下的表单主题化 ##

当你使用 PHP 作为模板引擎的时候，个性化碎片的唯一方法就是创建一个新的模板文件——这个和使用 Twig 的第二种方法很相似。  

模板文件必须在碎片之后命名。你必须创建一个 **integer_widget.html.php** 文件从而能个性化 **integer_widget** 碎片。  

```
<!-- app/Resources/views/Form/integer_widget.html.php -->
<div class="integer_widget">
    <?php echo $view['form']->block($form, 'form_widget_simple', array('type' => isset($type) ? $type : "number")) ?>
</div>
```  

既然你已经创建了个性化的表单模板，那么你就需要告诉 Symfony 来使用它。在你实际渲染表单的地方插入模板，通过 **setTheme** 帮助方法告诉 Symfony 来使用它：  

```
<?php $view['form']->setTheme($form, array('AppBundle:Form')); ?>

<?php $view['form']->widget($form['age']) ?>
```  

当 **form.age** 标签被渲染的时候，Symfony 将会使用个性化的 **integer_widget.html.php** 模板同时 **input** 标签将会被捆绑在 **div** 元素上。  

如果你想要将主题应用到特定的子表单上，将它传递给 **setTheme** 方法：  

```
<?php $view['form']->setTheme($form['child'], 'AppBundle:Form/Child'); ?>
```  

## 引用基本表单区域（Twig 特定的） ##

目前为止，重写特定的表单区域最好的方法就是从 [form_div_layout.html.twig](https://github.com/symfony/symfony/blob/master/src/Symfony/Bridge/Twig/Resources/views/Form/form_div_layout.html.twig) 复制默认的区域，然后将它粘贴到不同的模板中，然后将它个性化。在很多情况下，你可以通过在个性化时引用基本表单区域来避免这一步。  

这个很容易做，但是如果你的表单区域作为表单位于同一个模板或者分开的模板中就会有轻微的不一样。  

### 从作为表单的相同模板的内部引用区域 ###

通过添加 **use** 标签的方式来在你渲染表单的地方输入区域：  

```
{% use 'form_div_layout.html.twig' with integer_widget as base_integer_widget %}
```

现在，当从 [form_div_layout.html.twig](https://github.com/symfony/symfony/blob/master/src/Symfony/Bridge/Twig/Resources/views/Form/form_div_layout.html.twig) 中来的区域被输入后，**integer_widget** 区域被叫做 **base_integer_widget**。这就意味着当你重新定义 **integer_widget** 区域你可以通过 **base_integer_widget** 引用默认的标记：  

```
{% block integer_widget %}
    <div class="integer_widget">
        {{ block('base_integer_widget') }}
    </div>
{% endblock %}
```  

### 从外部模板引用基本区域 ###

如果你的表单个性化位于外部模板，你可以通过使用 Twig 的 **parent()** 功能来引用基本区域：  

```
{# app/Resources/views/Form/fields.html.twig #}
{% extends 'form_div_layout.html.twig' %}

{% block integer_widget %}
    <div class="integer_widget">
        {{ parent() }}
    </div>
{% endblock %}
```  

>当使用 PHP 作为模板引擎的时候将不可能引用基本区域。你必须手动复制基本区域的内容到你的新模板文件中。  

## 应用程序范围内的自定义 ##

如果你想要将特定的个性化全局应用到你的应用程序之中，你可以通过将表单的个性化放到外部模板然后将它输入到你的应用程序配置之中。  

### Twig ###

通过使用下列的配置，任何在 **AppBundle:Form:fields.html.twig** 模板中的自定义表单区域当表单被渲染时都会被全局使用。  

```YAML
# app/config/config.yml
twig:
    form_themes:
        - 'AppBundle:Form:fields.html.twig'
    # ...
```  

```XML
<!-- app/config/config.xml -->
<twig:config>
    <twig:form-theme>AppBundle:Form:fields.html.twig</twig:form-theme>
    <!-- ... -->
</twig:config>
```  

```PHP
// app/config/config.php
$container->loadFromExtension('twig', array(
    'form_themes' => array(
        'AppBundle:Form:fields.html.twig',
    ),

    // ...
));
```  

默认情况下，当渲染表单时 Twig 使用 *div* 布局。然而，有些人可能更喜欢使用 *table* 渲染表单。使用 **form_table_layout.html.twig** 资源来调用这样的布局：  

```YAML
# app/config/config.yml
twig:
    form_themes:
        - 'form_table_layout.html.twig'
    # ...
```  

```XML
<!-- app/config/config.xml -->
<twig:config>
    <twig:form-theme>form_table_layout.html.twig</twig:form-theme>
    <!-- ... -->
</twig:config>
```  

```PHP
// app/config/config.php
$container->loadFromExtension('twig', array(
    'form_themes' => array(
        'form_table_layout.html.twig',
    ),

    // ...
));
```  

如果你只是想用在一个模板中有更改，在你的模板中添加下列这一行代码而不是将模板添加成资源：  

```
{% form_theme form 'form_table_layout.html.twig' %}
```  

注意 **form** 变量在上述的代码中是你传递到你的模板表单视图变量。  

### PHP ###

通过使用下列配置，任何在 **app/Resources/views/Form** 文件夹下的表单的自定义的碎片在渲染表单时将会被全局应用。  

```YAML
# app/config/config.yml
framework:
    templating:
        form:
            resources:
                - 'AppBundle:Form'
    # ...
```  

```XML
<!-- app/config/config.xml -->
<framework:config>
    <framework:templating>
        <framework:form>
            <resource>AppBundle:Form</resource>
        </framework:form>
    </framework:templating>
    <!-- ... -->
</framework:config>
```  

```PHP
// app/config/config.php
// PHP
$container->loadFromExtension('framework', array(
    'templating' => array(
        'form' => array(
            'resources' => array(
                'AppBundle:Form',
            ),
        ),
     ),

     // ...
));
```  

默认情况下，当渲染表单时 PHP 使用 *div* 布局。然而，有些人可能更喜欢使用 *table* 渲染表单。使用 **FrameworkBundle:FormTable** 资源来调用这样的布局：  

```YAML
# app/config/config.yml
framework:
    templating:
        form:
            resources:
                - 'FrameworkBundle:FormTable'
```

```XML
<!-- app/config/config.xml -->
<framework:config>
    <framework:templating>
        <framework:form>
            <resource>FrameworkBundle:FormTable</resource>
        </framework:form>
    </framework:templating>
    <!-- ... -->
</framework:config>
```

```PHP
// app/config/config.php
$container->loadFromExtension('framework', array(
    'templating' => array(
        'form' => array(
            'resources' => array(
                'FrameworkBundle:FormTable',
            ),
        ),
    ),

     // ...
));
```

如果你只是想用在一个模板中有更改，在你的模板中添加下列这一行代码而不是将模板添加成资源：  

```
<?php $view['form']->setTheme($form, array('FrameworkBundle:FormTable')); ?>
```  

注意 **$form** 变量在上述的代码中是你传递到你的模板表单视图变量。  

## 如何在独立的字段中进行自定义 ##

目前为止，你已经看到了不同的方式来自定义所有的文本字段类型的控件输出。你也可以自定义独立的字段。举例来说，假设你在 **product** 表单中有两个文本字段——**name** 和 **description**——但是你只想自定义其中的一个。这个可以通过自定义名为区域的**代码**属性以及需要自定义的部分的名称的组合的碎片来完成。举例来说，只自定义 **name** 字段：  

```
{% form_theme form _self %}

{% block _product_name_widget %}
    <div class="text_widget">
        {{ block('form_widget_simple') }}
    </div>
{% endblock %}

{{ form_widget(form.name) }}
```  

```
<!-- Main template -->
<?php echo $view['form']->setTheme($form, array('AppBundle:Form')); ?>

<?php echo $view['form']->widget($form['name']); ?>

<!-- app/Resources/views/Form/_product_name_widget.html.php -->
<div class="text_widget">
      echo $view['form']->block('form_widget_simple') ?>
</div>
```  

在这里，**_product_name_widget** 碎片定义了模板使用编号为 **product_name** 的字段（名称是 **product[name]**）。  

>字段的 **product** 属性是表单的名称，这个可能是手动设置的或者是基于你的表单类型名称自动产生的（例如 **ProductType** 等同于 **product**）。如果你不确定你的表单的名称，就需要你的表单名称产生的源。  

>如果你想要改变 **_product_name_widget** 区域的**product** 或者 **name** 属性你可以在你的表单中设置 **block_name** 选项：  

>```
>use Symfony\Component\Form\FormBuilderInterface;

public function buildForm(FormBuilderInterface $builder, array $options)
{
    // ...

    $builder->add('name', 'text', array(
        'block_name' => 'custom_name',
    ));
}
>```

>然后区域的名称就会变成 **_product_custom_name_widget**。

你也可以使用相同的方法来重写整个字段行：  

```Twig
{% form_theme form _self %}

{% block _product_name_row %}
    <div class="name_row">
        {{ form_label(form) }}
        {{ form_errors(form) }}
        {{ form_widget(form) }}
    </div>
{% endblock %}

{{ form_row(form.name) }}
```  

```PHP
<!-- Main template -->
<?php echo $view['form']->setTheme($form, array('AppBundle:Form')); ?>

<?php echo $view['form']->row($form['name']); ?>

<!-- app/Resources/views/Form/_product_name_row.html.php -->
<div class="name_row">
    <?php echo $view['form']->label($form) ?>
    <?php echo $view['form']->errors($form) ?>
    <?php echo $view['form']->widget($form) ?>
</div>
```  

## 其他一些自定义 ##

目前为止，本指导已经介绍过一些如何渲染表单的不同的个性化方法。关键就是要个性化特定的碎片，这个碎片和你想要控制的表单的属性相关（详见[命名表单区域](http://symfony.com/doc/current/cookbook/form/form_customization.html#cookbook-form-customization-sidebar)）  

在下一节中，你将学习如何使几个普通的表单个性化。为了这些个性化，需要使用[表单主题化](http://symfony.com/doc/current/cookbook/form/form_customization.html#cookbook-form-theming-methods)一节中介绍的方法。  

### 个性化错误输出 ###

>表单的组件只是会处理校验错误*如何*被渲染以及不是实际的校验信息。这些错误信息由你应用到你的对象的校验限制自己决定。获取更多信息详见[校验](http://symfony.com/doc/current/book/validation.html)章节。  

当表单提交错误的时候有很多很多不同的方式来决定错误如何被渲染。当你使用 **form_errors** 的时候字段的错误信息就会被渲染：  

```Twig
{{ form_errors(form.age) }}
```

```PHP
<?php echo $view['form']->errors($form['age']); ?>
```  

默认情况下，错误在一个没有排序的列表中被渲染：  

```
<ul>
    <li>This field is required</li>
</ul>
```

为了重写所有文件的错误如何被渲染，简单的复制粘贴并且个性化 **form_errors** 碎片就好。  

```Twig
{# form_errors.html.twig #}
{% block form_errors %}
    {% spaceless %}
        {% if errors|length > 0 %}
        <ul>
            {% for error in errors %}
                <li>{{ error.message }}</li>
            {% endfor %}
        </ul>
        {% endif %}
    {% endspaceless %}
{% endblock form_errors %}
```

```PHP
<!-- form_errors.html.php -->
<?php if ($errors): ?>
    <ul>
        <?php foreach ($errors as $error): ?>
            <li><?php echo $error->getMessage() ?></li>
        <?php endforeach ?>
    </ul>
<?php endif ?>
```  

>如何应用这些个性化参见[表单主题化](http://symfony.com/doc/current/cookbook/form/form_customization.html#cookbook-form-theming-methods)。  

你也可以自定义这些错误输出成为一种特定的字段类型。为了*只是*个性化这些错误使用的标志，遵循和上面相同的步骤但是把内容放到相关的 **_errors** 区域（或者文件以防是 PHP 模板）。举例来说，**text_errors** (或者 **text_errors.html.php**)。  

>找出哪个是你必须个性化的区域或者文件详见[表单碎片命名](http://symfony.com/doc/current/book/forms.html#form-template-blocks)。  

对于你的表单来说是全局的特定的错误（例如不仅仅是一个字段）被分开来渲染。通常在你的表单的顶部：  

```Twig
{{ form_errors(form) }}
```

```PHP
<?php echo $view['form']->render($form); ?>
```

为了*仅仅*个性化这些错误所使用的标记，遵循上述所说的步骤，但是检查 **compound** 的值是否设置为**真**。如果为**真**的话，这就意味着现在正在被渲染的就是字段的集合（例如整个表单），并且不仅仅是一个独立的字段。  

```Twig
{# form_errors.html.twig #}
{% block form_errors %}
    {% spaceless %}
        {% if errors|length > 0 %}
            {% if compound %}
                <ul>
                    {% for error in errors %}
                        <li>{{ error.message }}</li>
                    {% endfor %}
                </ul>
            {% else %}
                {# ... display the errors for a single field #}
            {% endif %}
        {% endif %}
    {% endspaceless %}
{% endblock form_errors %}
```

```PHP
<!-- form_errors.html.php -->
<?php if ($errors): ?>
    <?php if ($compound): ?>
        <ul>
            <?php foreach ($errors as $error): ?>
                <li><?php echo $error->getMessage() ?></li>
            <?php endforeach ?>
        </ul>
    <?php else: ?>
        <!-- ... render the errors for a single field -->
    <?php endif ?>
<?php endif ?>
```

### 个性化“表单行” ###

当你可以管理的时候，最简单的渲染表单的字段的方法就是通过 **form_row** 功能，这个功能渲染标签，错误以及字段的 HTML 插件。为了个性化渲染*所有*表单字段行所使用的标志，重写 **form_row** 碎片。举例来说，你想要在每一个行的周围的 **div** 元素中添加一个类：  

```Twig
{# form_row.html.twig #}
{% block form_row %}
    <div class="form_row">
        {{ form_label(form) }}
        {{ form_errors(form) }}
        {{ form_widget(form) }}
    </div>
{% endblock form_row %}
```

```PHP
<!-- form_row.html.php -->
<div class="form_row">
    <?php echo $view['form']->label($form) ?>
    <?php echo $view['form']->errors($form) ?>
    <?php echo $view['form']->widget($form) ?>
</div>
```

>如何应用这些个性化详见[表单主题化](http://symfony.com/doc/current/cookbook/form/form_customization.html#cookbook-form-theming-methods)。  

### 为字段标签添加必填星号标注 ###

如果你想要将你的所有的必填字段加上必填星号标注（*）,你可以通过个性化 **form_label** 碎片来完成。  

在 Twig 中，如果你正在你的表单的相同的模板中个性化表单，修正 **use** 标签并且添加下列代码：  

```
{% use 'form_div_layout.html.twig' with form_label as base_form_label %}

{% block form_label %}
    {{ block('base_form_label') }}

    {% if required %}
        <span class="required" title="This field is required">*</span>
    {% endif %}
{% endblock %}
```

在 Twig 中，如果你正在你的表单的不同的模板中个性化表单，使用下列代码：  

```
{% extends 'form_div_layout.html.twig' %}

{% block form_label %}
    {{ parent() }}

    {% if required %}
        <span class="required" title="This field is required">*</span>
    {% endif %}
{% endblock %}
```

当使用 PHP 作为模板引擎时你必须从原始模板中复制内容：  

```
<!-- form_label.html.php -->

<!-- original content -->
<?php if ($required) { $label_attr['class'] = trim((isset($label_attr['class']) ? $label_attr['class'] : '').' required'); } ?>
<?php if (!$compound) { $label_attr['for'] = $id; } ?>
<?php if (!$label) { $label = $view['form']->humanize($name); } ?>
<label <?php foreach ($label_attr as $k => $v) { printf('%s="%s" ', $view->escape($k), $view->escape($v)); } ?>><?php echo $view->escape($view['translator']->trans($label, array(), $translation_domain)) ?></label>

<!-- customization -->
<?php if ($required) : ?>
    <span class="required" title="This field is required">*</span>
<?php endif ?>
```  

>如何应用这些个性化详见[表单主题化](http://symfony.com/doc/current/cookbook/form/form_customization.html#cookbook-form-theming-methods)。  

>**仅使用 CSS**

>默认情况下，请求字段的 **label** 标签被一个叫做 **required** CSS 类渲染。因此，你也可以只使用 CSS 来添加星号标注：  

>```
>label.required:before {
    content: "* ";
}
>```

### 添加“帮助”消息 ###

你也可以自定义你的表单控件从而拥有“帮助”信息选项。  

在 Twig 中，如果你正在你的表单的相同的模板中个性化表单，修正 **use** 标签并且添加下列代码：  

```
{% use 'form_div_layout.html.twig' with form_widget_simple as base_form_widget_simple %}

{% block form_widget_simple %}
    {{ block('base_form_widget_simple') }}

    {% if help is defined %}
        <span class="help">{{ help }}</span>
    {% endif %}
{% endblock %}
```

在 Twig 中，如果你正在你的表单的不同的模板中个性化表单，使用下列代码：  

```
{% extends 'form_div_layout.html.twig' %}

{% block form_widget_simple %}
    {{ parent() }}

    {% if help is defined %}
        <span class="help">{{ help }}</span>
    {% endif %}
{% endblock %}
```

当使用 PHP 作为模板引擎时你必须从原始模板中复制内容：  

```
<!-- form_widget_simple.html.php -->

<!-- Original content -->
<input
    type="<?php echo isset($type) ? $view->escape($type) : 'text' ?>"
    <?php if (!empty($value)): ?>value="<?php echo $view->escape($value) ?>"<?php endif ?>
    <?php echo $view['form']->block($form, 'widget_attributes') ?>
/>

<!-- Customization -->
<?php if (isset($help)) : ?>
    <span class="help"><?php echo $view->escape($help) ?></span>
<?php endif ?>
```

为了渲染字段下的帮助信息，传递一个 **help** 变量：  

```Twig
{{ form_widget(form.title, {'help': 'foobar'}) }}
```

```PHP
<?php echo $view['form']->widget($form['title'], array('help' => 'foobar')) ?>
```

>如何应用这些个性化详见[表单主题化](http://symfony.com/doc/current/cookbook/form/form_customization.html#cookbook-form-theming-methods)。  

## 使用表单变量 ##

渲染表单的不同部分的大多数功能（例如表单控件，表单标签，表单错误等等）都可以允许你直接做特定的自定义。看下面这个例子：  

```Twig
{# render a widget, but add a "foo" class to it #}
{{ form_widget(form.name, { 'attr': {'class': 'foo'} }) }}
```

```PHP
<!-- render a widget, but add a "foo" class to it -->
<?php echo $view['form']->widget($form['name'], array(
    'attr' => array(
        'class' => 'foo',
    ),
)) ?>
```

这个包含表单“变量”的数组作为第二变元传递。更多关于这方面的细节，详见[更多关于表单变量](http://symfony.com/doc/current/reference/forms/twig_reference.html#twig-reference-form-variables)。
