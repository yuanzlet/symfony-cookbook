# 如何以非继承方式自定义方法

## 在方法调用前后做一些事情

如果您想在调用一个方法之前后之后做一些事情，您可以在方法开始或结尾分别派遣一个事件：

```
class Foo
{
    // ...

    public function send($foo, $bar)
    {
        // do something before the method
        $event = new FilterBeforeSendEvent($foo, $bar);
        $this->dispatcher->dispatch('foo.pre_send', $event);

        // get $foo and $bar from the event, they may have been modified
        $foo = $event->getFoo();
        $bar = $event->getBar();

        // the real method implementation is here
        $ret = ...;

        // do something after the method
        $event = new FilterSendReturnValue($ret);
        $this->dispatcher->dispatch('foo.post_send', $event);

        return $event->getReturnValue();
    }
}
```

在这个例子中，两个事件被扔出：**foo.pre_send** 在方法前执行，**foo.post_send** 在方法后执行。每一个使用自定义事件类来与两个事件的监听器交流。这些事件类需要由您创建并且按照这个例子中，允许变量 **$foo, $bar** 和 **$ret** 被检索并由监听器设置。

例如，假设 **FilterSendReturnValue** 有一个 **setReturnValue** 方法，一个监听器可能看起来像这样：

```
public function onFooPostSend(FilterSendReturnValue $event)
{
    $ret = $event->getReturnValue();
    // modify the original ``$ret`` value

    $event->setReturnValue($ret);
}
```


