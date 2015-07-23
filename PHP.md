# 如何在模板中使用 PHP 而不是 Twig

Symfony 默认 Twig 作为其模板引擎，但如果你想，你仍然可以使用纯 PHP 代码。在 Symfony 中，两种模板引擎都同样支持。Symfony 中在 PHP 上添加了一些不错的功能，使得使用 PHP 来编写模板更强大。

## 呈现PHP模板

如果你想使用 PHP 模板引擎，首先，请确保在应用程序配置文件中启用它：

YAML：
```
# app/config/config.yml
framework:
    # ...
    templating:
        engines: ['twig', 'php']
```

XML：
```
<!-- app/config/config.xml -->
<framework:config>
    <!-- ... -->
    <framework:templating>
        <framework:engine id="twig" />
        <framework:engine id="php" />
    </framework:templating>
</framework:config
```

PHP：
```
$container->loadFromExtension('framework', array(
    // ...
    'templating' => array(
        'engines' => array('twig', 'php'),
    ),
));
```

现在，您可以仅仅通过在模板中使用 **.php** 扩展名代替 **.twig** 来提供一个 PHP 模板，而不是一个 Twig 模板。该控制器下呈现 **index.html.php** 模板：

```
// src/AppBundle/Controller/HelloController.php

// ...
public function indexAction($name)
{
    return $this->render(
        'AppBundle:Hello:index.html.php',
        array('name' => $name)
    );
}
```

您还可以使用 [@Template](https://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/annotations/view%60) 快捷方式提供默认的 **AppBundle:Hello:index.html.php** 模板：

```
// src/AppBundle/Controller/HelloController.php
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;

// ...

/**
 * @Template(engine="php")
 */
public function indexAction($name)
{
    return array('name' => $name);
}
```

> 同时启用 **php** 和 **twig** 模板引擎是允许的，但它会对您的应用程序产生不良的副作用：Twig 命名空间的 **@** 符号将不再支持 **render()** 方法：
> ```
> public function indexAction()
> {
>     // ...
> 
>     // namespaced templates will no longer work in controllers
>     $this->render('@App/Default/index.html.twig');
> 
>     // you must use the traditional template notation
>     $this->render('AppBundle:Default:index.html.twig');
> }
> ```
> ```
> {# inside a Twig template, namespaced templates work as expected #}
> {{ include('@App/Default/index.html.twig') }}
> 
> {# traditional template notation will also work #}
> {{ include('AppBundle:Default:index.html.twig') }}
> ```

## 修饰模板

通常情况下，模板项目中共享公用元素，像众所周知的页眉和页脚。在 Symfony 中，这个问题是以另一不同的方式被考虑的：一个模板可以由另一个修饰。

由于 **extend()** 的调用，**index.html.php** 模板是被 **layout.html.php** 修饰的。

```
<!-- app/Resources/views/Hello/index.html.php -->
<?php $view->extend('AppBundle::layout.html.php') ?>

Hello <?php echo $name ?>!
```

**AppBundle::layout.html.php** 符号听起来很熟悉，不是吗？它是用于引用模板的相同的符号。**::** 部分只是意味着控制器的元素是空的，所以相应的文件直接存储在 **views/** 下。

现在，我们来看一下 **layout.html.php** 文件：

```
<!-- app/Resources/views/layout.html.php -->
<?php $view->extend('::base.html.php') ?>

<h1>Hello Application</h1>

<?php $view['slots']->output('_content') ?>
```

布局本身就是被另一个修饰(**::base.html.php**)。 Symfony 支持多个修饰水平：一个布局本身可以被另一个装饰。在包的部分模板名字为空时，会在 **app/Resources/views/** 目录下寻找视图。此目录为您的工程存储全局视图：

```
<!-- app/Resources/views/base.html.php -->
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <title><?php $view['slots']->output('title', 'Hello Application') ?></title>
    </head>
    <body>
        <?php $view['slots']->output('_content') ?>
    </body>
</html>
```

这两种布局中，**$view['slots']->output('_content')** 的表达方式被 **index.html.php** 和 **layout.html.php** 各自的子模版的内容所取代（在下一节中有更多关于 slot 的信息）。

如您所见，Symfony 提供了在一个神秘的 $view 对象上的方法。在模板中，$view 变量总是可得到的并且指向一个特殊的对象，该对象提供一堆方法对模板引擎进行标记。

## 使用 Slots

一个 slot 是代码的一个片段，定义在模板中，且可重用于任何布局来修饰模板。在 **index.html.php** 模板中，定义一个**标题** slot：

```
<!-- app/Resources/views/Hello/index.html.php -->
<?php $view->extend('AppBundle::layout.html.php') ?>

<?php $view['slots']->set('title', 'Hello World Application') ?>

Hello <?php echo $name ?>!
```

基本布局已经有代码来输出页眉标题：

```
<!-- app/Resources/views/base.html.php -->
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <title><?php $view['slots']->output('title', 'Hello Application') ?></title>
</head>
```

在 **output()** 方法中插入一个 slot 的内容，并且如果 slot 未定义，则可以选择采用默认值。并且 **_content** 只是一个特殊的包含渲染后的子模版的 slot。

对于大型的 slots，还有一个扩展语法：

```
<?php $view['slots']->start('title') ?>
    Some large amount of HTML
<?php $view['slots']->stop() ?>
```

## 包括其他模板

共享模板代码的一个片段的最好的方法是定义一个可以被纳入其他模板的模板。

创建一个 **hello.html.php** 模板：

```
<!-- app/Resources/views/Hello/hello.html.php -->
Hello <?php echo $name ?>!
```

然后修改 **index.html.php** 模板来包含上面的模板：

```
<!-- app/Resources/views/Hello/index.html.php -->
<?php $view->extend('AppBundle::layout.html.php') ?>

<?php echo $view->render('AppBundle:Hello:hello.html.php', array('name' => $name)) ?>
```

**render()** 方法计算并返回另一个模板的内容(这是作为在控制器中使用的完全相同的方法)。

## 嵌入其他控制器

如果要嵌入模板中的另一个控制器的结果要怎样做呢？当处理 Ajax 时，这是非常有用的，或在嵌入模板时，需要主模板中一些不可用的变量。

如果你创建了一个 **fancy** 活动，并且希望把它包含到 **index.html.php template** 模板中，只需要使用下面的代码：

```
<!-- app/Resources/views/Hello/index.html.php -->
<?php echo $view['actions']->render(
    new \Symfony\Component\HttpKernel\Controller\ControllerReference('AppBundle:Hello:fancy', array(
        'name'  => $name,
        'color' => 'green',
    ))
) ?>
```

这里，**AppBundle:Hello:fancy** 字符串指代 **Hello** 控制器中的 **fancy** 活动：

```
// src/AppBundle/Controller/HelloController.php

class HelloController extends Controller
{
    public function fancyAction($name, $color)
    {
        // create some object, based on the $color variable
        $object = ...;

        return $this->render('AppBundle:Hello:fancy.html.php', array(
            'name'   => $name,
            'object' => $object
        ));
    }

    // ...
}
```

但是 **$view['actions']** 数组元素是在哪里定义的呢？像 **$view['slots']** 一样，它被叫做模板助手，下一节会告诉您更多这方面的信息。

## 使用模板助手

Symfony 的模板系统可以很容易地通过助手（helpers）扩展。Helpers 是在模板中提供实用功能的 PHP 对象。Symfony 的 **actions** 和 **slots** 是内置的两个助手。

## 创建两个页面之间的链接

说到 web 应用程序，创建页面之间的联系是必须的。作为对模板中的 URLs 进行硬编码的替换，**路由**助手知道如何生成基于路由配置的 URLs。这样，所有你可以很容易地更改配置更新的 URLs：

```
<a href="<?php echo $view['router']->generate('hello', array('name' => 'Thomas')) ?>">
    Greet Thomas!
</a>
```

**generate()** 方法需要路由名称和一个参数数组作为参数。路由名称是引用该主键下的被引用的路由并且参数是路由模式中定义的占位符的值：

```
# src/AppBundle/Resources/config/routing.yml
hello: # The route name
    path:     /hello/{name}
    defaults: { _controller: AppBundle:Hello:index }
```

## 使用 Assets: Images, JavaScripts 和 Stylesheets

如果没有 images, JavaScripts, 和 stylesheets，互联网会是什么样的呢？Symfony 提供了 **assets** 标记来容易地处理它们：

```
<link href="<?php echo $view['assets']->getUrl('css/blog.css') ?>" rel="stylesheet" type="text/css" />

<img src="<?php echo $view['assets']->getUrl('images/logo.png') ?>" />
```

**assets** helper 的主要目的是使您的应用程序更便携。由于这个 helper 不更改任何模板中的代码，您可以把应用程序根目录移动到在您的 web 根目录之下的任何地方。

## 分析模板

通过使用 **stopwatch** 帮助器，您可以对您的模板部分计时并将其显示在 WebProfilerBundle 的 timeline 上：

```
<?php $view['stopwatch']->start('foo') ?>
... things that get timed
<?php $view['stopwatch']->stop('foo') ?>
```

> 如果您在您的模板上使用超过一次相同的名称，timeline 中的时间将按相同的 line 来进行分组。

## 输出转义

使用 PHP 模板时，每当变量向用户展示时对变量转义：

```
<?php echo $view->escape($var) ?>
```

默认情况下，**escape()** 方法假定变量是在 HTML 环境范围内输出。第二个参数允许您更改环境。例如，在一个 JavaScript 脚本中的输出，使用 **js** 环境：

```
<?php echo $view->escape($var, 'js') ?>
```
