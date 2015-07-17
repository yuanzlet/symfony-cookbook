# 如何使用内建的 PHP Web 服务器

> **2.6** 在 Symfony 2.6 节中介绍了把服务器作为后台进程运行的功能。

从 PHP 5.4 版本以来，CLI SAPI 就带有内置的 web 服务器  [built-in web server](http://php.net/manual/en/features.commandline.webserver.php "built-in web server") ，当您在开发项目的时候，它可以在本地运行您的程序，并且进行测试和展示。通过这种方式，您就没有必要很麻烦的去配置如同 [Apache](http://symfony.com/doc/current/cookbook/configuration/web_server_configuration.html "Apache") 或者  [Nginx](http://symfony.com/doc/current/cookbook/configuration/web_server_configuration.html "Nginx")  等功能完整的 web 服务器。

> 内置的 web 服务器只用于您可控制的网络中，而不是用于一个人人都可访问的公共网络中。

## 启动 web 服务器

在一个 PHP 的内置服务器中运行 Symfony 程序就像执行启动服务器命令 **server:start** 那样简单：

```
$ php app/console server:start
```

通过这个命令就可以在 localhost 下的 8000 端口 **localhost:8000** 启动您的 Symfony 程序运行环境。

在默认的情况下，服务器在环回设备上监听的是 8000 端口，您也可以通过命令行传输一个 IP 地址和端口号来更改套接字：

```
$ php app/console server:run 192.168.0.1:8080
```

> 您可以通过使用 **server:status** 命令来检查一个服务器是否正在监听一个确定的套接字：

```
$ php app/console server:status

$ php app/console server:status 192.168.0.1:8080
```

>  上述第一行代码表示您将要通过 **localhost:8000** 访问服务器来运行 Symfony 程序，第二行代码表示也可以通过访问  **192.168.0.1:8080** 达到同样的目的。

> 在 Symfony 2.6 版本以前，**server:run** 命令式用来启动内置服务器的，在 2.6 版本以后，这个命令任然有效，但是略有不同。当使用这个命令时，它将会拦截当前的终端，除非您终止这个操作（通常是用按键 Ctrl 和 C 实现 ），而不是启动后台服务器。

> ## 在虚拟机内部使用内置 web 服务器
> 如果您想在内置的虚拟机上运行内置 web 服务器并且通过浏览器来加载您主机中的网站，那么您需要监听 0.0.0.0:8000 地址 （即分配给虚拟机的所有的 IP 地址）：

```
$ php app/console server:start 0.0.0.0:8000
```

> 切记，永远不要使用一个可以直接通过 Internet 直接访问的计算机来监听所有的接口。不能在公用的网络中使用内置的 web 服务器。

## 命令选项

内置 web 服务器期望把一个“路由器”脚本（“路由器”脚本章节见  [php.net](http://php.net/manual/en/features.commandline.webserver.php#example-401 "php.net")） 作为参数。当命令还在产品或者是其他开发环境中执行时，已经有一个这样的“路由器”脚本参数传递给了 Symfony。可以在任何环境或者路由器脚本中使用路由器选项：

```
$ php app/console server:start --env=test --router=app/config/router_test.php
```

如果您的程序的根文档和标准的目录布局不同，那么您需要通过使用 **--docroot** 选项来传递正确的位置：

```
$ php app/console server:start --docroot=public_html
```

## 停止服务器

当您完成了工作，您可以通过 **server:stop** 命令来停止服务器：

```
$ php app/console server:stop
```

就像使用启动服务器命令一样，如果你省略了套接字信息， Symfony 会停止 **localhost:8000** 下的服务器。所有，当您的服务器监听的不是默认地址或者端口的时候，请在执行命令的时候加上套接字信息：

```
$ php app/console server:stop 192.168.0.1:8080
```

