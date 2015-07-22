# 升级一个补丁版本

当一个新的补丁版本被发布时（只有最后一个数字改变了），被发布的内容只包含错误修正。这意味着升级一个补丁版本真的很简单。

```
$ composer update symfony/symfony
```

就是这个！您不应该遇到任何的后退 - 兼容性中断或是需要改变您的代码中的任何内容。这是因为当您开启您的工程的时候，您的 **composer.json** 包含应用了一个像 __2.6.*__ 的限制的 Symfony，在那里当您升级时只有最后一个版本数字会改变。

> 推荐尽快升级补丁版本，这是因为重要的程式错误和安全泄露可能在这些更新中被修补。

## 升级其它的包（Packages）

您可能也想升级剩余的库。如果您在 **composer.json** 中对您的[版本限制（version constraints）](https://getcomposer.org/doc/01-basic-usage.md#package-versions)做的很好。您可以执行以下代码来安全地升级：

```
$ composer update
```  

> 注意，如果在您的 **composer.json** （例如 **dev-master**）中含有非特定的[版本限制（version constraints）](https://getcomposer.org/doc/01-basic-usage.md#package-versions)，这可以使一些 non-Symfony 库升级到含有后向兼容中断改变的版本。
