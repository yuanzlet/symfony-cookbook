# 如何使用和注册命名空间路径

通常，当您引用模板的时候，您将使用 **MyBundle:Subdir:filename.html.twig** 格式 (请参见[模板命名和位置](http://symfony.com/doc/current/book/templating.html#template-naming-locations))。

Twig 也以本机方式提供了一种称为“命名空间路径”的功能支持，然后为您的包进行了自动内置。

下面的路径便是一个例子：

```
{% extends "AppBundle::layout.html.twig" %}
{% include "AppBundle:Foo:bar.html.twig" %}
```

使用命名空间的路径，以下的程序将会很好的运行：

```
{% extends "@App/layout.html.twig" %}
{% include "@App/Foo/bar.html.twig" %}
```

在 Symfony 中，默认情况下，这两个路径都是有效的。

> 额外补充一点。使用经过命名空间声明的语法会运行得更快一些。

## 注册自己的命名空间

您也可以注册您自己的自定义命名空间。假设你正在使用一些第三方库，包括存放在 **vendor/acme/foo-bar/templates** 下的 Twig 模板。首先，为这个目录注册一个命名空间：

YAML:

```
# app/config/config.yml
twig:
    # ...
    paths:
        "%kernel.root_dir%/../vendor/acme/foo-bar/templates": foo_bar
```

XML:

```
<!-- app/config/config.xml -->
<?xml version="1.0" ?>
<container xmlns="http://symfony.com/schema/dic/services"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:twig="http://symfony.com/schema/dic/twig"
>

    <twig:config debug="%kernel.debug%" strict-variables="%kernel.debug%">
        <twig:path namespace="foo_bar">%kernel.root_dir%/../vendor/acme/foo-bar/templates</twig:path>
    </twig:config>
</container>
```

PHP:

```
// app/config/config.php
$container->loadFromExtension('twig', array(
    'paths' => array(
        '%kernel.root_dir%/../vendor/acme/foo-bar/templates' => 'foo_bar',
    );
));
```

注册命名空间被称为 **foo_bar**，对应的是 **vendor/acme/foo-bar/templates** 目录。假设在该目录中的文件有一个叫做 **sidebar.twig** 的文件，那么您可以通过下面的代码来使用它：

```
{% include '@foo_bar/sidebar.twig' %}
```

## 每个命名空间对应多个路径

也可以将几个路径分配到相同的模板命名空间。但是在其中配置路径的顺序是非常重要的，因为如果配置的路径存在， Twig 总是从第一个配置的路径开始加载。当特定的模板不存在时，此功能可以作为一种回退机制来加载通用模板。

YAML:

```
# app/config/config.yml
twig:
    # ...
    paths:
        "%kernel.root_dir%/../vendor/acme/themes/theme1": theme
        "%kernel.root_dir%/../vendor/acme/themes/theme2": theme
        "%kernel.root_dir%/../vendor/acme/themes/common": theme
```

XML:

```
<!-- app/config/config.xml -->
<?xml version="1.0" ?>
<container xmlns="http://symfony.com/schema/dic/services"
           xmlns:twig="http://symfony.com/schema/dic/twig"
>

    <twig:config debug="%kernel.debug%" strict-variables="%kernel.debug%">
        <twig:path namespace="theme">%kernel.root_dir%/../vendor/acme/themes/theme1</twig:path>
        <twig:path namespace="theme">%kernel.root_dir%/../vendor/acme/themes/theme2</twig:path>
        <twig:path namespace="theme">%kernel.root_dir%/../vendor/acme/themes/common</twig:path>
    </twig:config>
</container>
```

PHP:

```
// app/config/config.php
$container->loadFromExtension('twig', array(
    'paths' => array(
        '%kernel.root_dir%/../vendor/acme/themes/theme1' => 'theme',
        '%kernel.root_dir%/../vendor/acme/themes/theme2' => 'theme',
        '%kernel.root_dir%/../vendor/acme/themes/common' => 'theme',
    ),
));
```

现在，你可以使用相同的 **@theme** 命名空间来指位于前三个目录中的任何模板：

```
{% include '@theme/header.twig' %}
```
