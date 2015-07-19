# 使用 PHP 库联合，编译和最小化 Web 资产

Symfony 官方的最佳实践推荐使用 Assetic 来[管理网页资产](http://symfony.com/doc/current/best_practices/web-assets.html)，除非你用的习惯基于 JavaScript 的前端工具。  
即使那些基于 JavaScript 的解决方案大多数适用于那些从技术角度来说的案例，使用纯的 PHP 可选函数库在一些脚本中也会很有用：  

- 如果你不能安装或者使用 **npm** 和其他的 JavaScript 解决方案；  
- 如果你更喜欢限制你的应用程序中使用不同技术的数量；  
- 如果你想要简化程序开发。  

在这篇文章中，你将学会如何合并和裁剪 CSS 和 JavaScript 文件，同时学会如何用 Assetic 应用单一的 PHP 函数库来编译 Sass 文件。  

## 安装第三方压缩函数库

Assetic 包含了很多可以使用的过滤器，但是并不包括与它们相联系的函数库。因此，在本文中你使用这些过滤器之前，你必须安装两个函数库。打开命令控制台，浏览你的工程目录并执行下列命令：  

```
$ composer require leafo/scssphp
$ composer require patchwork/jsqueeze:"~1.0"
```  

为 **jsqueeze** 添加一个 **~1.0** 的版本限制很重要，因为大多数最近稳定的版本都和 Assetic 不兼容。  

## 组织你的网页资产文件

这个例子将会包括一个使用 Bootstrap CSS 框架，jQuery， FontAwesome 和一些常规的 CSS 和 JavaScript 应用程序文件（被称为 **main.css** 和 **main.js**）。这个设置的推荐的目录结构如下所示:  

```
web/assets/
├── css
│   ├── main.css
│   └── code-highlight.css
├── js
│   ├── bootstrap.js
│   ├── jquery.js
│   └── main.js
└── scss
    ├── bootstrap
    │   ├── _alerts.scss
    │   ├── ...
    │   ├── _variables.scss
    │   ├── _wells.scss
    │   └── mixins
    │       ├── _alerts.scss
    │       ├── ...
    │       └── _vendor-prefixes.scss
    ├── bootstrap.scss
    ├── font-awesome
    │   ├── _animated.scss
    │   ├── ...
    │   └── _variables.scss
    └── font-awesome.scss
```  

## 合并和裁剪 CSS 和 JavaScript 文件

首先，设置一个新的 **scssphp** Assetic 过滤器：  

YAML:

```YAML
# app/config/config.yml
assetic:
    filters:
        scssphp:
            formatter: 'Leafo\ScssPhp\Formatter\Compressed'
        # ...
```  

XML:

```XML
<!-- app/config/config.xml -->
<?xml version="1.0" charset="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:assetic="http://symfony.com/schema/dic/assetic">

    <assetic:config>
        <filter name="scssphp" formatter="Leafo\ScssPhp\Formatter\Compressed" />
        <!-- ... -->
    </assetic:config>
</container>
```  

PHP:

```PHP
// app/config/config.php
$container->loadFromExtension('assetic', array(
    'filters' => array(
         'scssphp' => array(
             'formatter' => 'Leafo\ScssPhp\Formatter\Compressed',
         ),
         // ...
    ),
));
```  

**formatter** 选项的值是过滤器用来产生编译过的 CSS 文件的格式器的完全保留的类名称。使用压缩格式器将会缩小最终文件，不管原始文件是平常的 CSS 文件还是 SCSS 文件。  

接下来，更新你的 Twig 模板，添加由 Assetic 定义的 **{% stylesheets %}** 标签：  

```
{# app/Resources/views/base.html.twig #}
<!DOCTYPE html>
<html>
    <head>
        <!-- ... -->

        {% stylesheets filter="scssphp" output="css/app.css"
            "assets/scss/bootstrap.scss"
            "assets/scss/font-awesome.scss"
            "assets/css/*.css"
        %}
            <link rel="stylesheet" href="{{ asset_url }}" />
        {% endstylesheets %}
```  

这个简单的设置编译，合并，压缩了 SCSS 文件使之成为普通的可以放进 **web/css/app.css** 文件夹的 CSS 文件。这是唯一的一个提供给你的访问者的 CSS 文件。  

## 合并和压缩 JavaScript 文件

首先，按照下面步骤设置一个新的 **jsqueeze** Assetic 过滤器：  

YAML:

```YAML
# app/config/config.yml
assetic:
    filters:
        jsqueeze: ~
        # ...
```  

XML:

```XML
<!-- app/config/config.xml -->
<?xml version="1.0" charset="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:assetic="http://symfony.com/schema/dic/assetic">

    <assetic:config>
        <filter name="jsqueeze" />
        <!-- ... -->
    </assetic:config>
</container>
```  

PHP:

```PHP
// app/config/config.php
$container->loadFromExtension('assetic', array(
    'filters' => array(
         'jsqueeze' => null,
         // ...
    ),
));
```  

接下来，更新你的 Twig 模板，添加由 Assetic 定义的 **{% javascripts %}** 标签：  

```
<!-- ... -->

    {% javascripts filter="?jsqueeze" output="js/app.js"
        "assets/js/jquery.js"
        "assets/js/bootstrap.js"
        "assets/js/main.js"
    %}
        <script src="{{ asset_url }}"></script>
    {% endjavascripts %}

    </body>
</html>
```  

这个简单的设置合并了所有的 JavaScript 文件，压缩了目录并且保存输出于 **web/js/app.js** 文件，这是唯一的一个提供给你的访问者的文件。  

**jsqueeze** 过滤器中的前导符 **？** 告诉 Assetic 在*非* **调试** 模式下只应用这个过滤器。在实践中，这就意味着你将会在 **prod** 环境下开发和压缩文件时看到没有压缩过的文件。