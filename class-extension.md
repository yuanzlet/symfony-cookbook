# 如何以非继承方式扩展一个类

允许多个类向另一个类添加方法，您可以在您想要扩展的类中定义你的魔法 **__call()**，像这样：

```
class Foo
{
    // ...

    public function __call($method, $arguments)
    {
        // create an event named 'foo.method_is_not_found'
        $event = new HandleUndefinedMethodEvent($this, $method, $arguments);
        $this->dispatcher->dispatch('foo.method_is_not_found', $event);

        // no listener was able to process the event? The method does not exist
        if (!$event->isProcessed()) {
            throw new \Exception(sprintf('Call to undefined method %s::%s.', get_class($this), $method));
        }

        // return the listener returned value
        return $event->getReturnValue();
    }
}
```

这使用了特别的，应该也可被创建的 **HandleUndefinedMethodEvent**。这是一个通用类，每次您需要使用这种模式的类扩展可以重复使用：

```
use Symfony\Component\EventDispatcher\Event;

class HandleUndefinedMethodEvent extends Event
{
    protected $subject;
    protected $method;
    protected $arguments;
    protected $returnValue;
    protected $isProcessed = false;

    public function __construct($subject, $method, $arguments)
    {
        $this->subject = $subject;
        $this->method = $method;
        $this->arguments = $arguments;
    }

    public function getSubject()
    {
        return $this->subject;
    }

    public function getMethod()
    {
        return $this->method;
    }

    public function getArguments()
    {
        return $this->arguments;
    }

    /**
     * Sets the value to return and stops other listeners from being notified
     */
    public function setReturnValue($val)
    {
        $this->returnValue = $val;
        $this->isProcessed = true;
        $this->stopPropagation();
    }

    public function getReturnValue()
    {
        return $this->returnValue;
    }

    public function isProcessed()
    {
        return $this->isProcessed;
    }
}
```

接下来，创建一个类，将会听取 **foo.method_is_not_found** 事件并且*添加*方法 **bar()**：

```
class Bar
{
    public function onFooMethodIsNotFound(HandleUndefinedMethodEvent $event)
    {
        // only respond to the calls to the 'bar' method
        if ('bar' != $event->getMethod()) {
            // allow another listener to take care of this unknown method
            return;
        }

        // the subject object (the foo instance)
        $foo = $event->getSubject();

        // the bar method arguments
        $arguments = $event->getArguments();

        // ... do something

        // set the return value
        $event->setReturnValue($someValue);
    }
}
```

最后，通过注册一个 **Bar** 的实例和事件 **foo.method_is_not_found** 添加新的 **bar** 方法到 **Foo** 类。

```
$bar = new Bar();
$dispatcher->addListener('foo.method_is_not_found', array($bar, 'onFooMethodIsNotFound'));
```
