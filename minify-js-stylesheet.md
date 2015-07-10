# 如何使用 YUI Compressor 裁剪 Javascripts 和 Stylesheets

>YUI Compressor [不再归属于雅虎](http://www.yuiblog.com/blog/2013/01/24/yui-compressor-has-a-new-owner/)。这就是为什么**强烈建议你避免使用 YUI 实用程序**除非特别必要。阅读[如何裁剪 CSS/JS 文件(使用 UglifyJS 和 UglifyCSS)](http://symfony.com/doc/current/cookbook/assetic/uglifyjs.html)来寻求一种更新的现代的的替代品。  

雅虎提供了一款优秀的压缩 JavaScripts 和 stylesheets 的实用工具这样他们就可以在线路中传输的更快了，这就是 [YUI Compressor](http://developer.yahoo.com/yui/compressor/)。多亏了 Assetic，你可以很好地很容易的利用这个工具。  

## 下载 YUI Compressor JAR ##

YUI Compressor 是由 Java 编写的并且以 JAR 文件格式发布。从雅虎的网站[下载 JAR 文件](https://github.com/yui/yuicompressor/releases)将它保存到 **app/Resources/java/yuicompressor.jar**。  

## 设置 YUI Filters ##

现在你需要在你的应用程序中设置两个 Assetic 过滤器，一个用来使用 YUI Compressor 来压缩 JavaScripts，另一个用来压缩 stylesheets：  

```YAML
# app/config/config.yml
assetic:
    # java: "/usr/bin/java"
    filters:
        yui_css:
            jar: "%kernel.root_dir%/Resources/java/yuicompressor.jar"
        yui_js:
            jar: "%kernel.root_dir%/Resources/java/yuicompressor.jar"
```  

```XML
<!-- app/config/config.xml -->
<assetic:config>
    <assetic:filter
        name="yui_css"
        jar="%kernel.root_dir%/Resources/java/yuicompressor.jar" />
    <assetic:filter
        name="yui_js"
        jar="%kernel.root_dir%/Resources/java/yuicompressor.jar" />
</assetic:config>
```  

```PHP
// app/config/config.php
$container->loadFromExtension('assetic', array(
    // 'java' => '/usr/bin/java',
    'filters' => array(
        'yui_css' => array(
            'jar' => '%kernel.root_dir%/Resources/java/yuicompressor.jar',
        ),
        'yui_js' => array(
            'jar' => '%kernel.root_dir%/Resources/java/yuicompressor.jar',
        ),
    ),
));
```  

>Windows 用户需要记住升级设置来适应 Java 位置。在 Windows 7 64 位系统中默认目录是 **C:\Program Files (x86)\Java\jre6\bin\java.exe**。  

现在你可以在你的应用程序中访问两个新的 Assetic 过滤器：**yui_css** 和 **yui_js**。这些将会使用 YUI Compressor 来分别压缩 stylesheets 和 JavaScripts。  

## 压缩你的资产 ##

现在你已经设置了 YUI Compressor，但是在你在你的资产上应用这些过滤器之前什么都不会发生。由于你的资产是视图层的一部分，这个工作已经在你的模板中做完了：  

```Twig
{% javascripts '@AppBundle/Resources/public/js/*' filter='yui_js' %}
    <script src="{{ asset_url }}"></script>
{% endjavascripts %}
```  

```PHP
<?php foreach ($view['assetic']->javascripts(
    array('@AppBundle/Resources/public/js/*'),
    array('yui_js')
) as $url): ?>
    <script src="<?php echo $view->escape($url) ?>"></script>
<?php endforeach ?>
```  

>上述例子假设你已经有了名为 AppBundle 的 bundle 并且你的 JavaScript 文件在你的 bundle 下的 **Resources/public/js** 目录下。尽管这并不是很重要——你可以包括 JavaScript 文件无论他们在哪。  

在为上述的 asset 标签增加了 **yui_js** 过滤器之后，你应该可以看到压缩过的 JavaScript 文件在线路中传输的更快了。可以采用相同的过程来压缩你的 stylesheets。  

```Twig
{% stylesheets '@AppBundle/Resources/public/css/*' filter='yui_css' %}
    <link rel="stylesheet" type="text/css" media="screen" href="{{ asset_url }}" />
{% endstylesheets %}
```  

```PHP
<?php foreach ($view['assetic']->stylesheets(
    array('@AppBundle/Resources/public/css/*'),
    array('yui_css')
) as $url): ?>
    <link rel="stylesheet" type="text/css" media="screen" href="<?php echo $view->escape($url) ?>" />
<?php endforeach ?>
```  

## 在调试模式下禁用压缩 ##

压缩过的 JavaScripts 和 stylesheets 文件很难被读取，更不必说是调试模式。因为这个，当你的应用程序在调试模式的时候资产需要你禁用一些过滤器。你可以通过预先添加问号：**？**修改你的模板中的过滤器的名称来达到这个目的。这就可以告诉 Assetic 只有在调试模式关闭的时候才开启这些过滤器。  

```Twig
{% javascripts '@AppBundle/Resources/public/js/*' filter='?yui_js' %}
    <script src="{{ asset_url }}"></script>
{% endjavascripts %}
```  

```PHP
<?php foreach ($view['assetic']->javascripts(
    array('@AppBundle/Resources/public/js/*'),
    array('?yui_js')
) as $url): ?>
    <script src="<?php echo $view->escape($url) ?>"></script>
<?php endforeach ?>
```  

>为了替代在 asset 标签添加过滤器，你也可以通过添加过滤器设置中的 **apply_to** 来通通禁用，举例来说在 **yui_js** 过滤器的 **apply_to: "\.js$"**。在产品中仅仅使用过滤器，添加这个到 **config_prod** 文件而不是通用设置文件。更多详细的通过文件扩展应用过滤器，参见[基于文件扩展的过滤器应用](http://symfony.com/doc/current/cookbook/assetic/apply_to_option.html#cookbook-assetic-apply-to)。  

 

