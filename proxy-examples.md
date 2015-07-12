# 会话代理实例

会话代理机制有多种用途，这个例子展示了两个常见用法。不像通常的的那样加入会话处理程序，处理程序被添加到代理服务器，并在被会话储存驱动程序注册。

```PHP
use Symfony\Component\HttpFoundation\Session\Session;
use Symfony\Component\HttpFoundation\Session\Storage\NativeSessionStorage;
use Symfony\Component\HttpFoundation\Session\Storage\Handler\PdoSessionHandler;

$proxy = new YourProxy(new PdoSessionHandler());
$session = new Session(new NativeSessionStorage(array(), $proxy));
```

下面，您将会学习两个实例，可用于 **YourProxy**：会话数据的加密和只读的客人会话。

## 会话数据的加密

如果您想要加密会话数据，您可以使用代理服务器来加密和解密所需会话：

```PHP
use Symfony\Component\HttpFoundation\Session\Storage\Proxy\SessionHandlerProxy;

class EncryptedSessionProxy extends SessionHandlerProxy
{
    private $key;

    public function __construct(\SessionHandlerInterface $handler, $key)
    {
        $this->key = $key;

        parent::__construct($handler);
    }

    public function read($id)
    {
        $data = parent::read($id);

        return mcrypt_decrypt(\MCRYPT_3DES, $this->key, $data);
    }

    public function write($id, $data)
    {
        $data = mcrypt_encrypt(\MCRYPT_3DES, $this->key, $data);

        return parent::write($id, $data);
    }
}
```

## 只读客人会话

有一些应用程序，其会话被客人（guest）用户需求，但没有特别的需要储存会话的需求。在这种情况下，您可以在会话被写入之前拦截它。

```PHP
use Foo\User;
use Symfony\Component\HttpFoundation\Session\Storage\Proxy\SessionHandlerProxy;

class ReadOnlyGuestSessionProxy extends SessionHandlerProxy
{
    private $user;

    public function __construct(\SessionHandlerInterface $handler, User $user)
    {
        $this->user = $user;

        parent::__construct($handler);
    }

    public function write($id, $data)
    {
        if ($this->user->isGuest()) {
            return;
        }

        return parent::write($id, $data);
    }
}
```
