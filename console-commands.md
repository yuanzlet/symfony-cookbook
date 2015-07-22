# 控制台命令

Doctrine2 ORM 集成在 **doctrine** 命名空间下提供了几个控制台命令。为了查看命令列表您可以使用 **list** 命令：

```
$ php app/console list doctrine
```

一列可用的命令将打印出。您可以通过运行 **help** 命令发现更多关于任何这些命令的消息（或任何 Symfony 命令）。例如，要获取关于 **doctrine:database:create** 任务的细节，就运行：

```
$ php app/console help doctrine:database:create
```

一些值得注意并有趣的任务包括：

•	**doctrine:ensure-production-settings** — 检查看当前环境是否为生产有效地配置了。这应该总是在 **prod** 环境中运行的：

```
$ php app/console doctrine:ensure-production-settings --env=prod
```

•	**doctrine:mapping:import** — 允许 Doctrine 来内省一个已存在的数据库并创建映射信息。想知道更多信息，参见[如何在一个已有的数据库生成实体](http://symfony.com/doc/current/cookbook/doctrine/reverse_engineering.html)。

•	**doctrine:mapping:info** — 告诉您 Doctrine 所了解到的所有实体，以及映射中是否有基础错误。

•	**doctrine:query:dql** 和 **doctrine:query:sql**—允许您在命令行中直接执行 DQL 或者 SQL 问询。
