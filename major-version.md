# 升级一个主版本

每隔几年，Symfony 都会发布一个新的主版本（第一个数字改变）。这些版本是最棘手的升级，因为它们允许包含向后兼容的中断。然而，Symfony 试图使这个升级过程尽可能的平稳。

这意味着您可以在主版本发布之前更新您的代码。这被称为使您的代码未来兼容（future compatible）。

这里有三个步骤升级一个主版本：

1. [使您的代码不是过时的](http://symfony.com/doc/current/cookbook/upgrade/major_version.html#upgrade-major-symfony-deprecations)；
2. [通过 Composer 更新新的主版本](http://symfony.com/doc/current/cookbook/upgrade/major_version.html#upgrade-major-symfony-composer)；
3. [更新您的代码，使之在新的版本中工作](http://symfony.com/doc/current/cookbook/upgrade/major_version.html#upgrade-major-symfony-after)。

## 1）使您的代码不是过时的

在一个主版本的生命周期中，新的特性被添加，方法签名和公共 API 的用法改变。但是，[副版本](http://symfony.com/doc/current/cookbook/upgrade/minor_version.html)不应该包含任何向后兼容的更改。要做到这一点，“老的”（例如，函数，类，等等）的代码仍然工作，但被标记为过时，表明它将在未来删除/改变，您应该停止使用它。

当主版本（例如 3.0.0）发布，所有的过时的特性和功能都会被去除。所以，只要您将代码中的主版本之前的最后一个版本（例如 2.8.*）中过时的特性都停用，您应该能没有问题地升级。

为了帮助您完成这个，最后一个发布的副版本将会触发过期提醒。例如，2.7 和 2.8 版本触发过期提醒。当在浏览器的[开发环境](http://symfony.com/doc/current/cookbook/configuration/environments.html)上访问您的应用程序时，这些提醒会在 web 开发工具栏显示:

![1](images/deprecations-in-profiler.png)

### PHPUnit 中的过时部分

默认情况下，PHPUnit 会像处理真正的错误一样处理过时提醒。这意味着所有的任务都被终止，因为它使用了向后兼容层（BC layer）。

为了确保这样的情况不会发生，您可以安装 PHPUnit bridge：

```
$ composer require symfony/phpunit-bridge
```

现在，您的任务像通常一样执行，一个漂亮的过期提醒总结显示在测试报告末尾。

```
$ phpunit
...

OK (10 tests, 20 assertions)

Remaining deprecation notices (6)

The "request" service is deprecated and will be removed in 3.0. Add a typehint for
Symfony\Component\HttpFoundation\Request to your controller parameters to retrieve the
request instead: 6x
    3x in PageAdminTest::testPageShow from Symfony\Cmf\SimpleCmsBundle\Tests\WebTest\Admin
    2x in PageAdminTest::testPageList from Symfony\Cmf\SimpleCmsBundle\Tests\WebTest\Admin
    1x in PageAdminTest::testPageEdit from Symfony\Cmf\SimpleCmsBundle\Tests\WebTest\Admin
```

## 2）通过 Composer 更新新的主版本

如果您的代码不是过时的，您可以通过 Composer 使用修改 **composer.json** 的方法更新 Symfony 库：

```
{
    "...": "...",

    "require": {
        "symfony/symfony": "3.0.*",
    },
    "...": "...",
}
```

下一步，用 Composer 下载新版本的库：

```
$ composer update symfony/symfony
```

### 依赖错误

如果您得到了一个依赖的错误，它可能只是意味着您需要升级其他 Symfony 的依赖。在这种情况下，尝试以下命令：

> *$ composer update symfony/symfony --with-dependencies*

这个命令升级 **symfony/symfony** 和它依赖的所有包（packages），其中包含一些其他的包。通过使用 **composer.json** 里的严格版本限制（tight version constraints），您可以控制每个库升级到哪个版本。

如果这仍不管用，您的 **composer.json** 文件可能指定了某个库的一个版本，这个库与新版本的 Symfony 不兼容。这种情况下，更新 **composer.json** 中的那个库到更新的版本可能能够解决这个问题。

或者，您可能会有更深层的问题，不同的库依赖于其他库冲突的版本。检查错误信息以调试。

### 升级其他包

您可能也想升级剩余的库。如果您在 **composer.json** 中对您的[版本限制（version constraints）](https://getcomposer.org/doc/01-basic-usage.md#package-versions)做的很好。您可以执行以下代码来安全地升级：

```
$ composer update
```

> 注意，如果在您的 **composer.json** （例如 **dev-master**）中含有非特定的[版本限制（version constraints）](https://getcomposer.org/doc/01-basic-usage.md#package-versions)，这可以使一些 non-Symfony 库升级到含有后向兼容中断改变的版本。

## 3）更新您的代码，使之在新的版本中工作

现在您有一个很好的机会！然而，由于后向兼容层并不总是可能的，新的主版本可能包含新的向后兼容中断。确保您阅读 Symfony 中的 **UPGRADE-X.0.md**（其中 X 是新版本号），并查看您需要注意的向后兼容中断。
