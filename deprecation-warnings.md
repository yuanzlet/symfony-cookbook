# &quot;XXX is deprecated&quot; E-USER-DEPRECATED 的警告是什么意思?



从 Symfony 2.7 开始，如果您使用一个过时的类、功能或选项，Symfony 就会触发 **E_USER_DEPRECATED** 错误。在内部，它看起来是像这样的东西：

```
trigger_error(
    'The fooABC method is deprecated since version 2.4 and will be removed in 3.0.',
    E_USER_DEPRECATED
);
```

这很好，因为您能够通过检查您的日志知道在您升级程序之前需要改变什么。在 Symfony 框架中，过时的通知的数量会显示在 web 调试工具栏上。并且，如果您安装了 [phpunit-bridge](https://github.com/symfony/phpunit-bridge)，您会在运行完您的任务之后得到一份过时通知的报告。

## 我如何才能沉默这种警告？

这些都是有用的，您不想让它们在开发中出现并且您也可能想让它们在生成的时候沉默，以避免它们填充错误日志。

### 在 Symfony 框架中

在 Symfony 框架中，**~E_USER_DEPRECATED** 被自动添加到 **app/bootstrap.php.cache** 中，但是您至少需要 2.3.14 版本或者 [SensioDistributionBundle](https://github.com/sensiolabs/SensioDistributionBundle) 的 3.0.21 版本。所以，您也许需要升级：

```
$ composer update sensio/distribution-bundle
```

一旦您升级了，文件 **bootstrap.php.cache** 会自动重建。您会看到顶部添加了一行 **~E_USER_DEPRECATED**。

### 在 Symfony 框架之外

要做到这一点，添加 **~E_USER_DEPRECATED** 到 **php.ini** 里的 **error_reporting** 中：

```
; before
error_reporting = E_ALL
; after
error_reporting = E_ALL & ~E_USER_DEPRECATED
```

或者，您可以在您项目的引导程序中直接设置它：

```
error_reporting(error_reporting() & ~E_USER_DEPRECATED);
```

## 我如何才能修复这个警告？

基本上，当然，您想要停止使用过时的功能。有时这很容易：警告可能准确的告诉您需要改变什么。

但其他情况下，警告可能会不清楚：某个地方的一个设置可能会使一个更深层的类触发这个警告。这种情况下，Symfony 尽力给出了一个明确的信息，但您可能需要进一步的研究这一警告。

有时，警告可能来源于您所使用的第三方库或者包（bundle）。如果是这样的话，有一个很好的机会，那些过时的东西都已更新。这种情况下，升级库来修复它们。

一旦所有的过时警告都消失了，您就有更多的信心来升级了。
