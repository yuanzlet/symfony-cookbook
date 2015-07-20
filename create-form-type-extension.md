# 如何创建一个表单类型扩展

当你需要特定目的的表单类型时[自定义表单字段类型](http://symfony.com/doc/current/cookbook/form/create_custom_field_type.html)就很好，例如性别选择或者增值税发票号码输入。  

但是有时候，你不需要添加新的字段类型——你只是想要在已经存在的上面添加一些特征。这时候表单类型扩展就出现了。  

表单类型扩展主要有两种用途：  

1. 你想要**向几种类型中添加一般特征**（例如向每一个字段类型添加“帮助”文本）；  
2. 你想要**向一种类型中添加特定特征**（例如向“文件”字段添加“下载”特征）。

在这两种情况下，通过定制的表单渲染或者定制的表单字段类型可能能实现你的目标。但是使用表单类型扩展可以更加清楚（通过限制模板中的业务逻辑）并且更灵活（你可以向一个表单类型中添加几个类型扩展）。  

表单类型扩展能够完成大多是定制的字段类型能做的事情，但是代替自己成为字段类型，**他们插入到已经存在的类型中**。  

假设你管理一个**媒体**实体，并且每一个媒体都和一个文件相关联。你的**媒体**表单使用了文件类型，但是当编辑这个实体的时候，你就会看到它的挨着文件输入的图像自动渲染。  

你当然可以通过在模板中配置字段如何被渲染。但是字段类型扩展允许你以一种更加流行的方式。  

## 定义表单类型扩展

你的第一个任务就是创建一个表单类型扩展类（在本文中叫做 **ImageTypeExtension**）。标准情况下表单类型扩展通常位于你的某一个 bundle 的 **Form\Extension** 目录之下。  

当创建表单类型扩展的时候，你可以启用 [FormTypeExtensionInterface](http://api.symfony.com/2.7/Symfony/Component/Form/FormTypeExtensionInterface.html) 界面或者扩展 [AbstractTypeExtension](http://api.symfony.com/2.7/Symfony/Component/Form/AbstractTypeExtension.html) 类。大多数情况下扩展 abstract 类更容易一些：  

```
// src/Acme/DemoBundle/Form/Extension/ImageTypeExtension.php
namespace Acme\DemoBundle\Form\Extension;

use Symfony\Component\Form\AbstractTypeExtension;

class ImageTypeExtension extends AbstractTypeExtension
{
    /**
     * Returns the name of the type being extended.
     *
     * @return string The name of the type being extended
     */
    public function getExtendedType()
    {
        return 'file';
    }
}
```

你必须使用的一个方法就是 **getExtendedType** 功能。它是用来被你的扩展所扩展的表单类型的名称的。  

>**getExtendedType** 方法返回的值和你希望扩展的表单类型类的 **getName** 方法所返回的值相一致。  

处理 **getExtendedType** 方法，你可能还想重写下列的一个方法：  

- **buildForm()**
- **buildView()**
- **configureOptions()**
- **finishView()**

有关于这些方法的用途的更多信息，你可以参考[创建自定义表单类型](http://symfony.com/doc/current/cookbook/form/create_custom_field_type.html)这篇指导文章。  

## 将你的表单类型扩展注册为服务

接下来这一步就是使得 Symfony 知道你的扩展。你所需要做的就是使用 **form.type_extension** 标签将它声明为一个服务：  

```YAML
services:
    acme_demo_bundle.image_type_extension:
        class: Acme\DemoBundle\Form\Extension\ImageTypeExtension
        tags:
            - { name: form.type_extension, alias: file }
```

```XML
<service id="acme_demo_bundle.image_type_extension"
    class="Acme\DemoBundle\Form\Extension\ImageTypeExtension"
>
    <tag name="form.type_extension" alias="file" />
</service>
```

```PHP
$container
    ->register(
        'acme_demo_bundle.image_type_extension',
        'Acme\DemoBundle\Form\Extension\ImageTypeExtension'
    )
    ->addTag('form.type_extension', array('alias' => 'file'));
```

标签的**别名**值就是扩展将要应用到的字段的类型。在你应用的时候，如果你想要扩展**文件**字段类型，你就可以用**文件**作为别名。  

## 给扩展添加业务逻辑

你的扩展的目标就是展示完美的图片在文件输入旁边（当基本类型包括图片的时候）。为了达到这个目的，假设你是用的方法和[如何使用 Doctrine 处理文件上传](http://symfony.com/doc/current/cookbook/doctrine/file_uploads.html)中所描述的一样：你有一个具有文件属性的媒体模型（和表单中的文件字段相对应）以及一个路径属性（和数据库中的图片路径相对应）：  

```
// src/Acme/DemoBundle/Entity/Media.php
namespace Acme\DemoBundle\Entity;

use Symfony\Component\Validator\Constraints as Assert;

class Media
{
    // ...

    /**
     * @var string The path - typically stored in the database
     */
    private $path;

    /**
     * @var \Symfony\Component\HttpFoundation\File\UploadedFile
     * @Assert\File(maxSize="2M")
     */
    public $file;

    // ...

    /**
     * Get the image URL
     *
     * @return null|string
     */
    public function getWebPath()
    {
        // ... $webPath being the full image URL, to be used in templates

        return $webPath;
    }
}
```

你的表单类型扩展类将需要做两件事来扩展**文件**表单类型：  

1. 重写 **configureOptions** 方法来添加 **image_path** 选项；
2. 重写 **buildForm** 和 **buildView** 方法来将图片的地址传递到视图。  

逻辑如下：当添加**文件**类型的表单字段，你就能制定一个新的选项：**image_path**。这个选项将会告诉文件字段如何获得实际的图片地址并且在视图中展示它：  

```
// src/Acme/DemoBundle/Form/Extension/ImageTypeExtension.php
namespace Acme\DemoBundle\Form\Extension;

use Symfony\Component\Form\AbstractTypeExtension;
use Symfony\Component\Form\FormView;
use Symfony\Component\Form\FormInterface;
use Symfony\Component\PropertyAccess\PropertyAccess;
use Symfony\Component\OptionsResolver\OptionsResolver;

class ImageTypeExtension extends AbstractTypeExtension
{
    /**
     * Returns the name of the type being extended.
     *
     * @return string The name of the type being extended
     */
    public function getExtendedType()
    {
        return 'file';
    }

    /**
     * Add the image_path option
     *
     * @param OptionsResolver $resolver
     */
    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefined(array('image_path'));
    }

    /**
     * Pass the image URL to the view
     *
     * @param FormView $view
     * @param FormInterface $form
     * @param array $options
     */
    public function buildView(FormView $view, FormInterface $form, array $options)
    {
        if (array_key_exists('image_path', $options)) {
            $parentData = $form->getParent()->getData();

            if (null !== $parentData) {
                $accessor = PropertyAccess::createPropertyAccessor();
                $imageUrl = $accessor->getValue($parentData, $options['image_path']);
            } else {
                 $imageUrl = null;
            }

            // set an "image_url" variable that will be available when rendering this field
            $view->vars['image_url'] = $imageUrl;
        }
    }

}
```

## 重写文件控件模板碎片

每一个字段类型都是由模板碎片所渲染的。那些模板碎片可以被重写从而来自定义表单渲染。获取更多信息，你可以阅读[什么是表单主题？](http://symfony.com/doc/current/cookbook/form/form_customization.html#cookbook-form-customization-form-themes)这篇文章。  

在你的扩展类之中，你已经添加了一个新的变量（**image_url**），但是你依旧需要使用你的模板中的新的变量。特别的，你需要重写 **file_widget** 区域：  

```Twig
{# src/Acme/DemoBundle/Resources/views/Form/fields.html.twig #}
{% extends 'form_div_layout.html.twig' %}

{% block file_widget %}
    {% spaceless %}

    {{ block('form_widget') }}
    {% if image_url is not null %}
        <img src="{{ asset(image_url) }}"/>
    {% endif %}

    {% endspaceless %}
{% endblock %}
```

```PHP
<!-- src/Acme/DemoBundle/Resources/views/Form/file_widget.html.php -->
<?php echo $view['form']->widget($form) ?>
<?php if (null !== $image_url): ?>
    <img src="<?php echo $view['assets']->getUrl($image_url) ?>"/>
<?php endif ?>
```

>你需要改变你的配置文件或者明确指定你想要如何给你的表单加主题为了使 Symfony 使用你所重写的区域。更多信息详见[什么是表单主题？](http://symfony.com/doc/current/cookbook/form/form_customization.html#cookbook-form-customization-form-themes)这篇文章。  

## 使用表单类型扩展

从现在起，当在你的表单中添加**文件**类型的字段时，你就可以指定 **image_path** 选项，这个选项将用来展示文件字段旁的图片。举例来说：  

```
// src/Acme/DemoBundle/Form/Type/MediaType.php
namespace Acme\DemoBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;

class MediaType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('name', 'text')
            ->add('file', 'file', array('image_path' => 'webPath'));
    }

    public function getName()
    {
        return 'media';
    }
}
```

当展示表单的时候，如果基本的模型已经和图片关联，你就会看到它在文件输入旁边显示。  


