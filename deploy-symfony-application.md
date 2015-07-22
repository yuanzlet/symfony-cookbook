# 如何部署一个 Symfony 应用

> 部署的过程是一个复杂且多样的任务，它依靠于您应用的设置及需求的任务。本文不是按部就班的指导，而是一个关于部署大部分的需求和想法的大概的列表。

## Symfony 部署基础

在部署一个 Symfony 应用时采取的典型步骤包括：

1.	加载您的代码到成品服务器；

2.	安装您的供应商依赖（基本上通过 Composer 完成也可能是在加载前就完成）；

3.	运行数据库迁移或者相似的任务来更新任何发生变化的数据结构；

4.	清除（或者选择性地预热）缓存。

部署也可能包括其他任务，例如：

•	标记一个特别版本的代码作为您源代码控制存储库；

•	创建一个临时的集结待命区来创建您的更新设置“脱机”；

•	运行任何可用的测试来确保代码和/或者服务器的稳定性；

•	从 **web/** 目录中移除任何不必要文件来保持您的生产环境的洁净；

•	清除外部缓存系统（例如 [Memcached](http://memcached.org/) 或者 [Redis](http://memcached.org/)）

## 如何部署一个 Symfony 应用程序

部署 Symfony 应用程序有几种方式。首先从一些基本的部署策略开始然后由此开始创建。

### 基本文件转移

部署应用程序的最基本的方式是通过 ftp/scp （或相似的方法）手动复制文件。此方法也有弊端，因为您缺少在升级进度中对系统的控制。这个方法同样也需要您在转移文件后采取一些手动步骤（参见 [Common Post-Deployment Tasks](http://symfony.com/doc/current/cookbook/deployment/tools.html#common-post-deployment-tasks)）

### 使用源代码控件

如果您正只用源代码控件（例如 Git 或者 SVN），您可以通过安装现场储存库的副本来简化。当您准备去升级的话这非常简单，只需从您的源代码控制系统中获取最新的更新即可。

这会使您的文件更新更简单，但是您仍然需要担心手动采取其他步骤（参见 [Common Post-Deployment Tasks](http://symfony.com/doc/current/cookbook/deployment/tools.html#common-post-deployment-tasks)）。

### 使用构建脚本和其他工具

同样也有工具来减轻部署的负担。其中的一些明确符合 Symfony 的要求。

[Capifony](http://capifony.org/)

这个基于 Ruby 的工具在 [Capistrano](http://capistranorb.com/) 顶端提供了一整套专门的工具，专门针对 Symfony 项目定制。

[sf2debpkg](https://github.com/liip/sf2debpkg)

帮助您为您的 Symfony 项目创建一个本地 Debian 包装。

[Magallanes](https://github.com/andres-montane)

这个如同 Capotrano 的部署工具在 PHP 中创建，并且或许对于 PHP 开发者延伸他们的需要来说更简单一点。

[Fabric](http://www.fabfile.org/)

这个基于 Python 的库提供了一套操作的基本套件，来执行当地的或者远程的 shell 命令，还有加载/下载文件。

Bundles

有一些 [bundles 可以直接添加部署特点](http://knpbundles.com/search?q=deploy)到您的 Symfony 控制台上。

基本脚本

您当然可以使用 shell, [Ant](http://blog.sznapka.pl/deploying-symfony2-applications-with-ant/) 或者任何其他创建工具来编写您项目的部署脚本。

作为服务提供者的平台

Symfony Cookbook 包括一些最著名的服务（PaaS）提供商的平台的细节文章：

•	[Microsoft Azure](http://symfony.com/doc/current/cookbook/deployment/azure-website.html)

•	[Heroku](http://symfony.com/doc/current/cookbook/deployment/heroku.html)

•	[Platform.sh](http://symfony.com/doc/current/cookbook/deployment/platformsh.)

## 常见的后期部署任务

在部署了您实际的源代码之后，有许多您需要做的常规的事情：

### A) 检查要求

检查您的服务器是否在运行中符合要求：

```
$ php app/check.php

```

### B) 配置您的 app/config/parameters.yml 文件

这个文件不应被部署，而是通过由 Symfony 提供的自动工具管理。

### C) 安装/升级您的 Vendors

您的供应商可以在转移源代码之前进行升级（例如：升级 **vendor**/ 库，然后利用源代码进行转移）或者之后在服务器上升级。另一种方式就是，直接按平常的做法升级您的供应商。

```
$ composer install --no-dev --optimize-autoloader

```

> **--optimize-autoloader** 标记通过创建一个“类映射”显著地提高了 Composer 的自动装载机性能。**--no-dev** 标记确保开发软件包不安装在生产环境中。

> 如果您在此步骤中得到“未找到类”错误，您可能就需要在运行此命令前运行 **export SYMFONY_ENV=prod** 从而使 **post-install-cmd** 脚本在 **prod** 环境中运行。

### D) 清除您的 Symfony 缓存

确保您清除（并预热）您的 symfony 缓存：

```
$ php app/console cache:clear --env=prod --no-debug

```

### E) 清除您的 Assestic 资产

如果您正在使用 Assetic，您同样要清除您的资产：

```
$ php app/console assetic:dump --env=prod --no-debug

```

### F) 其他事情！

您需要做的事情还有很多，取决于您的设置：

•	运行任意数据库移植

•	清除您的 APC 缓存

•	运行 **assets:install** （已在 **composer install** 中处理）

•	添加/编辑 CRON 工作

•	推动资产到 CDN

•	…

## 应用程序生命周期：持续集成，QA，等等

虽然此条目涵盖了部署的技术细节，从开发到生产代码的完整生命周期或许需要更多的步骤（考虑部署到筹划，QA（质量保证），运行测试，等等）。

分段、测试、QA、持续集成、数据库合并以及回滚功能是防止失败而强烈建议使用的。有一些简单的以及更复杂的工具可以让部署像您环境所要求的那般简单（复杂）。

不要忘记部署应用程序同样包括更新任何依赖项（一般通过 Composer），合并您的数据库，清除您的缓存以及其他潜在的东西，如推动资产到 CDN 上（参见 [Common Post-Deployment Tasks](http://symfony.com/doc/current/cookbook/deployment/tools.html#common-post-deployment-tasks)）。
