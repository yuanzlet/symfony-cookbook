# 如何使用 Doctrine 扩展：Timestampable, Sluggable, Translatable 等等

Doctrine2 非常灵活，并且社区已经创建了一系列有用的 Doctrine 扩展来帮助您做一些常见的实体相关的任务。

特别有一个库— [DoctrineExtensions](https://github.com/Atlantic18/DoctrineExtensions) 库—为 [Sluggable](https://github.com/Atlantic18/DoctrineExtensions/blob/master/doc/sluggable.md), [Translatable](https://github.com/Atlantic18/DoctrineExtensions/blob/master/doc/translatable.md), [Timestampable](https://github.com/Atlantic18/DoctrineExtensions/blob/master/doc/timestampable.md), [Loggable](https://github.com/Atlantic18/DoctrineExtensions/blob/master/doc/loggable.md), [Tree](https://github.com/Atlantic18/DoctrineExtensions/blob/master/doc/tree.md) 和 [Sortable](https://github.com/Atlantic18/DoctrineExtensions/blob/master/doc/sortable.md) 行为提供了整合功能。

这些扩展的每一个使用将在库中解释。

然而，为了安装/激发每一个扩展，您必须注册并激活一个 [Event Listener](http://symfony.com/doc/current/cookbook/doctrine/event_listeners_subscribers.html)。做这个您有两个选择：

1.	使用 [StofDoctrineExtensionsBundle](https://github.com/stof/StofDoctrineExtensionsBundle)，整合以上的库。

2.	直接实施这项服务，通过此文档材料与 Symfony 的集成：[在 Symfony2 中安装 Gedmo Doctrine2 扩展](https://github.com/Atlantic18/DoctrineExtensions/blob/master/doc/symfony2.md)。
