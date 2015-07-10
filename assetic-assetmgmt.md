# 如何使用 Assetic 进行资产管理 #

Assetic 结合了两种主要的观点:[资产](http://symfony.com/doc/current/cookbook/assetic/asset_management.html#cookbook-assetic-assets)和[过滤器](http://symfony.com/doc/current/cookbook/assetic/asset_management.html#cookbook-assetic-filters)。资产文件如CSS、JavaScript 和图像文件。过滤器是一种可以在文件被应用到浏览器之前就应用到文件上的东西。这样就会将存储在应用程序中的资产文件与呈献给用户的文件分开。  

没有 Assetic，你就需要直接提供存储在应用程序之中的文件：  

```Twig
<script src="{{ asset('js/script.js') }}"></script>
```  

```PHP
<script src="<?php echo $view['assets']->getUrl('js/script.js') ?>"></script>
```  

但是*有了* Assetic，你就可以随心所欲地操纵这些资产了（或者可以从任何地方加载他们）。这就意味着你能：  

- 压缩和合并你的所有的 CSS 和 JS 文件  
- 运行所有（或者部分）的你的 CSS 或 JS 文件通过一些类型的编译器，如 LESS, SASS 或者 CoffeeScript  
- 在你的图片上运行图片优化  

## 资产 ##

利用 Assetic 比直接提供文件拥有更多的优点。文件没必要储存在他们需要被提供的地方并且他们可以从各种各样的源中被取出来例如可以从束中取出来。  

你可以运用 Assetic 编辑 [CSS 模板](http://symfony.com/doc/current/cookbook/assetic/asset_management.html#cookbook-assetic-including-css), [JavaScript 文件](http://symfony.com/doc/current/cookbook/assetic/asset_management.html#cookbook-assetic-including-javascript)和[图片](http://symfony.com/doc/current/cookbook/assetic/asset_management.html#cookbook-assetic-including-image)。添加的原理基本上是相同的,但语法略有不同。  

### 包含 JavaScript 文件 ###

为了包含 JavaScript 文件，可以在任何模板中应用 **javascripts** 标签：  

```	Twig
{% javascripts '@AppBundle/Resources/public/js/*' %}
    <script src="{{ asset_url }}"></script>
{% endjavascripts %}
```  

```PHP
<?php foreach ($view['assetic']->javascripts(
    array('@AppBundle/Resources/public/js/*')
) as $url): ?>
    <script src="<?php echo $view->escape($url) ?>"></script>
<?php endforeach ?>
```  

>如果你的应用程序模板应用的是 Symfony 的标准版本的默认的区域名称，**javascripts** 标签将会出现在 **javascripts** 区域：  

>```
{# ... #}
{% block javascripts %}
    {% javascripts '@AppBundle/Resources/public/js/*' %}
        <script src="{{ asset_url }}"></script>
    {% endjavascripts %}
{% endblock %}
{# ... #}
>```  

>你也可以包含 CSS 模板：参见[包含 CSS 模板](http://symfony.com/doc/current/cookbook/assetic/asset_management.html#cookbook-assetic-including-css)。  

在本例中，所有的 AppBundle 的在 **Resources/public/js/** 目录下的文件将会被装载并且从不同的地点被提供。实际的渲染过的标签就像这样：  

```
<script src="/app_dev.php/js/abcd123.js"></script>
```  

这是一个关键点：一旦你允许 Assetic 来处理你的资产，文件就将会从不同的地方提供。这就会引起 CSS 文件的关联图片的相对路径问题。参见[用 cssrewrite 过滤器修复 CSS 路径](http://symfony.com/doc/current/cookbook/assetic/asset_management.html#cookbook-assetic-cssrewrite)。  

### 包含 CSS 模板 ###

为了引进 CSS 模板，你可以使用上面介绍的技术手段，除了使用 **stylesheets** 标签：  

```Twig
{% stylesheets 'bundles/app/css/*' filter='cssrewrite' %}
    <link rel="stylesheet" href="{{ asset_url }}" />
{% endstylesheets %}
```  

```PHP
<?php foreach ($view['assetic']->stylesheets(
    array('bundles/app/css/*'),
    array('cssrewrite')
) as $url): ?>
    <link rel="stylesheet" href="<?php echo $view->escape($url) ?>" />
<?php endforeach ?>
```  

>如果你的应用程序模板应用的是 Symfony 的标准版本的默认的区域名称,**stylesheets** 标签将会大多数出现在 **stylesheets** 区域：  

>```
{# ... #}
{% block stylesheets %}
    {% stylesheets 'bundles/app/css/*' filter='cssrewrite' %}
        <link rel="stylesheet" href="{{ asset_url }}" />
    {% endstylesheets %}
{% endblock %}
{# ... #}
>```  

但是由于 Assetic 改变了你的资产的路径，这将会打乱所有的使用相对路径的背景图片(或者其他路径)，除非你使用了 [cssrewrite](http://symfony.com/doc/current/cookbook/assetic/asset_management.html#cookbook-assetic-cssrewrite) 过滤器。  

>注意在最初的例子里包含的 JavaScript 文件，你可以应用像 **@AppBundle/Resources/public/file.js** 这样的路径来引用，但是在这个例子里，你要使用它们实际的公开的访问的路径：**bundles/app/css** 来引用 CSS 文件。你可以使用任意一个除了那个已知的当为 CSS 模板使用 **@AppBundle**语法时会引起 **cssrewrite** 过滤器出问题的除外。  

### 包括图片 ###

为了包括图片你可以使用 **image** 标签。  

```Twig
{% image '@AppBundle/Resources/public/images/example.jpg' %}
    <img src="{{ asset_url }}" alt="Example" />
{% endimage %}
```  

```PHP
<?php foreach ($view['assetic']->image(
    array('@AppBundle/Resources/public/images/example.jpg')
) as $url): ?>
    <img src="<?php echo $view->escape($url) ?>" alt="Example" />
<?php endforeach ?>
``` 

你也可以使用 Assetic 来优化图片。更多信息详见[如何使用 Assetic 和 Twig Functions 进行图像优化](http://symfony.com/doc/current/cookbook/assetic/jpeg_optimize.html)。

>代替使用 Assetic 来包括图片，你可以考虑使用 [LiipImagineBundle](https://github.com/liip/LiipImagineBundle) 团体束，这个允许压缩和操控图片（旋转，调整大小，加水印等等）。  

### 使用 cssrewrite 过滤器修复 CSS 路径 ###

由于 Assetic 为你的资产产生新的链接，你的 CSS 文件中的任何的相对路径都会被破坏。为了修复这个问题，确保你在 **stylesheets** 标签中使用了 **cssrewrite** 过滤器。这个解析了你的 CSS 文件并且在内部修正了其路径来反映新的位置。  

你可以在以前的章节看到一个例子。  

>当你使用 **cssrewrite** 过滤器时，不要用 **@AppBundle** 语法关联你的 CSS 文件。你可以在上面的章节的笔记里找到更详细的解释。  

### 组合资产 ###

Assetic 的一个特征就是它将会把很多文件组合成一个。这有助于减少 HTTP 请求的数量，这给前端的性能带来很多好处。它也会允许你更容易地维护文件，通过把他们分割成可管理的部分。这将有助于再利用，你可以很轻松地将特定的工程文件分离出来用于其他的应用程序，但是仍然将他们作为一个简单的文件提供：  

```Twig
{% javascripts
    '@AppBundle/Resources/public/js/*'
    '@AcmeBarBundle/Resources/public/js/form.js'
    '@AcmeBarBundle/Resources/public/js/calendar.js' %}
    <script src="{{ asset_url }}"></script>
{% endjavascripts %}
```  

```PHP
<?php foreach ($view['assetic']->javascripts(
    array(
        '@AppBundle/Resources/public/js/*',
        '@AcmeBarBundle/Resources/public/js/form.js',
        '@AcmeBarBundle/Resources/public/js/calendar.js',
    )
) as $url): ?>
    <script src="<?php echo $view->escape($url) ?>"></script>
<?php endforeach ?>
``` 

在 **dev** 环境下，每一个文件仍然单独存放，如此你就可以更容易地找出问题。然而，在 **prod** 环境下（或者特定地，在 **debug** 标记为 **false** 时），这个将会作为 **script** 标签渲染，这个标签包含了所有的 JavaScript 文件的目录。  

>如果你是刚开始学习 Assetic 并且试着在 **prod** 环境下（使用 **app.php** 控制器）应用你的应用程序，那么你将会很乐意看到你的所有的 CSS 和 JS 中断。不要担心！就是这么设计的。在 **prod** 环境下使用 Assetic 的相关细节知识可以参见[转储资产文件](http://symfony.com/doc/current/cookbook/assetic/asset_management.html#cookbook-assetic-dumping)。  

合并文件并不是只能应用于*你自己的*文件。你也可以使用 Assetic 来将第三方资产，比如 jQuery， 和你自己的文件合并为一个文件：  

```Twig
{% javascripts
    '@AppBundle/Resources/public/js/thirdparty/jquery.js'
    '@AppBundle/Resources/public/js/*' %}
    <script src="{{ asset_url }}"></script>
{% endjavascripts %}
```  

```PHP
<?php foreach ($view['assetic']->javascripts(
    array(
        '@AppBundle/Resources/public/js/thirdparty/jquery.js',
        '@AppBundle/Resources/public/js/*',
    )
) as $url): ?>
    <script src="<?php echo $view->escape($url) ?>"></script>
<?php endforeach ?>
```  

### 使用已命名的资产 ###

AsseticBundle 配置指令允许你定义已经命名的资产集。你可以在 **assetic** 节的设置中通过定义输入文件，过滤器和输出文件来进行配置。你可以在 [assetic 设置指南](http://symfony.com/doc/current/reference/configuration/assetic.html)中详细学习。  

```YAML
# app/config/config.yml
assetic:
    assets:
        jquery_and_ui:
            inputs:
                - '@AppBundle/Resources/public/js/thirdparty/jquery.js'
                - '@AppBundle/Resources/public/js/thirdparty/jquery.ui.js'
```  

```XML
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:assetic="http://symfony.com/schema/dic/assetic">

    <assetic:config>
        <assetic:asset name="jquery_and_ui">
            <assetic:input>@AppBundle/Resources/public/js/thirdparty/jquery.js</assetic:input>
            <assetic:input>@AppBundle/Resources/public/js/thirdparty/jquery.ui.js</assetic:input>
        </assetic:asset>
    </assetic:config>
</container>
```  

```PHP
// app/config/config.php
$container->loadFromExtension('assetic', array(
    'assets' => array(
        'jquery_and_ui' => array(
            'inputs' => array(
                '@AppBundle/Resources/public/js/thirdparty/jquery.js',
                '@AppBundle/Resources/public/js/thirdparty/jquery.ui.js',
            ),
        ),
    ),
);
```  

在你定义了已命名的资产之后，你可以在你的模板中引用他们通过 **@named_asset** 注释：

```Twig
{% javascripts
    '@jquery_and_ui'
    '@AppBundle/Resources/public/js/*' %}
    <script src="{{ asset_url }}"></script>
{% endjavascripts %}
```  

```PHP
<?php foreach ($view['assetic']->javascripts(
    array(
        '@jquery_and_ui',
        '@AppBundle/Resources/public/js/*',
    )
) as $url): ?>
    <script src="<?php echo $view->escape($url) ?>"></script>
<?php endforeach ?>
```  

## 过滤器 ##

一旦他们被 Assetic 所管理，你就可以在提供你的资产前应用过滤器。这包括了压缩你的资产使其输出更小的文件的过滤器（同时有好的前端性能）。其他的过滤器可以从 CoffeeScript 文件编译 JavaScript 文件并且将 SASS 变成 CSS。实际上，Assetic 有很多的可用的过滤器。  

大多数的过滤器不是直接工作的，但是使用现存的第三方函数库做最主要的处理。这就意味着你将经常需要安装第三方函数库来使用过滤器。使用 Assetic 调用这些函数库（而不是直接使用他们）的最大的优点就是在你调试完文件之后你不用人为地运行他们，Assetic 可以帮你打理这一切并且将这些步骤从你的开发部署过程中一起剔除。  

使用过滤器，你首先需要在 Assetic 设置中进行指定。在这里添加一个过滤器并不意味着它已经被使用——它只是意味着过滤器可以使用（你将会在以后使用）。  

举例来说，使用 UglifyJS JavaScript minifier 就需要做如下的定义：  

```YAML
# app/config/config.yml
assetic:
    filters:
        uglifyjs2:
            bin: /usr/local/bin/uglifyjs
```  

```XML
<!-- app/config/config.xml -->
<assetic:config>
    <assetic:filter
        name="uglifyjs2"
        bin="/usr/local/bin/uglifyjs" />
</assetic:config>
```  

```PHP
// app/config/config.php
$container->loadFromExtension('assetic', array(
    'filters' => array(
        'uglifyjs2' => array(
            'bin' => '/usr/local/bin/uglifyjs',
        ),
    ),
));
```  

现在，实际使用过滤器在一组 JavaScript 文件上，把它添加到你的模板上：  

```Twig
{% javascripts '@AppBundle/Resources/public/js/*' filter='uglifyjs2' %}
    <script src="{{ asset_url }}"></script>
{% endjavascripts %}
```  

```PHP
<?php foreach ($view['assetic']->javascripts(
    array('@AppBundle/Resources/public/js/*'),
    array('uglifyjs2')
) as $url): ?>
    <script src="<?php echo $view->escape($url) ?>"></script>
<?php endforeach ?>
```  

更详细地关于设置和使用 Assetic 过滤器同时也是  Assetic 的调试模式指导的资料详见[如何减小 CSS/JS 文件（应用 UglifyJS 和 UglifyCSS）](http://symfony.com/doc/current/cookbook/assetic/uglifyjs.html)  

## 控制已用的链接 ##

如果你想，那么你就可以控制 Assetic 产生的链接。这个是由模板做好的并且和公共文档的根相关：  

```Twig
{% javascripts '@AppBundle/Resources/public/js/*' output='js/compiled/main.js' %}
    <script src="{{ asset_url }}"></script>
{% endjavascripts %}
```  

```PHP
<?php foreach ($view['assetic']->javascripts(
    array('@AppBundle/Resources/public/js/*'),
    array(),
    array('output' => 'js/compiled/main.js')
) as $url): ?>
    <script src="<?php echo $view->escape($url) ?>"></script>
<?php endforeach ?>
```  

>Symfony 也包括了一种缓存*溢出*方法，这种方法中 Assetic 产生的最终的链接中包含了一个查询变量，这个变量可以通过在每个配置上设置来增加。获取更多信息，详见[资产版本](http://symfony.com/doc/current/reference/configuration/framework.html#ref-framework-assets-version)设置选择。  

## 转储资产文件 ##

在 **dev** 环境下，Assetic 为 CSS 和 JavaScript 文件产生的路径并不是真实存在于你的电脑上。但是尽管如此他们也渲染，因为内部的 Symfony 控制器打开文件并且提供内容（在运行任意过滤器之后）。  

这种动态的提供加工过的资产的方法非常好，因为这就意味着你可以立刻看到你所更改的任何资产文件的最新状态。这样做也有缺点，因为这样做可能会很慢。如果你用了很多过滤器，它可能彻底让人沮丧。  

幸运的是，Assetic 提供了一种方法来将你的资产转储成真正的文件，而不是动态的产生。  

### 在 prod 环境下转储资产文件 ###

在 **prod** 环境下，你的 JS 和 CSS 文件会被每个的简单的标签所代表。换句话说，替代看到每一个你包含在你的源中的 JavaScript 文件，你将会看到如下的东西：  

```
<script src="/js/abcd123.js"></script>
```  

除此之外，那个文件并**不是**真实存在，也不是由 Symfony 动态渲染的（由于资产文件在 **dev** 环境下）。这个是故意的——让 Symfony 产生这些动态的文件在生产环境太慢了。  

作为替代，每次你在 **prod** 环境下使用（并且因此，每次你部署）你的应用程序时，你需要运行如下的命令：  

```
$ php app/console assetic:dump --env=prod --no-debug
```  

这个将会物理的产生并且写入你需要的每一个文件（例如 **/js/abcd123.js**）。如果你更新你的任意的资产，你都需要再次运行这个命令来重新产生文件。  

### 在 dev 环境下转储资产文件 ###

在默认设置下，每一个资产在 **dev** 环境下的路径的产生是由 Symfony 进行动态处理的。这样做没有缺点（你可以迅速看见你的改动），除了资产可以明显的缓慢加载。如果你感觉你的资产装载的很慢，看看下面的指导。  

首先，让 Symfony 停止尝试动态地操作这些文件。在你的 **config_dev.yml** 文件中做如下改动：  

```YAML
# app/config/config_dev.yml
assetic:
    use_controller: false
```  

```XML
<!-- app/config/config_dev.xml -->
<assetic:config use-controller="false" />
```  

```PHP
// app/config/config_dev.php
$container->loadFromExtension('assetic', array(
    'use_controller' => false,
));
```  

接下来，由于 Symfony 不再为你产生这些资产，你将会需要手动转储这些文件。运行下列命令来完成这项工作：  

```
$ php app/console assetic:dump
```  

这样做写了所有的你的 **dev** 环境需要的资产文件。这个最大的缺点就是你需要每次更新资产文件的时候运行一次。幸运的是，通过采用 **assetic:watc** 命令，资产文件会自动重新生成*当他们改变的时候*：  

```
$ php app/console assetic:watch
```  

**assetic:watch** 命令在 AsseticBundle 2.4. 的测试版本中被引进，你必须用 **assetic:dump** 命令的 **--watch** 选项来达到相同效果。  

由于在 **dev** 环境下运行这个命令可能产生一束文件，这通常是一个好的想法来将你产生的资产文件指向一些隔离的区域（例如 **/js/compiled**），来使这些有组织：  

```Twig
{% javascripts '@AppBundle/Resources/public/js/*' output='js/compiled/main.js' %}
    <script src="{{ asset_url }}"></script>
{% endjavascripts %}
``` 

```PHP
<?php foreach ($view['assetic']->javascripts(
    array('@AppBundle/Resources/public/js/*'),
    array(),
    array('output' => 'js/compiled/main.js')
) as $url): ?>
    <script src="<?php echo $view->escape($url) ?>"></script>
<?php endforeach ?>
```  


