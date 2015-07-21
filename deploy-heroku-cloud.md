# 部署在 Heroku 云

这本按部就班的教程描述了如何在 Heroku 云平台上部署一个 Symfony 网页应用程序。其内容基于在 Heroku 上出版的[原创文章](https://devcenter.heroku.com/articles/getting-started-with-symfony2)。

## 设置

创建一个新的 Heroku 网站，首先用 [Heroku 注册](https://signup.heroku.com/dc)或者用您自己的证书注册。然后在您的本地计算机上下载并安装 [Heroku Toolbelt](https://devcenter.heroku.com/articles/getting-started-with-php#introduction)。

您也可以查看[在 Heroku 上开始使用 PHP](https://devcenter.heroku.com/articles/getting-started-with-php#introduction) 的指导来获取更多的对于在 Heroku 上使用 PHP 应用程序的细节的熟悉度。

### 准备您的应用程序

在 Heroku 上部署一个 Symfony 应用程序不需要其代码的任何变化，但是需要对其配置的一些轻微调整。

默认情况下，Symfony 应用程序会登录进入您应用程序的 **app/log/** 目录。这不是很理想的因为 Heroku 使用的是[短暂的文件系统](https://devcenter.heroku.com/articles/dynos#ephemeral-filesystem)。在 Heroku 中，处理登录的最好方法是使用 [Logplex](https://devcenter.heroku.com/articles/logplex)。并且发送登录数据到 Logplex 最好的方式是通过编写 **STDERR** 或者 **STDOUT**。幸运的是，Symfony 使用的是非常好的 Monolog 库来登录。因此，一个新的日志目的就是仅仅改变一个配置文件。

打开 **app/config/config_prod.yml** 文件，把 **monolog/handlers/nested** 片段（若还未存在就创建一个）并改变 **path** 的值，从 **"%kernel.logs_dir%/%kernel.environment%.log"** 到 **"php://stderr"**：

```
# app/config/config_prod.yml
monolog:
    # ...
    handlers:
        # ...
        nested:
            # ...
            path: "php://stderr"
```

一旦应用程序被部署，运行 **heroku logs –tail** 使 Heroku 中的日志流在您的终端保持开启状态。

## 在 Heroku 中创建一个新的应用程序

创建一个您可以启动的新的 Heroku 应用程序，使用 CLI **create** 命令：

```
$ heroku create

Creating mighty-hamlet-1981 in organization heroku... done, stack is cedar
http://mighty-hamlet-1981.herokuapp.com/ | git@heroku.com:mighty-hamlet-1981.git
Git remote heroku added
```

现在您已经准备好部署应用程序，如同在下一个部分解释的那样。

## 在 Heroku 上部署您的应用程序

在您的首次部署前，您仅需要再做三件事，解释如下：

1.[创建一个 Procfile](http://symfony.com/doc/current/cookbook/deployment/heroku.html#heroku-procfile)

2.[设置环境 prod](http://symfony.com/doc/current/cookbook/deployment/heroku.html#heroku-setting-env-to-prod)

3.[启动 Heroku 的代码](http://symfony.com/doc/current/cookbook/deployment/heroku.html#heroku-push-code)

### 1)创建一个 Procfile

默认情况下，Heroku 会开启一个 Apache 网页服务器和 PHP 一并来服务应用程序。然而，有两个特殊情况适用于 Symfony 应用程序：

1.	文件根是在 **web/** 目录中，而不是在用应用程序的根目录中。  

2.	Composer **bin-dir**，即供应商的二进制文件（Heroku 自身的引导脚本）放置的地方，是 **bin/**，不是默认的 **vendor/bin**。

供应商的二进制文件一般由 Composer 安装在 **vendor/bin**，但是有些时候（例如：当运行一个 Symfony 标准版本项目！），定位是不同的。如果有疑问的话，您可以运行 **composer config bin-dir** 来找出正确的位置。

在应用程序的根目录创建一个新的文件叫做 **Procfile**（没有任何扩展），并且仅添加以下内容：

```
b: bin/heroku-php-apache2 web/
```

如果您更喜欢使用 Nginx，在 Heroku 同样是可以使用的，您可以创建一个配置文件或者按照在 [Heroku documentation](https://devcenter.heroku.com/articles/custom-php-settings#nginx) 描述的那样从 Procfile 指向它：

```
web: bin/heroku-php-nginx -C nginx_app.conf web/
```

如果您更喜欢使用命令控制台工作，执行以下命令来创建 **Procfile** 文件，并且将其添加到存储库中：

```
$ echo "web: bin/heroku-php-apache2 web/" > Procfile
$ git add .
$ git commit -m "Procfile for Apache and PHP"
[master 35075db] Procfile for Apache and PHP
 1 file changed, 1 insertion(+)
```

### 2) 设置环境 prod

在部署中，Heroku 运行 **composer install --no-dev** 来安装应用程序所需要的所有依赖。然而，一般在 **composer.json** [安装后命令](https://getcomposer.org/doc/articles/scripts.md)，例如：安装资产或者清除（或预热）缓存，在默认情况下运行使用 Symfony 的 **dev** 环境。

这明显不是您想要的—应用程序在“生成”中运行（尽管您只是用它来做个试验，或者作为一个过渡环境），所以任何构建的步骤也应该使用同样的 **prod** 环境。

幸好这个问题的解决方案是很简单的：Symfony 将会选择一个环境变量称 **SYMFONY_ENV** 并且如果没有其他特别的设置就会使用此环境。当 Heroku 公开所有[配置变量](https://devcenter.heroku.com/articles/config-vars)作为环境变量，您可以发出一个单独命令来为您的应用程序做部署的准备：

```
$ heroku config:set SYMFONY_ENV=prod
```

注意在 **require-dev** 区段 **composer.json** 列出的依赖在 Heroku 中部署的过程中从不被安装。如果您的 Symfony 环境依赖这些包的话可能会引发问题。解决方案就是从 **require-dev** 移除这些包到 **require** 区段。

### 3)启动 Heroku 的代码

下一步，终于到了在 Heroku 部署您的应用程序的时间。如果您是第一次做这个的话，您将会看到像如下的信息：

```
The authenticity of host 'heroku.com (50.19.85.132)' can't be established.
RSA key fingerprint is 8b:48:5e:67:0e:c9:16:47:32:f2:87:0c:1f:c8:60:ad.
Are you sure you want to continue connecting (yes/no)?
```

这个情况下，您需要通过输入 **yes** 并点击 **<Enter>** 按键来确认—理想的情况是您[验证了 RSA 秘钥指纹是正确的](https://devcenter.heroku.com/articles/git-repository-ssh-fingerprints)。

然后，执行这个命令部署您的应用程序：

```
$ git push heroku master

Initializing repository, done.
Counting objects: 130, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (107/107), done.
Writing objects: 100% (130/130), 70.88 KiB | 0 bytes/s, done.
Total 130 (delta 17), reused 0 (delta 0)

-----> PHP app detected

-----> Setting up runtime environment...
       - PHP 5.5.12
       - Apache 2.4.9
       - Nginx 1.4.6

-----> Installing PHP extensions:
       - opcache (automatic; bundled, using 'ext-opcache.ini')

-----> Installing dependencies...
       Composer version 64ac32fca9e64eb38e50abfadc6eb6f2d0470039 2014-05-24 20:57:50
       Loading composer repositories with package information
       Installing dependencies from lock file
         - ...

       Generating optimized autoload files
       Creating the "app/config/parameters.yml" file
       Clearing the cache for the dev environment with debug true
       Installing assets using the hard copy option
       Installing assets for Symfony\Bundle\FrameworkBundle into web/bundles/framework
       Installing assets for Acme\DemoBundle into web/bundles/acmedemo
       Installing assets for Sensio\Bundle\DistributionBundle into web/bundles/sensiodistribution

-----> Building runtime environment...

-----> Discovering process types
       Procfile declares types -> web

-----> Compressing... done, 61.5MB

-----> Launching... done, v3
       http://mighty-hamlet-1981.herokuapp.com/ deployed to Heroku

To git@heroku.com:mighty-hamlet-1981.git
 * [new branch]      master -> master
```

就是这样！如果您现在打开您的浏览器，要不就手动指向 **heroku create** 给您的 URL，或者也可以使用 Heroku Toolbelt，应用程序将会响应：

```
$ heroku open
Opening mighty-hamlet-1981... done
```

您应该在您的浏览器中看到 Symfony 应用程序。

如果您第一步在 Heroku 上安装全新的 Symfony 标准版本，您或许会遇到一个 404 页面没有找到错误。这是因为 / 的路径是由 AcmeDemoBundle 定义的，但是 AcmeDemoBundle 只在 dev 环境中加载（查看您的 **AppKernel** 类）。尝试从 AppBundle 打开 **/app/example**。

### 自定义编译步骤

如果您想在创建过程中执行另外的自定义命令，您可以利用 Heroku 的[自定义编译步骤](https://devcenter.heroku.com/articles/php-support#custom-compile-step)。想象您为了避免潜在的易损性想从 Heroku 中的产出环境移除 **dev** 前端控件。添加一个命令来移除 **web/app_dev.php**，Composer 的[安装后命令](https://getcomposer.org/doc/articles/scripts.md)就会工作，但是它也会分别地移除本地 **composer install** 或 **composer update** 开发环境中的控件。相反，您可以在您的 **composer.json** 的 **scripts** 区段里添加一个[自定义 Composer 命令](https://getcomposer.org/doc/articles/scripts.md#writing-custom-commands)叫做 **compile** （这个关键的名字是 Heroku 惯例）。列出的命令钩挂到 Heroku 的部署过程：

```
{
    "scripts": {
        "compile": [
            "rm web/app_dev.php"
        ]
    }
}
```

对于在产出系统中创建资产也是很有用的，例如：用 Assetic：

```
{
    "scripts": {
        "compile": [
            "app/console assetic:dump"
        ]
    }
}
```

### *Node.js 依赖*

构建资产可能取决于节点包，例如 **uglifyjs** 或 **uglifycss** 资产缩小。在部署过程中安装节点包需要节点的安装。但是现在，Heroku 使用 PHP 构建包编译您的应用程序，是由 **composer.json** 文件的存在而自动侦测，不包括节点安装。因为 Node.js 构建包比 PHP 构建包（参见 [Heroku 构建包](https://devcenter.heroku.com/articles/buildpacks)）更优先，添加 **package.json** 列出您的节点依赖从而使 Heroku 选择 Node.js 构建包：

```
{
    "name": "myApp",
    "engines": {
        "node": "0.12.x"
    },
    "dependencies": {
        "uglifycss": "*",
        "uglify-js": "*"
    }
}
```

根据下一次部署，Heroku 使用 Node.js 构建包来编译应用程序并且安装您的 npm 包。另一方面，**composer.json** 现在被忽略。用两个构建包,Node.js *和* PHP 编译您的应用程序的话，您可以使用一个特殊的[多样构建](https://github.com/ddollar/heroku-buildpack-multi)。为了覆盖构建包自动侦测，您需要明确地设置构建包 URL：

```
$ heroku buildpacks:set https://github.com/ddollar/heroku-buildpack-multi.git
```

接下来，添加 **.buildpacks** 文件到您的项目中，列出您所需要的构建包：

```
ttps://github.com/heroku/heroku-buildpack-nodejs.git
https://github.com/heroku/heroku-buildpack-php.git
```

有了下一次部署，您可以从两个构建包获益。此设置也启动您的 Heroku 环境，充分使用像 [Grunt](http://gruntjs.com/) 或者 [gulp](http://gulpjs.com/) 这些基于自动构建工具的结点。
