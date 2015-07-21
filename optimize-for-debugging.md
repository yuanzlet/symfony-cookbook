# 如何将你的开发环境优化为调试环境

当您在本地机器上运行一个 Symfony 工程时，您应该使用 **dev** 环境（**app dev .php** 前端控制器）。优化此配置主要有两个目的：

•	在任何时候有不当情况发生（网页调试工具栏，nice 异常页面，分析器…）时，给开发者正确的反馈；

•	尽可能与生产环境相似来避免在部署项目过程中的问题。

## 禁用引导文件和类缓存

并且，为了让生产环境尽可能的快，Symfony 在您的缓存中创建大型的 PHP 文件，包含您项目所需的每个请求的 PHP 类的聚集。然而，此行为可以混淆您的 IDE 或者您的调试器。此教程显示了当您需要调试含有 Symfony 类的代码时如何微调此缓存机制让它更加友好。

**app_dev.php** 前端控制器在默认情况下按照以下读出：

```
/ ...

$loader = require_once __DIR__.'/../app/bootstrap.php.cache';
require_once __DIR__.'/../app/AppKernel.php';

$kernel = new AppKernel('dev', true);
$kernel->loadClassCache();
$request = Request::createFromGlobals();
```

为了您的调试器更配合一点儿，移除 **loadClassCache()** 的请求从而禁用所有的 PHP 类缓存，并按照以下进行替换所需的语句：

```
// ...

// $loader = require_once __DIR__.'/../app/bootstrap.php.cache';
$loader = require_once __DIR__.'/../app/autoload.php';
require_once __DIR__.'/../app/AppKernel.php';

$kernel = new AppKernel('dev', true);
// $kernel->loadClassCache();
$request = Request::createFromGlobals();
```

如果您禁用 PHP 缓存，在您调试会话后不要忘记复原。

一些 IDEs 不喜欢一些类储存在不同的地点。为了避免问题，您可以选择让您的 IDE 忽略 PHP 缓存文件，或者您可以改变 Symfony 为这些文件使用的扩展。

```
1	$kernel->loadClassCache('classes', '.php.cache');

```





