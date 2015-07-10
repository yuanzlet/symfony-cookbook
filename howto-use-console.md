# 如何使用控制台

组件文档的[使用控制台命令，快捷键和内建命令](http://symfony.com/doc/current/components/console/usage.html)页全局的看控制台选项。当你使用控制台作为全栈框架的一部分时，一些附加的全局选项也是可用的。  

默认情况下，控制台命令在 **dev** 环境中运行，你可能要对某些命令改变这种情况。举例来说，你可能由于性能的原因想要在 **prod** 环境下运行一些命令。除此之外，一些命令的结果可能会由于环境额不同而不同。举例来说，**cache:clear** 命令将会在特定的环境下清空和加热缓存。为了清空和加热 **prod** 缓存，你需要运行下列代码：  

```
$ php app/console cache:clear --env=prod
```  

或者同样的代码：  

```
$ php app/console cache:clear -e prod
```  

除了改变环境，你也可以选择禁用调试模式。在 **dev** 环境中你想要运行命令避免收集调试数据的性能损失这个可能有用：  

```
$ php app/console list --no-debug
```  

有一个交互的 shell 这个 shell 每次允许你在没有区分 **php app/console** 的情况下输入命令，如果你需要应用几个命令这个将会很有用。输入下列代码进入 shell：  

```
$ php app/console --shell
$ php app/console -s
```  

现在你就可以使用命令名称运行命令：  

```
Symfony > list
```  

当你使用 shell 的时候你可以选择在分开的进程中运行每个命令：  

```
$ php app/console --shell --process-isolation
$ php app/console -s --process-isolation
```  

当你做这个的时候，输出结果将不会被粉饰并且不支持交互因此你需要清晰地传递所有的命令参数。  

>除非你使用隔离的进程，在 shell 中清空缓存将不会你随后运行的命令中有效果，这是因为原始的缓存文件还在使用。  


