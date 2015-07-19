# 如何在服务容器内设置外部参数

在[如何掌握并创建新的环境](http://symfony.com/doc/current/cookbook/configuration/environments.html)一章中，你学会了如何管理你的应用程序的配置。有些时候，在你的工程代码之外储存证书会对你的应用程序有好处。数据库配置就是这样一个例子。Symfony 的服务容器的灵活性使得你很容易这么做。  

## 环境变量

Symfony 将会抓取以 **SYMFONY__** 作为前缀的变量并且将其设置成为服务容器中的参数。结果的参数名称都应用了一些转换：  

- **SYMFONY__** 前缀被移除；
- 参数名称小写；
- 双下划线被替换成句号，由于句号在环境变量名称中不是有效的字符。  

举例来说，如果你使用 Apache，环境变量可以使用下面的 **VirtualHost** 配置进行设置：  

```
<VirtualHost *:80>
    ServerName      Symfony
    DocumentRoot    "/path/to/symfony_2_app/web"
    DirectoryIndex  index.php index.html
    SetEnv          SYMFONY__DATABASE__USER user
    SetEnv          SYMFONY__DATABASE__PASSWORD secret

    <Directory "/path/to/symfony_2_app/web">
        AllowOverride All
        Allow from All
    </Directory>
</VirtualHost>
```  

> 上述的例子是 Apache 中的设置，使用的是 [SetEnv](http://httpd.apache.org/docs/current/env.html) 指令。然而，这个将会为支持环境变量设置的任何网页服务器工作。  

> 同时，为了使你的控制台工作（那个不使用 Apache 的），你必须将这些作为 shell 变量输出。在 Unix 系统下，你可以运行如下代码：  

>```
>$ export SYMFONY__DATABASE__USER=user
$ export SYMFONY__DATABASE__PASSWORD=secret
>```  

既然你已经声明了环境变量，它将会在 PHP **$_SERVER** 全局变量中出现。之后 Symfony 将会自动将以 **SYMFONY__** 为前缀的所有 **$_SERVER** 变量设置成服务容器中的参数。  

现在你可以在你需要的时候随时使用这些参数。  

YAML:

```YAML
doctrine:
    dbal:
        driver    pdo_mysql
        dbname:   symfony_project
        user:     "%database.user%"
        password: "%database.password%"
```  

XML:

```XML
<!-- xmlns:doctrine="http://symfony.com/schema/dic/doctrine" -->
<!-- xsi:schemaLocation="http://symfony.com/schema/dic/doctrine http://symfony.com/schema/dic/doctrine/doctrine-1.0.xsd"> -->

<doctrine:config>
    <doctrine:dbal
        driver="pdo_mysql"
        dbname="symfony_project"
        user="%database.user%"
        password="%database.password%"
    />
</doctrine:config>
```  

PHP:

```PHP
$container->loadFromExtension('doctrine', array(
    'dbal' => array(
        'driver'   => 'pdo_mysql',
        'dbname'   => 'symfony_project',
        'user'     => '%database.user%',
        'password' => '%database.password%',
    )
));
```  

## 常量

容器也支持设置 PHP 常量作为参数。更多细节详见[常量作为参数](http://symfony.com/doc/current/components/dependency_injection/parameters.html#component-di-parameters-constants)。  

## 其它参数的设置

**imports** 指令可以用来将存储在各处的参数拉出来。输入 PHP 文件使得你在添加容器所需要的东西时更加的灵活。下面的例子就是输入一个名为 **parameters.php** 的文件。

YAML:

```YAML
# app/config/config.yml
imports:
    - { resource: parameters.php }
```  

XML：

```XML
<!-- app/config/config.xml -->
<imports>
    <import resource="parameters.php" />
</imports>
```  

PHP：

```PHP
// app/config/config.php
$loader->import('parameters.php');
```  

> 资源文件可以有很多类型。PHP, XML, YAML, INI，同时闭包资源都支持 **imports** 指令。

在 **parameters.php** 之中，告诉了服务容器你想要设置的参数。当重要的配置文件没有标准的格式时很有用。下面的例子包含了 Symfony 服务容器中的 Drupal 数据库配置。  

```
// app/config/parameters.php
include_once('/path/to/drupal/sites/default/settings.php');
$container->setParameter('drupal.database.url', $db_url);
``` 