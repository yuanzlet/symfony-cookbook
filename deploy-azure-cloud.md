# 部署在 Microsoft Azure 云

这个按部就班的教程描述了如何在 Microsoft Azure 云平台上部署一个小型的 Symfony 网页应用。它将会解释如何设置一个新的 Azure 网页，包括设置正确的 PHP 版本和全局环境变量。文件还展示了您如何可以利用 Git 和 Composer 在云上部署您的 Symfony 应用。

## 建立 Azure 网站

建立一个新的 Microsoft Azure 网站，首先[注册 Azure](https://signup.live.com/signup?uaid=5342819efab64b99bd8238efe0e39bcc&lic=1) 或者用证书注册。一旦您连接到了 [Azure Portal](https://login.microsoftonline.com/common/oauth2/authorize?response_type=code+id_token&redirect_uri=https%3a%2f%2fmanage.windowsazure.com%2f&client_id=00000013-0000-0000-c000-000000000000&resource=https%3a%2f%2fmanagement.core.windows.net%2f&scope=user_impersonation+openid&nonce=097f59b2-ccfc-46d7-8576-3f1e7542234e&domain_hint=&site_id=500879&response_mode=form_post) 的界面，向下滚动到按钮然后选择 **New** 面板。在此面板上，点击 **Web Site** 然后选择 **Custome Create**：

![image](images/step-01.png)

### 步骤 1：创建 Web Site

这里，系统将提示您填写一些基本信息。

![image](images/step-02.png)

对于 URL 来说，进入您将会用于 Symfony 应用程序的 URL，然后在您想要的区域内挑选 **Create new web hosting plan**。默认情况下，在下拉列表中的数据库中选择 *free 20 MB SQL datebase*。在本教程中，Symfony 应用程序将会连接到 MySQL 数据库上。在下拉列表的数据库中选择 **Create a new MySQL database**。您可以保持 **DefaultConnection** 的字符串名称。最后，确认会话框中的 **Publish from source control** 来启动 Git 库然后进入下一步。

### 步骤 2：新的 MySQL 数据库

在此步骤中，系统将会提示您建立您的 MySQL 数据库存储，有数据库名称和区域。MySQL 数据库存储由 Microsoft 和 ClearDB 联合提供。选择与您上一步为主机计划配置选择的相同的区域。

![image](images/step-03.png)

同意条款和条件后，点击向右箭头继续。

### 步骤3：您的源代码在哪里

现在，第三步，选择 **Local Git repository** 项目，然后点击右箭头来配置您的 Azure 网站证书。

![image](images/step-04.png)

### 步骤4：新的用户名与密码

太棒了！您现在在最后一步。创建一个用户名和一个安全密码：这些将成为连接到 FTP 服务器最根本的身份标识，并且也能推动您的应用程序代码到 Git 库。

![image](images/step-05.png)

祝贺！您的 Azure 网站现在已建立并正在运行了。您可以通过浏览您在第一步中配置的网站 url 来进行验证。您应该在您的网页浏览器看到以下显示：

![image](images/step-06.png)

Microsoft Azure 门户同样为 Azure Website 提供了一个完整的控制面板。

![image](images/step-07.png)

您的 Azure Website 已经做好准备！但是要运行一个 Symfony 网址，您需要配置一些其他的东西。

## 为 Symfony 配置 Azure Website

教程的这个部分详细地展示了怎样配置正确的 PHP 版本来运行 Symfony。它同样也展示了您怎样启动一些强制性的 PHP 扩张以及如何适当地为生产环境配置 PHP。

### 配置最新的 PHP 运行时间

尽管 Symfony 只需要 PHP 5.3.9 来运行，但还总是会建议使用最新的 PHP 版本。PHP 5.3 不再由 PHP 核心团队所支持，但是您可以在 Azure 里很容易地升级。

在 Azure 里升级您的 PHP 版本，在控制面板内找到 **Configure** 标记，然后选择您想要的版本。

![image](images/step-08.png)

在视窗下方点击 **Save** 按钮来保存您的变化，然后重启网页服务器。

> 选择一个较新一点的 PHP 版本可以极大地提高运行时性能。PHP 5.5 装载了一个新植入的 PHP 加速器称作 OPCache，替代了 APC。在 Azure Website 上，OPCache 已经被启动并且无需再安装和建立 APC 了。

> 以下的截屏显示了在 Azure Website 上运行的 [phpinfo](http://php.net/manual/en/function.phpinfo.php) 脚本的输出，来验证 PHP 5.5 是在启动了 OPCache 的情况下运行的。

> ![image](images/step-09.png)

### 对 php.ini 配置设置微调 

Microsoft Azure 允许您覆盖掉 **php.ini** 全局配置设置，通过在项目根目录（**site/wwwroot**）创建一个自定义文件 **.user.ini**。

```
; .user.ini
expose_php = Off
memory_limit = 256M
upload_max_filesize = 10M
```

这些设置均不需要覆盖。默认 PHP 配置已经足够好，所以只是一个例子来展示如何通过加载您的自定义文件 **.ini** 来轻松地微调 PHP 内部设置。

您也可以在您的 Azure Website 服务器上手动创建此文件，在 **site/wwwroot** 目录下或者用 Git 部署。您可以从 Azure Website Control 面板上获取您的 FTP 服务器证书，它在右侧侧边栏的 **Dashboard** 标记下。如果您想使用 Git，仅需要把您的 **.user.ini** 文件放置在您本地存储库的根目录下，然后在您的 Azure Website 库中启动提交。

> 本教程有一个部分专门解释如何配置您的 Azure Website Git 存储库以及如何启动部署提交。参见 [Deploying from Git](http://symfony.com/doc/current/cookbook/deployment/azure-website.html#deploying-from-git)。 您也可以在官方网页 [PHP MSDN documentation](http://blogs.msdn.com/b/silverlining/archive/2012/07/10/configuring-php-in-windows-azure-websites-with-user-ini-files.aspx) 获取更多关于配置 PHP 内部设置的信息。

### 启动 PHP intl 扩展

这是教程中很有趣的部分！在写这本教程的时候，Microsoft Azure Website 提供了 **intl** 扩展，但不是在默认情况下被启动。要启动 **intl** 扩展的话，无需加载任何 DLL 文件因为 **php_intl.dll** 文件已存在于 Azure 中。实际上，此文件只需要被移动至自定义网站的扩展目录。

> Microsoft Azure 团队如今致力于在默认情况下启动 **intl** PHP 扩展。在不久的将来，接下来的步骤架构不再是必要的了。

为了在 **site/wwwroot** 目录中获取 **php_intl.dll** 文件，只需要浏览以下网址就可以连接到在线 **Kudu** 工具：

```
https://[your-website-name].scm.azurewebsites.net
```

**Kudu** 是一套管理应用程序的工具。它带有一个文件资源管理器，一个命令行提示，一个日志流以及一个配置设置总结页面。当然，这个部分只有当您注册进入到您的 Azure Website 账号才可以访问。

![image](images/step-10.png)

在 Kudu 的头版，在主目录上点击 **Debug Console** 导航条目，然后选择 **CMD**。这将打开 **Debug Console** 页面，展示一个文件资源管理器和下方的控制台提示符。

在控制台提示符中，输入以下三个命令来复制原有的 **php_intl.dll** 扩展文件到自定义网站 **ext/** 目录。这个新的目录必须在主目录 **site/wwwroot** 下创建。

```
$ cd site\wwwroot
$ mkdir ext
$ copy "D:\Program Files (x86)\PHP\v5.5\ext\php_intl.dll" ext
```

整个过程和输出应该如此：

![image](images/step-11.png)

为了完成 **php_intl.dll** 扩展的启动，您必须让 Azure Website 从新创建的 **ext** 目录中加载。这个可以通过在 Azure Website Control 的主面板上的 **Configure** 标记上注册一个全局 **PHP_EXTENSIONS** 环境变量来完成。

在 **app settings** 部分，用值 **ext\php_intl.dll** 注册 **PHP_EXTENSIONS** 环境变量，如截屏所示：

![image](images/step-12.png)

点击 “save” 来确认您的变化并且重新启动网页服务器。PHP **Intl** 扩展应该在您的网页服务器环境是可用的了。接下来 [phpinfo](http://php.net/manual/en/function.phpinfo.php) 页面的截屏验证了 **intl** 扩展被正确启动。

![image](images/step-13.png)

太棒了！PHP 环境建立现已完成。接下来，您将学习如何配置 Git 存储库以及推动代码去产出。您将会学习到在部署之后如何安装和配置 Symfony 应用程序。

## 在 Git 中部署

首先，确保在您的终端使用以下命令使 Git 正确地安装在您本地机器中：

```
$ git --version
```

> 从 [git-scm.com](http://git-scm.com/download) 网站中获取您的 Git，并且按照指示在您本地机器上安装配置。

在 Azure Website Control 面板上，浏览 **Deployment** 标记来获取 Git 存储库 URL，即您应该启动代码的地方。

![image](images/step-14.png)

现在，您将要把您本地的 Symfony 应用程序与在 Azure Website 的远程 Git 存储库连接起来。如果您的 Symfony 应用程序还没有存储到库中，您必须首先在您的 Symfony 应用程序目录中用 **git init** 命令创建一个 GIt 存储库，然后用 **git commit** 命令提交。

同样，确保您的 Symfony 存储库有一个 **.gitignore** 文件在其根目录中，并至少含有一下内容：

```
/app/bootstrap.php.cache
/app/cache/*
/app/config/parameters.yml
/app/logs/*
!app/cache/.gitkeep
!app/logs/.gitkeep
/app/SymfonyRequirements.php
/build/
/vendor/
/bin/
/composer.phar
/web/app_dev.php
/web/bundles/
/web/config.php

```

**.gitignore** 文件要求 Git 不跟踪匹配这些模式的任何文件或目录。这意味着这些文件不被部署到 Azure Website 中。

现在，在您本地机器中的命令行中，在您的 Symfony 项目的根目录中输入以下：

```
$ git remote add azure https://<username>@<your-website-name>.scm.azurewebsites.net:443/<your-website-name>.git
$ git push azure master
```

不要忘记替换值，即在您的 Azure Website 面板上的 **Deployment** 展示的默认设置下由 **<** and **>** 封闭的值。**git remote** 命令连接了 Azure Website 远程 Git 存储库并分配其一个替换入口 **azure**。第二个 **git push** 命令启动所有对您的远程 **azure** Git 存储库的远程 **master** 分支的提交。

用 Git 部署应该产生与以下截屏相似的输出：

![image](images/step-15.png)

Symfony 应用程序的代码现在已被部署到 Azure Website，您可以在 Kudu 应用程序的文件资源管理器中浏览。您应该在 Azure Webiste 文件系统中您的 **site/wwwroot** 目录下看到目录  **app/**, **src/** 和 **web/**。

### 配置 Symfony 应用程序

PHP 已被配置，您的代码也已用 Git 启动。最后一步就是来配置应用程序并安装它所需要的第三方依赖，并不被 Git 追踪到。转回到在线 Kudu 应用程序的 **Console** 并在其中执行以下命令：

```
$ cd site\wwwroot
$ curl -sS https://getcomposer.org/installer | php
$ php -d extension=php_intl.dll composer.phar install
```

**curl** 命令检索并下载 Composer 命令行工具并在根目录 **site/wwwroot** 安装。然后，运行 Composer 命令下载并安装所有必要的三方库。

这样可能会花费一些时间，取决于您配置在 **composer.json** 文件中第三方依赖的数量。

> **-d** 开关允许您快速地重置或添加任何 **php.ini** 设置。在这个命令中，我们强制 PHP 使用 **intl** 扩展，因为此时此刻不是在 Azure Website 默认情况下启动的。不久，将不再需要 **-d** 选项因为 Microsoft 会在默认情况下启动 **intl** 扩展。

在 **composer install** 命令的结尾，系统将提示您填写一些 Symfony 设置的值，比如说数据库证书，区域设置，邮件程序证书，CSRF 保护盾牌等等。这些参数来自 **app/config/parameters.yml.dist** 文件。

![image](images/step-16.png)

本教程中最重要的事情是正确地建立您的数据库设置，您可以在 **Azure Website Dashboard** 面板的右侧边栏获取您的 MYSQL 数据库设置。简单地点击 **View Connection Strings** 链接来让它们突然出现。

![image](images/step-17.png)

所显示的 MySQL 数据库设置应该是和下面代码相似的一些东西。当然，每一个值取决于您的配置。

```
Database=mysymfonyMySQL;Data Source=eu-cdbr-azure-north-c.cloudapp.net;User Id=bff2481a5b6074;Password=bdf50b42
```

转换回控制台并回答提示的问题，并提供以下答案。不要忘记根据您在 MySQL 连接字符串里真实的值来调整以下的值。

```
database_driver: pdo_mysql
database_host: u-cdbr-azure-north-c.cloudapp.net
database_port: null
database_name: mysymfonyMySQL
database_user: bff2481a5b6074
database_password: bdf50b42
// ...
```

不要忘记回答所有的问题。为 **secret** 变量设置一个独立任意的字符串是很重要的。对于邮件程序配置来说，Azure Website 不提供一个内置的邮件程序服务。如果您的应用程序需要发送邮件，那么您应该考虑配置一些其他三方的邮件服务的主机名和证书。

![image](images/step-18.png)

您的 Symfony 应用程序现已配置完毕，应该几乎是可操作的了。最终的步骤就是建立数据库模式。如果您正在使用 Doctrine，那么用命令行界面很容易完成这个。Kudu 应用程序的在线 **Console** 工具中，运行以下命令将表格加载到您的 MySQL 数据库中。

```
$ php app/console doctrine:schema:update --force
```

这个命令为您的 MySQL 数据库建立表格和表单。如果您的 Symfony 应用程序比起基本的 Symfony 标准版本更复杂一些，您或许需要附加的命令来执行建立。（参见 [How to Deploy a Symfony Application](http://symfony.com/doc/current/cookbook/deployment/tools.html)）

确保您的应用程序是通过使用您的网页浏览器和以下网址浏览 **app.php** 前端控制器来运行。

```
http://<your-website-name>.azurewebsites.net/web/app.php
```

如果 Symfony 是正确安装的，您应该看一下您的 Symfony 应用程序显示的首页。

### 配置网页服务器

此刻，Symfony 应用程序已被部署并且在 Azure Website 上顺利工作。然而，**web** 文件夹仍然是网址的一部分，即您肯定不会想要的。但是不要担心！您可以轻松地配置网页服务器来指到 **web** 文件夹并移除 URL 中的 **web**（并确保没有人可以读取 **web** 目录中的外部文件。）

为了做这件事，创造并部署（参见关于 Git 前面的部分）以下 **web.config** 文件。这个文件必须位于 **composer.json** 文件旁边的根项目中。这个文件是 Microsoft IIS Server，它等同于 Apache 著名的 **.htaccess** 文件。对于一个 Symfony 应用程序来说，用以下内容配置：

```
<!-- web.config -->
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <rewrite>
      <rules>
        <clear />
        <rule name="BlockAccessToPublic" patternSyntax="Wildcard" stopProcessing="true">
          <match url="*" />
          <conditions logicalGrouping="MatchAll" trackAllCaptures="false">
            <add input="{URL}" pattern="/web/*" />
          </conditions>
          <action type="CustomResponse" statusCode="403" statusReason="Forbidden: Access is denied." statusDescription="You do not have permission to view this directory or page using the credentials that you supplied." />
        </rule>
        <rule name="RewriteAssetsToPublic" stopProcessing="true">
          <match url="^(.*)(\.css|\.js|\.jpg|\.png|\.gif)$" />
          <conditions logicalGrouping="MatchAll" trackAllCaptures="false">
          </conditions>
          <action type="Rewrite" url="web/{R:0}" />
        </rule>
        <rule name="RewriteRequestsToPublic" stopProcessing="true">
          <match url="^(.*)$" />
          <conditions logicalGrouping="MatchAll" trackAllCaptures="false">
          </conditions>
          <action type="Rewrite" url="web/app.php/{R:0}" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```

正如您可以看到的，最新的规则 **RewriteRequestsToPublic** 负责重新编写任何网址到 **web/app.php** 前端控制器，可以允许您跳过 URL 的 **web/** 文件夹。第一条规则叫做 **BlockAccessToPublic**，匹配所有的网址模式，包括 **web/** 文件夹并提供一个 **403 Forbidden** HTTP 响应。这个例子是基于 Benjamin Eberlei 的样例，您可以在 [SymfonyAzureEdition](https://github.com/icodeu/symfony-cookbook/blob/master/TOC.md) bundle 的 GitHub 中找到。

在 Azure Website 中的 **site/wwwroot** 目录中部署此文件并且在您的应用程序中浏览，不需要 URL 中的 **web/app.php** 片段。

## 总结

很好！您现在已经完成了在 Microsoft Azure Website 云平台上部署您的 Symfony 应用程序。您同样看到 Symfony 可以简单地在 Microsoft IIS 网页服务器上配置并执行。整个过程是简洁并易于实施的。作为奖励，Microsoft 将继续减少所需步骤的数量使部署变得更加简单。
