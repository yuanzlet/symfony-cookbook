# 如何测试多个客户端的交互



如果您需要模拟不同客户之间的互动（比如说一个聊天），先创建几个客户：

```PHP
// ...

$harry = static::createClient();
$sally = static::createClient();

$harry->request('POST', '/say/sally/Hello');
$sally->request('GET', '/messages');

$this->assertEquals(Response::HTTP_CREATED, $harry->getResponse()->getStatusCode());
$this->assertRegExp('/Hello/', $sally->getResponse()->getContent());
```

当您的代码存在一个全局状态（global state）或者它依赖的第三方库中存在全局状态，不存在这个工作。这种情况下，您可以隔离您的用户：

```PHP
// ...

$harry = static::createClient();
$sally = static::createClient();

$harry->insulate();
$sally->insulate();

$harry->request('POST', '/say/sally/Hello');
$sally->request('GET', '/messages');

$this->assertEquals(Response::HTTP_CREATED, $harry->getResponse()->getStatusCode());
$this->assertRegExp('/Hello/', $sally->getResponse()->getContent());
```

隔离的用户在一个专用和整洁的 PHP 过程中透明地执行它们的需求，从而避免一切副作用。

> 由于隔离的用户更慢，您可以在主过程（main process）中留下一个用户，然后隔离其他用户。
