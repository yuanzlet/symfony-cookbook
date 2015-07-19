# 如何在 Git 中创建并保存一个 Symfony 项目

> 虽然本文是基于 Git 讲述的，但是如果您想把项目存储于 Subversion 中，本文所讲的泛型原则同样适用。

当您通读了[用 Symfony 创建您的第一个页面](http://symfony.com/doc/current/book/page_creation.html)这篇文章，您将会对 Symfony 越来越熟悉。所以，理所当然这时候您可以创建您自己的项目了。在本章中，您将会学到一个最好的办法即通过使用 [Git](http://git-scm.com/) 源码控制管理系统去创建一个新的 Symfony 项目。

## 初始化项目设置

第一步，您需要下载 Symfony 框架并且让它运行起来。请仔细阅读 [安装并且配置 Symfony ](http://symfony.com/doc/current/book/installation.html) 这一章。

一旦您的程序运行起来，请遵循下面的这些简单的步骤：

1. 初始化 Git 仓库：

```
$ git init
```

2. 将所有的初始文件添加到 Git 中：

```
$ git add .
```

可能您已经注意到了，并不是所有的文件都在第一步中被 Composer 管理工具下载了，而是被 Git 暂时的提交了。就如同这个项目的依赖项（是由依赖管理工具 Composer 所管理），以及 **parameters.yml**  （其中包含了敏感信息，比如数据库的凭据） 一样，某些文件和文件夹不应该被提交到 Git。为了帮助您避免意外的提交这些文件和文件夹，有一个相应的包含一个名叫 **.gitignore**  文件的标准分布规则，在这个文件中包含了一个列表，这个列表中指明了哪些文件和文件夹不应该被提交。

> 您可能想要创建一个被整个系统使用的 .gitignore 文件。这样的话，就允许您排除一些通过您的 IDE 或者操作系统 创建的项目中的文件和文件夹。有关详细信息，请阅读 [GitHub .gitignore](https://help.github.com/articles/ignoring-files/)。

3. 给您新建的项目创建一个初始提交。

```
$ git commit -m "Initial commit"
```

此时，你有一个功能齐全并且能够正确提交到 Git 的 Symfony 项目，那么此时您就可以开始您的扩展了，把您新的改动提交到您的 Git 仓库。

您可以继续阅读[用 Symfony 创建您的第一个页面](http://symfony.com/doc/current/book/page_creation.html)这一章来了解更多的关于怎么在您的应用程序中配置和开发。 

> Symfony 标准版配备了一些示例功能。若要删除示例代码，请按照["如何到删除 AcmeDemoBundle"](http://symfony.com/doc/current/cookbook/bundles/remove.html) 中的说明。

## 带有 composer.json 的管理供应商库

### 它是怎么工作的？

每个 Symfony 项目中都有一系列的第三方提供的 “vendor” 库。其目标就是用一种或多种方法把这些文件下载到您的 **vendor/** 的目录，并且，在理想情况下，这些库会给您一些确切的方法去管理这些每个您需要的确切的版本。

默认情况下，这些库文件通过运行一个 **composer** 程序来安装二进制“下载程序”来下载文件。这些 **composer** 文件来自于一个叫做  [Composer](https://getcomposer.org/) 依赖管理工具，您可以通过阅读 [Installation](http://symfony.com/doc/current/book/installation.html#installation-updating-vendors) 来了解更多关于安装它的内容。

所执行的 **composer** 命令都是在您的项目根目录下的 **composer.json** 文件中读取的。这是一个 JSON 格式的文件，在这个文件中包含了您所需要的外部包名称和一些需要下载的版本信息以及其它更多信息的列表。**composer**  也从文件 **composer.lock** 读取信息，这个文件允许您把一个确切的版本连接任何一个您需要的库。事实上，如果存在 **composer.lock** 文件，那么这个文件中的版本信息就会附带 **composer.json**  中的信息，通过这样来更新您的库文件的版本，来允许 **composer** 更新。 

> 如果您想在您的程序中添加一个新的程序包，那么请运行下面的 **composer require** 命令：

```
$ composer require doctrine/doctrine-fixtures-bundle
```

如果想了解更多关于 Composer 管理工具的信息，请参阅 [GetComposer.org](https://getcomposer.org/)：

我们必须明白的是，这些供应商库实际上并不是您的仓库的一部分，相反，它们仅仅是一些被下载到 vendor/ 目录下的一些未监视的文件。但是因为下载这些文件所需要的所有的信息都保存在 composer.json 和 composer.lock（被存储在仓库中） 文件中，任何开发人员都可以使用这个项目，运行 **composer** 进行安装，并且下载这些确切相同的供应商库。这意味着您在不需要将供应商库提交到您的仓库的情况下，就能准确的了解这个库的作用，并且运用它。

所以，一个开发人员无论在什么时候使用您的项目，他们都应该先运行您的 **composer install ** 程序脚本来确保每个所需要的供应商库已经被下载了。

> ## Symfony 升级

> 由于 Symfony 是一系列的第三方库，并且这些第三方库可以通过 **composer.json** 和 **composer.lock** 文件来对 Symfony 库进行完全控制，升级 Symfony 就是指简单的升级这些文件，以使它们匹配最新的 Symfony 标准版本中的状态。

> 当然，如果您在 **composer.json** 中添加了新的条目，那么请确保只替换原来的那些部分(即不能同时删除任何您自定义的条目)。

## 在远程服务器上存储您的项目

你现在有一个存储在 Git 中的功能齐全的 Symfony 项目。然而，在大多数情况下，你会还想在远程服务器上存储您项目的备份以便其他开发人员可以在该项目进行协作。

在远程服务器上存储您的项目的最简单方法是通过基于 web 的托管服务，如 [GitHub](http://git-scm.com/) 或者  [Bitbucke](https://bitbucket.org/) 。当然，这里还有更多的服务，你可以在 [comparison of hosting services](https://en.wikipedia.org/wiki/Comparison_of_source_code_hosting_facilities) 进行研究和查找。

或者，您可以通过创建一个新的存储仓库系统把您的 Git 仓库存储到任何一个服务器中 ，然后将它送到服务器上。仓库能够帮助管理  [Gitolite](https://github.com/sitaramc/gitolite)。