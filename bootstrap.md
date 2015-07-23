# 如何在运行测试之前自定义引导过程

有时运行测试，在运行测试之前，需要做额外的引导工作。例如，如果您正在运行一个功能测试并引入了一个新的翻译资源，那么您需要在运行这些测试之前清除缓存。这份码元书包括了如何做到这一点。

首先，添加以下文件：

```PHP
// app/tests.bootstrap.php
if (isset($_ENV['BOOTSTRAP_CLEAR_CACHE_ENV'])) {
    passthru(sprintf(
        'php "%s/console" cache:clear --env=%s --no-warmup',
        __DIR__,
        $_ENV['BOOTSTRAP_CLEAR_CACHE_ENV']
    ));
}

require __DIR__.'/bootstrap.php.cache';
```

用 **tests.bootstrap.php** 替换 **app/phpunit.xml.dist** 里的测试引导文件 **bootstrap.php.cache**：

```
<!-- app/phpunit.xml.dist -->

<!-- ... -->
<phpunit
    ...
    bootstrap = "tests.bootstrap.php"
>
```

现在您可以在您的 **phpunit.xml.dist** 文件中定义您想要在哪种环境下清理缓存：

```PHP
<!-- app/phpunit.xml.dist -->
<php>
    <env name="BOOTSTRAP_CLEAR_CACHE_ENV" value="test"/>
</php>
```

现在，这已经变成了一个环境变量（即 **$_ENV**），可以在自定义引导文件（**tests.bootstrap.php**）中找到。
