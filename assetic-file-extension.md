# 如何将 Assetic Filter 应用到具体的文件扩展名上

Assetic 的过滤器可以应用到独立的文件上，甚至一组文件，就像你看到的这样，文件都有特定的扩展名。为了向你展示如何处理每一个选项，假设你想使用 Assetic 的 CoffeeScript 过滤器，这个过滤器将 CoffeeScript 文件编译成 JavaScript 文件。  

主要的设置就是 **coffee, node** 和 **node_modules** 的路径。设置的例子如下所示：  

```YAML
# app/config/config.yml
assetic:
    filters:
        coffee:
            bin:        /usr/bin/coffee
            node:       /usr/bin/node
            node_paths: [/usr/lib/node_modules/]
```  

```XML
<!-- app/config/config.xml -->
<assetic:config>
    <assetic:filter
        name="coffee"
        bin="/usr/bin/coffee/"
        node="/usr/bin/node/">
        <assetic:node-path>/usr/lib/node_modules/</assetic:node-path>
    </assetic:filter>
</assetic:config>
```  

```PHP
// app/config/config.php
$container->loadFromExtension('assetic', array(
    'filters' => array(
        'coffee' => array(
            'bin'  => '/usr/bin/coffee',
            'node' => '/usr/bin/node',
            'node_paths' => array('/usr/lib/node_modules/'),
        ),
    ),
));
```  

## 过滤一个简单的文件 ##

现在你可以建立一个简单的 CoffeeScript 作为  JavaScript 从你的模板中：  

```Twig
{% javascripts '@AppBundle/Resources/public/js/example.coffee' filter='coffee' %}
    <script src="{{ asset_url }}"></script>
{% endjavascripts %}
```  

```PHP
<?php foreach ($view['assetic']->javascripts(
    array('@AppBundle/Resources/public/js/example.coffee'),
    array('coffee')
) as $url): ?>
    <script src="<?php echo $view->escape($url) ?>"></script>
<?php endforeach ?>
```  

这些都需要编译 CoffeeScript 文件并且作为编译过的 JavaScript 提供。  

## 过滤多个文件 ##

你也可以将多个 CoffeeScript 文件合并成一个单一的输出文件：  

```Twig
{% javascripts '@AppBundle/Resources/public/js/example.coffee'
               '@AppBundle/Resources/public/js/another.coffee'
    filter='coffee' %}
    <script src="{{ asset_url }}"></script>
{% endjavascripts %}
```  

```PHP
<?php foreach ($view['assetic']->javascripts(
    array(
        '@AppBundle/Resources/public/js/example.coffee',
        '@AppBundle/Resources/public/js/another.coffee',
    ),
    array('coffee')
) as $url): ?>
    <script src="<?php echo $view->escape($url) ?>"></script>
<?php endforeach ?>
```  

两个文件现在都被合并成一个文件编译成普通的 JavaScript。  

## 基于文件扩展名的过滤 ##

使用 Assetic 来减少资产文件数量的一个优点就是降低了 HTTP 请求的数量。为了更好地应用这一点，将你的*所有的* JavaScript 和 CoffeeScript 文件合并在一起会更好，因为他们将会最终作为 JavaScript 提供。不幸的是仅仅将 JavaScript 添加到将被合并的文件中将不会像正常的一样 JavaScript 文件将不会和 CoffeeScript 编译。  

这个问题可以通过使用 **apply_to** 选项避免，这个选项可以允许你确定哪一个过滤器应当总是被应用于特定的文件扩展名。这样你可以指定 **coffee** 过滤器是应用于所有的 **.coffee** 文件：  

```YAML
# app/config/config.yml
assetic:
    filters:
        coffee:
            bin:        /usr/bin/coffee
            node:       /usr/bin/node
            node_paths: [/usr/lib/node_modules/]
            apply_to:   "\.coffee$"
```  

```XML
<!-- app/config/config.xml -->
<assetic:config>
    <assetic:filter
        name="coffee"
        bin="/usr/bin/coffee"
        node="/usr/bin/node"
        apply_to="\.coffee$" />
        <assetic:node-paths>/usr/lib/node_modules/</assetic:node-path>
</assetic:config>
```  

```PHP
// app/config/config.php
$container->loadFromExtension('assetic', array(
    'filters' => array(
        'coffee' => array(
            'bin'      => '/usr/bin/coffee',
            'node'     => '/usr/bin/node',
            'node_paths' => array('/usr/lib/node_modules/'),
            'apply_to' => '\.coffee$',
        ),
    ),
));
```  

使用这个选项，你不在需要在模板中指定 **coffee** 过滤器。你也可以列出普通的 JavaScript 文件，所有这些都会被合并并且渲染成一个简单的 JavaScript 文件（伴随着 **.coffee** 文件仅仅通过 CoffeeScript 过滤器运行）：  

```Twig
{% javascripts '@AppBundle/Resources/public/js/example.coffee'
               '@AppBundle/Resources/public/js/another.coffee'
               '@AppBundle/Resources/public/js/regular.js' %}
    <script src="{{ asset_url }}"></script>
{% endjavascripts %}
```  

```PHP
<?php foreach ($view['assetic']->javascripts(
    array(
        '@AppBundle/Resources/public/js/example.coffee',
        '@AppBundle/Resources/public/js/another.coffee',
        '@AppBundle/Resources/public/js/regular.js',
    )
) as $url): ?>
    <script src="<?php echo $view->escape($url) ?>"></script>
<?php endforeach ?>
```  

