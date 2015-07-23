# 如何注入变量到所有的模板(如全局变量)

有时您想要一个对您所用的所有模板都可用的变量。这在您的 **app/config/config.yml** 文件夹里是可行的。

```YAML
# app/config/config.yml
twig:
    # ...
    globals:
        ga_tracking: UA-xxxxx-x
```

```XML
<!-- app/config/config.xml -->
<twig:config>
    <!-- ... -->
    <twig:global key="ga_tracking">UA-xxxxx-x</twig:global>
</twig:config> 
```

```PHP
// app/config/config.php
$container->loadFromExtension('twig', array(
     // ...
     'globals' => array(
         'ga_tracking' => 'UA-xxxxx-x',
     ),
));
```

现在，变量 **ga_tracking** 在所有的 Twig 模板里都是可用的了。

```
<p>The google tracking code is: {{ ga_tracking }}</p>
```

就是这么简单！

## 使用服务容器参数

您还可以利用内置的服务参数系统，它可以让您隔离或重用该值：

```
# app/config/parameters.yml
parameters:
    ga_tracking: UA-xxxxx-x
```

```YAML
# app/config/config.yml
twig:
    globals:
        ga_tracking: "%ga_tracking%"
```

```XML
<!-- app/config/config.xml -->
<twig:config>
    <twig:global key="ga_tracking">%ga_tracking%</twig:global>
</twig:config>
```

```PHP
// app/config/config.php
$container->loadFromExtension('twig', array(
     'globals' => array(
         'ga_tracking' => '%ga_tracking%',
     ),
));
```

同一个变量还是像以前那样能用。

## 引用服务

除了使用静态值，您还可以将该值设置为服务。当在模板中访问全局变量时，将从服务容器中请求服务，并访问该对象。

> 服务不会延迟加载。换句话说，当 Twig 被加载时，即使您从来没有使用全局变量，您的服务也会被实例化。

要将服务定义为全局 Twig 变量，以 **@** 为前缀的字符串。这应该是熟悉的，因为它是在服务配置中使用的相同语法。

```YAML
# app/config/config.yml
twig:
    # ...
    globals:
        user_management: "@acme_user.user_management"
```

```XML
<!-- app/config/config.xml -->
<twig:config>
    <!-- ... -->
    <twig:global key="user_management">@acme_user.user_management</twig:global>
</twig:config>
```

```PHP
// app/config/config.php
$container->loadFromExtension('twig', array(
     // ...
     'globals' => array(
         'user_management' => '@acme_user.user_management',
     ),
))
```

## 使用 Twig 扩充

如果全局变量要设置更为复杂的话-说一个对象-，那么您就不能用上面的方法了。替代上面的方法，您需要创建一个 ![Twig 扩充](http://symfony.com/doc/current/reference/dic_tags.html#reference-dic-tags-twig-extension)并且在 **getglobals** 方法返回一个全局变量条目。
