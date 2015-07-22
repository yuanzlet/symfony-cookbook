# 使用 Bower 安装 Symfony

Symfony 以及它的所有的包都是完美的由 Composer 所管理。Bower 是前端依赖性管理的附属工具，就像 Bootstrap 或者 jQuery。由于 Symfony 是一个纯的后端框架，因此他不能使用 Bower 来帮助你。幸运的是，它很好用！  

## 安装 Bower

[Bower](http://bower.io/) 位于 [Node.js](https://nodejs.org/) 的顶层。确保你安装了它然后运行下列代码：  

```
$ npm install -g bower
```

在完成这个命令之后，在你终端运行 **bower** 来检查它是否正确安装了。  

>如果你的电脑上没有安装 NodeJS，你也可以使用 [BowerPHP](http://bowerphp.org/)（一个非官方的 Bower 的 PHP 接口）。注意这个依然运行在阿尔法状态下。如果你使用 BowerPHP，那么就用 **BowerPHP** 代替例子中的 **bower**。  

## 在你的工程中配置 Bower

正常情况下，Bower 将所有东西下载到 **bower_components/** 命令中。在 Symfony 中，只有在 **web/** 目录下的文件才是可以公开访问的，因此作为替代你需要在那配置 Bower 来下载东西。为了完成这个，你需要创建一个具有新的目的地（例如 **web/assets/vendor**）的 **.bowerrc** 文件：  

```
{
    "directory": "web/assets/vendor/"
}
```

>如果你在使用基于前端的系统例如 Gulp 或者 Grunt，那么你就可以随意按照你想的来配置目录。典型地，你最终将使用这些工具来将你的所有资产移动到 **web/** 目录。  

## 一个例子：安装 Bootstrap

信不信由你，但是现在你已经准备好在你的 Symfony 应用程序中使用 Bower 了。作为一个例子，你将在你的工程中安装 Bootstrap 并且在你的布局中包含它。  

### 安装依赖性

只要运行 **bower init** 就能创建 **bower.json** 文件。现在你已经准备开始向你的工程中添加东西了。举例来说，向你的 [Bootstrap](http://getbootstrap.com/) 中添加 **bower.json** 并且下载它，只要运行下列代码就好：  

```
$ bower install --save bootstrap
```

这个将会在 **web/assets/vendor/** 中（或者你在 **.bowerrc** 中设置的其他路径）安装 Bootstrap 以及它的依赖性。  

>获取更多如何使用 Bower 的信息，请查阅 [Bower 文档](http://bower.io/)。  

### 在你的模板中包含依赖性

既然已经安装了依赖性，你可以像正常的 CSS/JS 一样在你的模板中包含 bootstrap：  

Twig：  

```
{# app/Resources/views/layout.html.twig #}
<!doctype html>
<html>
    <head>
        {# ... #}

        <link rel="stylesheet"
            href="{{ asset('assets/vendor/bootstrap/dist/css/bootstrap.min.css') }}">
    </head>

    {# ... #}
</html>
```

PHP：  

```
<!-- app/Resources/views/layout.html.php -->
<!doctype html>
<html>
    <head>
        {# ... #}

        <link rel="stylesheet" href="<?php echo $view['assets']->getUrl(
            'assets/vendor/bootstrap/dist/css/bootstrap.min.css'
        ) ?>">
    </head>

    {# ... #}
</html>
```

好样的！你的站点现在正在使用 Bootstrap。你现在可以轻松地将 bootstrap 升级到最新版本并且也可以管理其他的前端依赖性。  

### 我是应该 Git 忽略还是提交 Bower 资产？

目前，你应当*提交*由 Bower 下载的注册而不是向你的 **.gitignore** 文件添加目录（例如 **web/assets/vendor**）：  

```
$ git add web/assets/vendor
```

为什么呢？不像 Composer，Bower 目前没有“锁定”特征，这就意味着在不同的服务器上运行 **bower install** 将会给你*额外*的你在其它机器上的资产这没有保障。更多细节，详见[检查前端的依赖性](http://addyosmani.com/blog/checking-in-front-end-dependencies/)这篇文章。  

但是，在将来 Bower 很可能添加锁定特征（例如 [bower/bower#1748](https://github.com/bower/bower/pull/1748)）。  


