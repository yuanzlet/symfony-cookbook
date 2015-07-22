# 升级一个副版本

如果您正在升级一个副版本（中间的数字变化），那么您不应该会遇到重要的向后兼容性的改变。细节参见 [Symfony backwards compatibility promise](http://symfony.com/doc/current/contributing/code/bc.html)。

然而，一些向后兼容的中断是可能发生的，您会在一秒钟内学习如何准备他们。

这里有 2 个步骤来升级一个副版本：

1. [通过 Composer 升级 Symfony 库](http://symfony.com/doc/current/cookbook/upgrade/minor_version.html#upgrade-minor-symfony-composer)；
2. [更新您的代码，使之在新的版本中工作](http://symfony.com/doc/current/cookbook/upgrade/minor_version.html#upgrade-minor-symfony-code)。

## 1）通过 Composer 升级 Symfony 库

首先，您需要通过修改您的 **composer.json** 升级 Symfony 来使用新版本：

```
{
    "...": "...",

    "require": {
        "symfony/symfony": "2.6.*",
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

> $ composer update symfony/symfony --with-dependencies

这个命令升级 **symfony/symfony** 和它依赖的所有包（packages），其中包含一些其他的包。通过使用 **composer.json** 里的严格版本限制（tight version constraints），您可以控制每个库升级到哪个版本。

如果这仍不管用，你的 **composer.json** 文件可能指定了某个库的一个版本，这个库与新版本的 Symfony 不兼容。这种情况下，更新 **composer.json** 中的那个库到更新的版本可能能够解决这个问题。

或者，你可能会有更深层的问题，不同的库依赖于其他库冲突的版本。检查错误信息以调试。

### 升级其他包

您可能也想升级剩余的库。如果您在 **composer.json** 中对您的[版本限制（version constraints）](https://getcomposer.org/doc/01-basic-usage.md#package-versions)做的很好。您可以执行以下代码来安全地升级：

```
$ composer update
```

> 注意，如果在您的 **composer.json** （例如 **dev-master**）中含有非特定的[版本限制（version constraints）](https://getcomposer.org/doc/01-basic-usage.md#package-versions)，这可以使一些 non-Symfony 库升级到含有后向兼容中断改变的版本。

## 2）更新您的代码，使之在新的版本中工作

理论上，您本应做！然而，您可能需要对您的代码进行少许修改，使所有部分起作用。此外，一些功能在您使用的时候仍然可以工作，但现在可能已经弃用。这不是大问题，如果您知道弃用的东西是什么，您就可以花时间来修复它们了。

Symfony 每个版本都附带升级文件（例如 [UPGRADE-2.7.md](https://github.com/symfony/symfony/blob/2.7/UPGRADE-2.7.md)），储存在 Symfony 目录下，里面描述了这些改变。如果您按照文件中的说明更新相应代码，那么更新将会是安全的。

这些更新文件也可以在 [Symfony Repository](https://github.com/symfony/symfony) 找到。
