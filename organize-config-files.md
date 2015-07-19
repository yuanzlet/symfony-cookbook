
Symfony 的标准版本默认定义了三种[运行环境](http://symfony.com/doc/current/cookbook/configuration/environments.html)名为 **dev**, **prod** 和 **test**。环境简单的代表了用不同的配置执行相同的基础代码的方式。  

为了选择每个环境需要加载的配置文件，Symfony 执行 **AppKernel** 类的 **registerContainerConfiguration()** 方法：  

```
// app/AppKernel.php
use Symfony\Component\HttpKernel\Kernel;
use Symfony\Component\Config\Loader\LoaderInterface;

class AppKernel extends Kernel
{
    // ...

    public function registerContainerConfiguration(LoaderInterface $loader)
    {
        $loader->load($this->getRootDir().'/config/config_'.$this->getEnvironment().'.yml');
    }
}
```  

这个方法为 **dev** 环境加载 **app/config/config_dev.yml** 文件等等。反过来，这个文件加载位于 **app/config/config.yml** 的普通配置文件。因此，Symfony 的标准版本中的配置文件的结构如下所示：  

```
<your-project>/
├─ app/
│  └─ config/
│     ├─ config.yml
│     ├─ config_dev.yml
│     ├─ config_prod.yml
│     ├─ config_test.yml
│     ├─ parameters.yml
│     ├─ parameters.yml.dist
│     ├─ routing.yml
│     ├─ routing_dev.yml
│     └─ security.yml
├─ src/
├─ vendor/
└─ web/
```  

默认的结构就是为了它的简便而选择的——每个环境一个文件。但是由于 Symfony 的其它的特征，你可以将其设置成更加适合你需要的。下面一节将会介绍组织你的配置文件的不同方法。为了简化这个例子，只考虑 **dev** 和 **prod** 环境。  

## 每个环境下的不同的目录

代替在文件加 **_dev** 和 **_prod** 后缀，这个技术将所有相关的配置文件组织在和环境相同名称的目录之下：  

```
<your-project>/
├─ app/
│  └─ config/
│     ├─ common/
│     │  ├─ config.yml
│     │  ├─ parameters.yml
│     │  ├─ routing.yml
│     │  └─ security.yml
│     ├─ dev/
│     │  ├─ config.yml
│     │  ├─ parameters.yml
│     │  ├─ routing.yml
│     │  └─ security.yml
│     └─ prod/
│        ├─ config.yml
│        ├─ parameters.yml
│        ├─ routing.yml
│        └─ security.yml
├─ src/
├─ vendor/
└─ web/
```  

为了使这个起作用，改变 [registerContainerConfiguration()](http://api.symfony.com/2.7/Symfony/Component/HttpKernel/KernelInterface.html#registerContainerConfiguration()) 方法的代码：  

```
// app/AppKernel.php
use Symfony\Component\HttpKernel\Kernel;
use Symfony\Component\Config\Loader\LoaderInterface;

class AppKernel extends Kernel
{
    // ...

    public function registerContainerConfiguration(LoaderInterface $loader)
    {
        $loader->load($this->getRootDir().'/config/'.$this->getEnvironment().'/config.yml');
    }
}
```  

然后确保每一个 **config.yml** 文件都加载剩下的配置文件，包括普通文件。举例来说，这将是 **app/config/dev/config.yml** 文件的输入需要：  

YAML:

```YAML
# app/config/dev/config.yml
imports:
    - { resource: '../common/config.yml' }
    - { resource: 'parameters.yml' }
    - { resource: 'security.yml' }

# ...
```  

XML:

```XML
<!-- app/config/dev/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/symfony
        http://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

    <imports>
        <import resource="../common/config.xml" />
        <import resource="parameters.xml" />
        <import resource="security.xml" />
    </imports>

    <!-- ... -->
</container>
```  

PHP:

```PHP
// app/config/dev/config.php
$loader->import('../common/config.php');
$loader->import('parameters.php');
$loader->import('security.php');

// ...
``` 

> 由于参数解析的方式，你不能使用它们来动态建立输入路径。这也就意味着下列所示的一些将不起作用：  

>```YAML
># app/config/config.yml
imports:
    - { resource: "%kernel.root_dir%/parameters.yml" }
>```

>```XML
><!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">

    <imports>
        <import resource="%kernel.root_dir%/parameters.yml" />
    </imports>
</container>
>```

>```PHP
>// app/config/config.php
$loader->import('%kernel.root_dir%/parameters.yml');
>```  

## 语意性的配置文件

不同的组织策略可能需要应用程序有复杂的配置文件。举例来说，你可以每个 bundle 创建一个文件并且将几个文件定义为所有的应用程序服务：  

```
<your-project>/
├─ app/
│  └─ config/
│     ├─ bundles/
│     │  ├─ bundle1.yml
│     │  ├─ bundle2.yml
│     │  ├─ ...
│     │  └─ bundleN.yml
│     ├─ environments/
│     │  ├─ common.yml
│     │  ├─ dev.yml
│     │  └─ prod.yml
│     ├─ routing/
│     │  ├─ common.yml
│     │  ├─ dev.yml
│     │  └─ prod.yml
│     └─ services/
│        ├─ frontend.yml
│        ├─ backend.yml
│        ├─ ...
│        └─ security.yml
├─ src/
├─ vendor/
└─ web/
```  

除此之外，改变 **registerContainerConfiguration()** 方法的代码以确保 Symfony 知道新的文件组织方式：  

```
// app/AppKernel.php
use Symfony\Component\HttpKernel\Kernel;
use Symfony\Component\Config\Loader\LoaderInterface;

class AppKernel extends Kernel
{
    // ...

    public function registerContainerConfiguration(LoaderInterface $loader)
    {
        $loader->load($this->getRootDir().'/config/environments/'.$this->getEnvironment().'.yml');
    }
}
```  

顺着以前章节所说的相同的技术，确保从每一个主文件（common.yml, dev.yml 和 prod.yml）输入恰当的配置文件。  

## 高级技术

Symfony 使用 [Config 组件](http://symfony.com/doc/current/components/config/introduction.html)来加载配置文件，这提供了一些高级的特征。  

### 混合和匹配配置格式

配置文件可以输入使用内建配置格式（**.yml**, **.xml**, **.php**, **.ini**）定义的文件：  

YAML:

```YAML
# app/config/config.yml
imports:
    - { resource: 'parameters.yml' }
    - { resource: 'services.xml' }
    - { resource: 'security.yml' }
    - { resource: 'legacy.php' }

# ...
```  

XML:

```XML
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/symfony
        http://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

    <imports>
        <import resource="parameters.yml" />
        <import resource="services.xml" />
        <import resource="security.yml" />
        <import resource="legacy.php" />
    </imports>

    <!-- ... -->
</container>
```  

PHP:

```PHP
// app/config/config.php
$loader->import('parameters.yml');
$loader->import('services.xml');
$loader->import('security.yml');
$loader->import('legacy.php');

// ...
```  

> **IniFileLoader** 解释了使用 [parse_ini_file](http://php.net/manual/en/function.parse-ini-file.php) 功能的文件内容。因此，你只能将参数设置成字符串的值。如果你想要使用其它的数据类型（例如布尔型，整型等等）那么请使用其它的加载器。  

如果你使用其它的配置格式，你必须自己定义你的加载器类将它从 [FileLoader](http://api.symfony.com/2.7/Symfony/Component/DependencyInjection/Loader/FileLoader.html) 扩展。当配置的值是动态时，你可以使用 PHP 配置来执行你自己的逻辑。除此之外，你可以定义你自己的服务来从数据库或者网页服务器加载配置。  

### 全局配置文件

一些系统管理员可能更喜欢将敏感的参数储存在工程目录之外的文件中。可以想象你网页的数据库正数储存在 **/etc/sites/mysite.com/parameters.yml** 文件中。当你从其它的配置文件中输入时，加载这个文件就好像指出整个文件路径一样简单：  

YAML:

```YAML
# app/config/config.yml
imports:
    - { resource: 'parameters.yml' }
    - { resource: '/etc/sites/mysite.com/parameters.yml' }

# ...
```  

XML:

```XML
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/symfony
        http://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

    <imports>
        <import resource="parameters.yml" />
        <import resource="/etc/sites/mysite.com/parameters.yml" />
    </imports>

    <!-- ... -->
</container>
```  

PHP:

```PHP
// app/config/config.php
$loader->import('parameters.yml');
$loader->import('/etc/sites/mysite.com/parameters.yml');

// ...
```  

大多数的时候，本地开发者不会在开发服务器上存储相同文件。由于这个原因，Config 组件提供了 **ignore_errors** 选项来在加载文件不存在时悄悄忽视错误：  

YAML:

```YAML
# app/config/config.yml
imports:
    - { resource: 'parameters.yml' }
    - { resource: '/etc/sites/mysite.com/parameters.yml', ignore_errors: true }

# ...
```  

XML:

```XML
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/symfony
        http://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

    <imports>
        <import resource="parameters.yml" />
        <import resource="/etc/sites/mysite.com/parameters.yml" ignore-errors="true" />
    </imports>

    <!-- ... -->
</container>
```  

PHP:

```PHP
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd
        http://symfony.com/schema/dic/symfony
        http://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

    <imports>
        <import resource="parameters.yml" />
        <import resource="/etc/sites/mysite.com/parameters.yml" ignore-errors="true" />
    </imports>

    <!-- ... -->
</container>
```  

正如你所看到的那样，有很多组织你的配置文件的方法。你可以选择其中一种或者你甚至也可以创建你自己风格的组织文件的方式。不要被由 Symfony 产生的 Symfony 标准版本所限制。更多的个性化信息，参见“[如何重写 Symfony 的默认目录结构](http://symfony.com/doc/current/cookbook/configuration/override_dir_structure.html)”。