# 如何在 SubVersion 中创建并保存一个 Symfony 项目

> 本章是基于 Subversion 进行阐述的，并且所讲到的规则是建立在  [如何在 Git 中创建并保存一个 Symfony 项目](http://symfony.com/doc/current/cookbook/workflow/new_project_git.html) 一章中所讲内容的基础上。

一旦当您通读了[用 Symfony 创建您的第一个页面](http://symfony.com/doc/current/book/page_creation.html)这篇文章，您将会对 Symfony 越来越熟悉。所以，理所当然这时候您可以创建您自己的项目了。管理 Symfony 项目最常用的方法就是使用 [Git](http://git-scm.com/)，但是如果有些人喜欢使用 [Subversion](http://subversion.apache.org/) 也是完全没有问题的，就像在 [Git](http://git-scm.com/) 中管理您的项目一样，在本章中，您将会学习到如何使用 [SVN](http://subversion.apache.org/) 去管理您的项目。

> 这是一个在 Subversion 仓库中监视您的 Symfony 项目的众多方法中的一个简单有效的方法。

## Subversion 仓库

在本章中，我们假定您的存储仓库布局遵循普遍的标准结构：

```
myproject/
    branches/
    tags/
    trunk/
```

> 大多数 Subversion 托管都遵循这个标准的做法。这是在 [Subversion 版本控制系统](http://svnbook.red-bean.com/)一文中推荐的布局，并且这个对于大多数免费托管平台来说，这个布局都是最常用的(请参见 [Subversion 托管方案  ](http://symfony.com/doc/current/cookbook/workflow/new_project_svn.html#svn-hosting)）。

## 初始化项目设置

第一步，您需要下载 Symfony 框架并且让它运行起来。请仔细阅读 [安装并且配置 Symfony ](http://symfony.com/doc/current/book/installation.html) 这一章。

一旦您的程序运行起来，请遵循下面的这些简单的步骤：

1\. 检查将要托管这个项目的 Subversion 仓库，假定它名为 **myproject** 并且被托管在 [Google code ](https://code.google.com/hosting/) 平台：

```
$ svn checkout http://myproject.googlecode.com/svn/trunk myproject
```

2\. 在 Subversion 文件夹中拷贝 Symfony 项目：

```
$ mv Symfony/* myproject/
```

3\. 现在，设置某些文件的忽略规则。并不是所有文件都应该被存储在你的 Subversion 仓库中，比如某些生成的文件（如缓存）或者每台计算机中自定义的文件（如数据库的配置信息）。我们利用 **svn:ignore** 属性实现上述功能，这样的话，某些特定的文件就可以被忽略。

```
$ cd myproject/
$ svn add --depth=empty app app/cache app/logs app/config web

$ svn propset svn:ignore "vendor" .
$ svn propset svn:ignore "bootstrap*" app/
$ svn propset svn:ignore "parameters.yml" app/config/
$ svn propset svn:ignore "*" app/cache/
$ svn propset svn:ignore "*" app/logs/

$ svn propset svn:ignore "bundles" web

$ svn ci -m "commit basic Symfony ignore list (vendor, app/bootstrap*, app/config/parameters.yml, app/cache/*, app/logs/*, web/bundles)"
```

4\. 现在，这些剩下的文件就可以被添加并且提交到项目中了：

```
$ svn add --force .
$ svn ci -m "add basic Symfony Standard 2.X.Y"
```

就像这样！因为 **app/config/parameters.yml** 文件将被忽略，所以您可以存储您计算机的特定设置比如您数据库密码，但是又可以不用提交它们。虽然文件 **parameters.yml.dist** 被提交了，但是并不是由 Symfony 进行读取的。新的开发人员可以通过使用您对您项目文件添加的秘钥来克隆您的项目，并且把该文件拷贝到  **parameters.yml** 文件中，进行自定义，然后就可以开始开发了。

此时，在你 Subversion 仓库中有一个功能齐全的 Symfony 项目，并且可以开始把您项目的扩展内容提交到 Subversion repository 仓库中了。

您可以继续阅读[用 Symfony 创建您的第一个页面](http://symfony.com/doc/current/book/page_creation.html)这一章来了解更多的关于怎么在您的应用程序中配置和开发。

> Symfony 标准版配备了一些示例功能。若要删除示例代码，请按照[如何到删除 AcmeDemoBundle](http://symfony.com/doc/current/cookbook/bundles/remove.html) 中的说明。

## 带有 composer.json 的管理供应商库

### 它是怎么工作的？

每个 Symfony 项目中都有一系列的第三方提供的 “vendor” 库。其目标就是用一种或多种方法把这些文件下载到您的 **vendor/** 的目录，并且，在理想情况下，这些库会给您一些确切的方法去管理这些每个您需要的版本。

默认情况下，这些库文件通过运行一个 **composer** 程序来安装二进制“下载程序”来下载文件。这些 **composer** 文件来自于一个叫做 [Composer](https://getcomposer.org/) 依赖管理工具，您可以通过阅读 [Installation](http://symfony.com/doc/current/book/installation.html#installation-updating-vendors) 来了解更多关于安装它的内容。

所执行的 **composer** 命令都是在您的项目根目录下的 **composer.json** 文件中读取的。这是一个 JSON 格式的文件，在这个文件中包含了您所需要的外部包名称和一些需要下载的版本信息以及其它更多信息的列表。**composer**  也从文件 **composer.lock** 读取信息，这个文件允许您把一个确切的版本连接任何一个您需要的库。事实上，如果存在 **composer.lock** 文件，那么这个文件中的版本信息就会附带 **composer.json** 中的信息，通过这样来更新您的库文件的版本，来允许 **composer** 更新。 

> 如果您想在您的程序中添加一个新的程序包，那么请运行下面的 **composer require** 命令：

```
$ composer require doctrine/doctrine-fixtures-bundle
```

如果想了解更多关于 Composer 管理工具的信息，请参阅 [GetComposer.org](https://getcomposer.org/)：

我们必须明白的是，这些供应商库实际上并不是您的仓库的一部分，相反，它们仅仅是一些被下载到 vendor/ 目录下的一些未监视的文件。但是因为下载这些文件所需要的所有的信息都保存在 composer.json 和 composer.lock（被存储在仓库中） 文件中，任何开发人员都可以使用这个项目，运行 **composer** 进行安装，并且下载这些确切相同的供应商库。这意味着您在不需要将供应商库提交到您的仓库的情况下，就能准确的了解这个库的作用，并且运用它。

所以，一个开发人员无论在什么时候使用您的项目，他们都应该先运行您的 **composer install ** 程序脚本来确保每个所需要的供应商库已经被下载了。

> ## Symfony 升级

> 由于 Symfony 是一系列的第三方库，并且这些第三方库可以通过 **composer.json** 和 **composer.lock** 文件来对 Symfony 库进行完全控制，升级 Symfony 就是指简单的升级这些文件，以使它们匹配最新的 Symfony 标准版本中的状态。

> 当然，如果您在 **composer.json** 中添加了新的条目，那么请确保只替换原来的那些部分(即不能同时删除任何您自定义的条目)

## Subversion 托管解决方案

[Git](http://git-scm.com/) 和 [Git](http://git-scm.com/) 的最大的区别就是 Subversion 需要在一个中央仓库上工作，您现在有以下几种解决方案：

- 自主托管：创建您自己的仓库，并且通过文件系统或者网络来访问它。如果您想获得更多资料来帮助您理解，请参阅 [Subversion 托管方案](http://symfony.com/doc/current/cookbook/workflow/new_project_svn.html#svn-hosting)

- 第三方托管：目前有很多很好但是免费可用的托管方案，就比如 [GitHub](https://github.com/)，[Google code ](https://code.google.com/hosting/)，SourceForge[SourceForge](http://sourceforge.net/) 和  [Gna](http://gna.org/) 等，它们之中的部分平台也提供 Git 托管服务。