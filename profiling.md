# 如何在功能测试中使用分析器



重点推荐一个功能测试只测试 Response。但是如果您写了监视您的生产服务器的功能测试，您也许想要写一个分析器数据的测试，因为它给了您一个很好的方法来检查各种事情并执行一些度量。

[Symfony 分析器](http://symfony.com/doc/current/cookbook/profiler/index.html)为每一个请求采集大量数据。用这些数据来检查数据可调用的次数，框架中花费的时间，等等。但在写断言之前，启动分析器并检查它，是确实很有效的（它在在 **test** 环境中默认启动）。

```PHP
class HelloControllerTest extends WebTestCase
{
    public function testIndex()
    {
        $client = static::createClient();

        // Enable the profiler for the next request
        // (it does nothing if the profiler is not available)
        $client->enableProfiler();

        $crawler = $client->request('GET', '/hello/Fabien');

        // ... write some assertions about the Response

        // Check that the profiler is enabled
        if ($profile = $client->getProfile()) {
            // check the number of requests
            $this->assertLessThan(
                10,
                $profile->getCollector('db')->getQueryCount()
            );

            // check the time spent in the framework
            $this->assertLessThan(
                500,
                $profile->getCollector('time')->getDuration()
            );
        }
    }
}
```

如果测试因为分析器数据（例如太多的数据库问题）而失败，您可能想要在测试结束后用 Web Profiler 来分析请求，如果您将 token 嵌入到错误信息中，这是很容易完成的：

```PHP
$this->assertLessThan(
    30,
    $profile->getCollector('db')->getQueryCount(),
    sprintf(
        'Checks that query count is less than 30 (token %s)',
        $profile->getToken()
    )
);
```

> 分析器 store 可以由环境决定而不同（尤其如果您使用默认配置使用的 SQLite store 的话）。

> 即使您隔离了客户端或为您的测试使用了 HTTP 层，分析器信息都是可用的。

> 阅读内置的[数据收集器](http://symfony.com/doc/current/cookbook/profiler/data_collector.html)的 API 来了解更多关于它们的接口。

## 加速测试不收集分析器数据

避免在每次测试中都收集数据，您可以将收集参数设置为 false：

```YAML
# app/config/config_test.yml

# ...
framework:
    profiler:
        enabled: true
        collect: false
```

```XML
<!-- app/config/config.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:framework="http://symfony.com/schema/dic/symfony"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                http://symfony.com/schema/dic/symfony http://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

    <!-- ... -->

    <framework:config>
        <framework:profiler enabled="true" collect="false" />
    </framework:config>
</container>
```

```PHP
// app/config/config.php

// ...
$container->loadFromExtension('framework', array(
    'profiler' => array(
        'enabled' => true,
        'collect' => false,
    ),
));
```

这样的话，只有调用 **$client->enableProfiler()** 的测试才会收集数据。
