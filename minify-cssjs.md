# 如何裁剪 CSS/JS 文件(使用 UglifyJS 和 UglifyCSS)

[UglifyJS](https://github.com/mishoo/UglifyJS) 是一个 JavaScript 的集合了 parser/compressor/beautifier 的工具包。它可以用来合并压缩 JavaScript 资产从而减少 HTTP 的请求的数量并且加快你的网站的加载速度。[UglifyCSS](https://github.com/fmarcia/UglifyCSS) 是一个和 UglifyJS 十分相似的 CSS compressor/beautifier。  

在本指导中，UglifyJS 的安装，配置以及使用都进行了详细的讲解。UglifyCSS 的工作原理和 UglifyJS 很相似所以只是在这里进行简单介绍。  

## 安装 UglifyJS ##

UglifyJS 是作为 [Node.js](http://nodejs.org/) 的模块使用。首先，你需要[安装 Node.js](http://nodejs.org/) 然后决定安装方式：全局或者局部。  

### 全局安装 ###

全局的安装方法可以使得你的所有工程都使用相同的 UglifyJS 版本，这大大简化了维护环节。打开你的命令控制台执行下列命令（你可能需要以超级用户和身份运行）：  

```
$ npm install -g uglify-js
```  

现在你可以在你的系统的任何地方执行全局 **uglifyjs** 命令：  

```
$ uglifyjs --help
```  

### 局部安装 ###

只在你的工程中安装 UglifyJS 也是可以的，当你的工程需要特定的版本的时候这就会很有用。为了完成这个，不带 **-g** 选项安装并且制定放置模块的路径：  

```
$ cd /path/to/your/symfony/project
$ npm install uglify-js --prefix app/Resources
```  

我们建议你将 UglifyJS 安装在你的 **app/Resources** 文件夹并且添加 **node_modules** 来进行版本控制。或者，你也可以创建一个 npm [package.json](http://package.json.nodejitsu.com/) 文件并且在那里声明你的依赖性。  

现在你可以执行位于 **node_modules** 目录下的 **uglifyjs** 命令：  

```
$ "./app/Resources/node_modules/.bin/uglifyjs" --help
```  

## 设置 uglifyjs2 过滤器 ##

现在你需要设置 Symfony 在编辑 JavaScripts 时来使用 **uglifyjs2** 过滤器：  

```YAML
# app/config/config.yml
assetic:
    filters:
        uglifyjs2:
            # the path to the uglifyjs executable
            bin: /usr/local/bin/uglifyjs
```  

```XML
<!-- app/config/config.xml -->
<assetic:config>
    <!-- bin: the path to the uglifyjs executable -->
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
            // the path to the uglifyjs executable
            'bin' => '/usr/local/bin/uglifyjs',
        ),
    ),
));
```  

>UglifyJS 安装的路径可能取决于你的系统。为例找出 **bin** 文件夹中的 npm 的储存位置。可以执行下列命令：  

>```
$ npm bin -g
>```  

>这会在你的系统中输出一个文件夹，在这个文件夹中你能够找到可执行的 UglifyJS。

>如果你是局部安装 UglifyJS，你可以在 **node_modules** 文件夹中的 **bin** 文件夹中找到。在这里它叫做 **.bin**。

现在你就拥有了在你的应用程序中访问 **uglifyjs2** 过滤器的权限。  

## 设置 node Binary ##

Assetic 会尝试自动寻找 node Binary。如果它找不到，你可以使用 **node** 键对它的位置进行设置：  

```YAML
# app/config/config.yml
assetic:
    # the path to the node executable
    node: /usr/bin/nodejs
    filters:
        uglifyjs2:
            # the path to the uglifyjs executable
            bin: /usr/local/bin/uglifyjs
```  

```XML
<!-- app/config/config.xml -->
<assetic:config
    node="/usr/bin/nodejs" >
    <assetic:filter
        name="uglifyjs2"
        bin="/usr/local/bin/uglifyjs" />
</assetic:config>
```  

```PHP
// app/config/config.php
$container->loadFromExtension('assetic', array(
    'node' => '/usr/bin/nodejs',
    'uglifyjs2' => array(
            // the path to the uglifyjs executable
            'bin' => '/usr/local/bin/uglifyjs',
        ),
));
```  

## 压缩你的资产 ##

为了在你的资产上应用 UglifyJS，在你的模板的资产标签上添加 **filter** 选项告诉 Assetic 来使用 **uglifyjs2** 过滤器：  

```Twig
{% javascripts '@AppBundle/Resources/public/js/*' filter='uglifyjs2' %}
    <script src="{{ asset_url }}"></script>
{% endjavascripts %}
```  

```PHP
<?php foreach ($view['assetic']->javascripts(
    array('@AppBundle/Resources/public/js/*'),
    array('uglifyj2s')
) as $url): ?>
    <script src="<?php echo $view->escape($url) ?>"></script>
<?php endforeach ?>
```  

>上述的例子假设你拥有一个名为 AppBundle 的 bundle 并且你的 JavaScript 文件位于你的 bundle 的 **Resources/public/js** 目录下。然而你可以包含 JavaScript 文件无论他们在哪。  

随着以上在你的资产标签上添加 **uglifyjs2** 过滤器，现在你将会看到压缩过的 JavaScripts 在线路中传输的更快了。  

### 在调试模式下禁用压缩 ###

压缩过的 JavaScripts 很难被读取，更别说是调试模式了。正是因为这个，Assetic 要求你在你的应用程序处于调试（例如 **app_dev.php**）模式时禁用一些特定的过滤器。你可以通过预先添加问号：**？**修改你的模板中的过滤器的名称来达到这个目的。这就可以告诉 Assetic 只有在调试模式关闭的时候才开启这些过滤器（例如 **app.php**）。  

```Twig
{% javascripts '@AppBundle/Resources/public/js/*' filter='?uglifyjs2' %}
    <script src="{{ asset_url }}"></script>
{% endjavascripts %}
```  

```PHP
<?php foreach ($view['assetic']->javascripts(
    array('@AppBundle/Resources/public/js/*'),
    array('?uglifyjs2')
) as $url): ?>
    <script src="<?php echo $view->escape($url) ?>"></script>
<?php endforeach ?>
```  

为了做出这个，要切换到你的 **prod** 环境（**app.php**）。但是你切换之前，不要忘了[清空你的缓存](http://symfony.com/doc/current/book/configuration.html#book-page-creation-prod-cache-clear)[并且转储你的 assetic 资产](http://symfony.com/doc/current/cookbook/assetic/asset_management.html#cookbook-assetic-dump-prod)。  

>为了替代在资产标签添加过滤器，你也可以在应用程序配置文件中设置你的应用程序的每一个文件所使用的过滤器。更多信息参见[基于文件扩展的过滤器应用](http://symfony.com/doc/current/cookbook/assetic/apply_to_option.html#cookbook-assetic-apply-to)。

## 安装，设置和使用 UglifyCSS ##

UglifyCSS 的使用和 UglifyJS 是一样的。首先，确保节点包已经安装：  

```
# global installation
$ npm install -g uglifycss

# local installation
$ cd /path/to/your/symfony/project
$ npm install uglifycss --prefix app/Resources
```  

接下来，在这个过滤器中添加设置：  

```YAML
# app/config/config.yml
assetic:
    filters:
        uglifycss:
            bin: /usr/local/bin/uglifycss
```  

```XML
<!-- app/config/config.xml -->
<assetic:config>
    <assetic:filter
        name="uglifycss"
        bin="/usr/local/bin/uglifycss" />
</assetic:config>
```  

```PHP
// app/config/config.php
$container->loadFromExtension('assetic', array(
    'filters' => array(
        'uglifycss' => array(
            'bin' => '/usr/local/bin/uglifycss',
        ),
    ),
));
```  

在你的 CSS 文件上应用这个过滤器，需要在 Assetic 的 **stylesheets** 助手上添加这个过滤器：  

```Twig
{% stylesheets 'bundles/App/css/*' filter='uglifycss' filter='cssrewrite' %}
     <link rel="stylesheet" href="{{ asset_url }}" />
{% endstylesheets %}
```  

```PHP
<?php foreach ($view['assetic']->stylesheets(
    array('bundles/App/css/*'),
    array('uglifycss'),
    array('cssrewrite')
) as $url): ?>
    <link rel="stylesheet" href="<?php echo $view->escape($url) ?>" />
<?php endforeach ?>
```  

就好像 **uglifyjs2** 过滤器一样，如果你事先在过滤器的名称前加了**？**（例如 **?uglifycss**），压缩将只在非调试模式下进行。 
