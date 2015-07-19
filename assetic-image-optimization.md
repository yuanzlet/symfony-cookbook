# 如何使用 Assetic 和 Twig Functions 进行图像优化

在众多的过滤器之中，Assetic 拥有四个可以用作动态的图像优化的过滤器。这就让你可以在没有应用图片编辑器以及没有编辑每一张图片的情况下就可以享受小尺寸图片的好处。这个结果可以是缓存也可以进行产出转储，所以你的最终用户也没有备份。  

## 使用 Jpegoptim

Jpegoptim 是一款优化 JPEG 格式文件的实用工具。在 Assetic 下应用它，首先要确保它安装在你的系统中，然后使用 **jpegoptim** 过滤器中的 **bin** 选项设置它的位置：  

YAML:

```YAML
# app/config/config.yml
assetic:
    filters:
        jpegoptim:
            bin: path/to/jpegoptim
```  

XML:

```XML
<!-- app/config/config.xml -->
<assetic:config>
    <assetic:filter
        name="jpegoptim"
        bin="path/to/jpegoptim" />
</assetic:config>
```  

PHP:

```PHP
// app/config/config.php
$container->loadFromExtension('assetic', array(
    'filters' => array(
        'jpegoptim' => array(
            'bin' => 'path/to/jpegoptim',
        ),
    ),
));
```  

它现在可以从模板中应用：  

```
{% image '@AppBundle/Resources/public/images/example.jpg'
    filter='jpegoptim' output='/images/example.jpg' %}
    <img src="{{ asset_url }}" alt="Example"/>
{% endimage %}
```  

```
<?php foreach ($view['assetic']->image(
    array('@AppBundle/Resources/public/images/example.jpg'),
    array('jpegoptim')
) as $url): ?>
    <img src="<?php echo $view->escape($url) ?>" alt="Example"/>
<?php endforeach ?>
```  

## 移除所有 EXIF 数据

默认设置下，**jpegoptim** 过滤器移除了一些存储在图片中的元信息。为了移除所有的 EXIF 数据和评论，将 **strip_all** 选项设置为 **true**：  

YAML:

```YAML
# app/config/config.yml
assetic:
    filters:
        jpegoptim:
            bin: path/to/jpegoptim
            strip_all: true
```  

XML:

```XML
<!-- app/config/config.xml -->
<assetic:config>
    <assetic:filter
        name="jpegoptim"
        bin="path/to/jpegoptim"
        strip_all="true" />
</assetic:config>
```  

PHP:

```PHP
// app/config/config.php
$container->loadFromExtension('assetic', array(
    'filters' => array(
        'jpegoptim' => array(
            'bin'       => 'path/to/jpegoptim',
            'strip_all' => 'true',
        ),
    ),
));
```  

### 降低最大的质量

默认设置下，**jpegoptim** 过滤器不会在 JPEG 图片的品质层面进行选择。使用 max 选项来设置最大的质量值（在 **0-100** 这个区间）。图像尺寸的减小必然是以降低它的质量为代价的：  

YAML:

```YAML
# app/config/config.yml
assetic:
    filters:
        jpegoptim:
            bin: path/to/jpegoptim
            max: 70
```  

XML:

```XML
<!-- app/config/config.xml -->
<assetic:config>
    <assetic:filter
        name="jpegoptim"
        bin="path/to/jpegoptim"
        max="70" />
</assetic:config>
```  

PHP:

```PHP
// app/config/config.php
$container->loadFromExtension('assetic', array(
    'filters' => array(
        'jpegoptim' => array(
            'bin' => 'path/to/jpegoptim',
            'max' => '70',
        ),
    ),
));
```  

## 缩短语法：Twig Function

如果你正在用 Twig，通过应用一个 Twig 的特殊功能你很可能完成这些实现一个更短的语法。从添加下列设置开始：  

YAML:

```YAML
# app/config/config.yml
assetic:
    filters:
        jpegoptim:
            bin: path/to/jpegoptim
    twig:
        functions:
            jpegoptim: ~
```  

XML:

```XML
<!-- app/config/config.xml -->
<assetic:config>
    <assetic:filter
        name="jpegoptim"
        bin="path/to/jpegoptim" />
    <assetic:twig>
        <assetic:twig_function
            name="jpegoptim" />
    </assetic:twig>
</assetic:config>
```  

PHP:

```PHP
// app/config/config.php
$container->loadFromExtension('assetic', array(
    'filters' => array(
        'jpegoptim' => array(
            'bin' => 'path/to/jpegoptim',
        ),
    ),
    'twig' => array(
        'functions' => array('jpegoptim'),
        ),
    ),
));
```  

Twig 的模板现在被修改成了以下这样：  

```
<img src="{{ jpegoptim('@AppBundle/Resources/public/images/example.jpg') }}" alt="Example"/>
```  

你也可以在 Assetic 的设置文件中指定图片的输出目录：  

YAML:

```YAML
# app/config/config.yml
assetic:
    filters:
        jpegoptim:
            bin: path/to/jpegoptim
    twig:
        functions:
            jpegoptim: { output: images/*.jpg }
```  

XML:

```XML
<!-- app/config/config.xml -->
<assetic:config>
    <assetic:filter
        name="jpegoptim"
        bin="path/to/jpegoptim" />
    <assetic:twig>
        <assetic:twig_function
            name="jpegoptim"
            output="images/*.jpg" />
    </assetic:twig>
</assetic:config>
```  

PHP:

```PHP
// app/config/config.php
$container->loadFromExtension('assetic', array(
    'filters' => array(
        'jpegoptim' => array(
            'bin' => 'path/to/jpegoptim',
        ),
    ),
    'twig' => array(
        'functions' => array(
            'jpegoptim' => array(
                output => 'images/*.jpg'
            ),
        ),
    ),
));
```  

>上传图片，你可以使用 [LiipImagineBundle](http://knpbundles.com/liip/LiipImagineBundle) community bundle 来压缩和操作它们。