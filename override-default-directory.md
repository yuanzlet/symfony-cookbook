# 如何重写 Symfony 默认的目录结构

Symfony 自动与默认目录结构建立联系。你可以轻松地重写这个目录来建立你自己的。默认目录结构如下：  

```
your-project/
├─ app/
│  ├─ cache/
│  ├─ config/
│  ├─ logs/
│  └─ ...
├─ src/
│  └─ ...
├─ vendor/
│  └─ ...
└─ web/
   ├─ app.php
   └─ ...
```  

## 重写缓存目录
你可以通过重写你的应用程序的 **AppKernel** 类中的 **getCacheDir** 方法来改变缓存的默认目录：  

```
// app/AppKernel.php

// ...
class AppKernel extends Kernel
{
    // ...

    public function getCacheDir()
    {
        return $this->rootDir.'/'.$this->environment.'/cache';
    }
}
```  

**$this->rootDir** 是 app 目录的绝对路径 **$this->environment**，其是现在的运行环境（换言之就是 **dev**）。在这种情况下你可以将缓存目录改为 **app/{environment}/cache**。  

> 你应当保持不同环境下的**缓存**目录不同，否则有些不好的行为可能发生。每一个环境都会产生缓存配置文件，所以每个环境都需要自己的目录来存储那些缓存文件。  

## 重写日志目录

重写**日志**目录和重写**缓存**目录一样。唯一的不同就是你需要重写 **getLogDir** 方法：  

```
// app/AppKernel.php

// ...
class AppKernel extends Kernel
{
    // ...

    public function getLogDir()
    {
        return $this->rootDir.'/'.$this->environment.'/logs';
    }
}
```  

这里你将目录改成了 **app/{environment}/logs**。  

## 重写网页目录

如果你需要重命名或者移动你的**网页**目录，你需要做的唯一的事情就是保证在你的 **app.php** 和 **app_dev.php** 前端控制器中通向 **app** 目录的路径可访问。如果你简单的重命名这些目录，就不会有问题。但是如果你将它以某种方式移动了，那么你需要在这些文件中修正这些路径：  

```
require_once __DIR__.'/../Symfony/app/bootstrap.php.cache';
require_once __DIR__.'/../Symfony/app/AppKernel.php';
```  

你也需要在 **composer.json** 文件中改变 **extra.symfony-web-dir** 选项：  

```
{
    ...
    "extra": {
        ...
        "symfony-web-dir": "my_new_web_dir"
    }
}
```  

> 一些共享的主机具有 **public_html** 网页目录的根。保持你的网页由 **web** 到 **public_html** 是一个使得你的 Symfony 工程在你的共享主机上工作的方法。另外一个把你的应用程序从你的网页根中分离的方法是，删除你的 **public_html** 目录，然后在你的工程中的链接到**网页**的 symbolic 链接替换它。

> 如果你使用 AsseticBundle，你需要设置 **read_from** 选项指向正确的**网页**目录：  

>```
># app/config/config.yml
 # ...
assetic:
    # ...
    read_from: "%kernel.root_dir%/../../public_html"
>```  

>```
><!-- app/config/config.xml -->
<!-- ... -->
<assetic:config read-from="%kernel.root_dir%/../../public_html" />
>```  

>```
>// app/config/config.php
 // ...
 $container->loadFromExtension('assetic', array(
     // ...
     'read_from' => '%kernel.root_dir%/../../public_html',
 ));
>```

> 现在你只需要清空缓存然后再一次转储资产然后你的应用程序就应该工作了：

>```
>$ php app/console cache:clear --env=prod
$ php app/console assetic:dump --env=prod --no-debug
>```  

## 重写供应商目录

为了重写**供应商**目录，你需要在 **app/autoload.php** 和 **composer.json** 文件中进行更改。  

在 **composer.json** 文件中进行更改如下所示：  

```
{
    ...
    "config": {
        "bin-dir": "bin",
        "vendor-dir": "/some/dir/vendor"
    },
    ...
}
```  

在 **app/autoload.php** 中，你需要修正 **vendor/autoload.php** 文件的路径：  

```
// app/autoload.php
// ...
$loader = require '/some/dir/vendor/autoload.php';
```  

> 如果你在虚拟环境下这个修正可能会很有意思并且不能使用 NFS。举例来说，如果你在客户操作系统中使用虚拟机运行 Symfony 应用程序。