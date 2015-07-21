# 如何注册事件监听器和订阅

Doctrine 包括一个丰富的事件系统可以在任何事情在系统内部发生的时候激发事件。对于您来说，这意味着您可以创建任意的[服务](http://symfony.com/doc/current/book/service_container.html)并且让 Doctrine 在任何时候，一个特定的动作（例如：**prePersist**）在 Doctrine 内部发生时，通知那些对象。这可以是有用的，例如，在任何时候您的数据库保存一个对象就创建一个独立的搜索表单。

Doctrine 定义了两种类型的对象，可以监听 Doctrine 事件：监听器和订阅。它们都很相似，但是监听器更为直接。更多的，可以参见 Doctrine 网站上的[事件系统](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/events.html)。

Doctrine 网站也解释了所有可以被监听的现存事件。

## 配置监听器或订阅

注册一个服务充当一个事件监听器或者订阅，您只需要用过适当的名字[标记](http://symfony.com/doc/current/book/service_container.html#book-service-container-tags)它即可。取决于您的使用实例，您可以钩挂一个监听器到每一个 DBAL 连接和 ORM 实体管理器或者只是进入一个具体的 DBAL 连接和所有使用这个连接的实体管理器。

YAML

```
doctrine:
    dbal:
        default_connection: default
        connections:
            default:
                driver: pdo_sqlite
                memory: true

services:
    my.listener:
        class: Acme\SearchBundle\EventListener\SearchIndexer
        tags:
            - { name: doctrine.event_listener, event: postPersist }
    my.listener2:
        class: Acme\SearchBundle\EventListener\SearchIndexer2
        tags:
            - { name: doctrine.event_listener, event: postPersist, connection: default }
    my.subscriber:
        class: Acme\SearchBundle\EventListener\SearchIndexerSubscriber
        tags:
            - { name: doctrine.event_subscriber, connection: default }
```

XML

```
<?xml version="1.0" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:doctrine="http://symfony.com/schema/dic/doctrine">

    <doctrine:config>
        <doctrine:dbal default-connection="default">
            <doctrine:connection driver="pdo_sqlite" memory="true" />
        </doctrine:dbal>
    </doctrine:config>

    <services>
        <service id="my.listener" class="Acme\SearchBundle\EventListener\SearchIndexer">
            <tag name="doctrine.event_listener" event="postPersist" />
        </service>
        <service id="my.listener2" class="Acme\SearchBundle\EventListener\SearchIndexer2">
            <tag name="doctrine.event_listener" event="postPersist" connection="default" />
        </service>
        <service id="my.subscriber" class="Acme\SearchBundle\EventListener\SearchIndexerSubscriber">
            <tag name="doctrine.event_subscriber" connection="default" />
        </service>
    </services>
</container>
```

PHP

```
use Symfony\Component\DependencyInjection\Definition;

$container->loadFromExtension('doctrine', array(
    'dbal' => array(
        'default_connection' => 'default',
        'connections' => array(
            'default' => array(
                'driver' => 'pdo_sqlite',
                'memory' => true,
            ),
        ),
    ),
));

$container
    ->setDefinition(
        'my.listener',
        new Definition('Acme\SearchBundle\EventListener\SearchIndexer')
    )
    ->addTag('doctrine.event_listener', array('event' => 'postPersist'))
;
$container
    ->setDefinition(
        'my.listener2',
        new Definition('Acme\SearchBundle\EventListener\SearchIndexer2')
    )
    ->addTag('doctrine.event_listener', array('event' => 'postPersist', 'connection' => 'default'))
;
$container
    ->setDefinition(
        'my.subscriber',
        new Definition('Acme\SearchBundle\EventListener\SearchIndexerSubscriber')
    )
    ->addTag('doctrine.event_subscriber', array('connection' => 'default'))
;
```

## 创建监听器类

在之前的例子中，服务 **my.listener** 被配置为 **postPersist** 事件中的 Doctrine 监听器。服务后的类必须有 **postPersist** 方法，当事件被发送时，此方法被调用。

```
// src/Acme/SearchBundle/EventListener/SearchIndexer.php
namespace Acme\SearchBundle\EventListener;

use Doctrine\ORM\Event\LifecycleEventArgs;
use Acme\StoreBundle\Entity\Product;

class SearchIndexer
{
    public function postPersist(LifecycleEventArgs $args)
    {
        $entity = $args->getEntity();
        $entityManager = $args->getEntityManager();

        // perhaps you only want to act on some "Product" entity
        if ($entity instanceof Product) {
            // ... do something with the Product
        }
    }
}
```

在每一个事件中，您可以访问 **LifecycleEventArgs** 对象，可以让您既访问事件的实例对象也可以访问实例管理器本身。

一个需要注意的重要的事情是监听器将会监听到应用程序中*所有的*实例。所以，如果您只是对实例具体的一个类型感兴趣（例如：**Product** 实例而不是 **BlogPost** 实例），您应该在您的方法中检查实例类型（如上面所示）。

在 Doctrine2.4 中，将要介绍一个功能叫做实体监听器。这是用于实例的一个生命周期监听器类。您可以在 [Doctrine 文档](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/events.html#entity-listeners)中阅读有关信息。

## 创建订阅类

Doctrine 事件订阅必须实现 **Doctrine\Common\EventSubscriber** 接口，并且对于每一个订阅的事件都得有一个事件方法：

```
// src/Acme/SearchBundle/EventListener/SearchIndexerSubscriber.php
namespace Acme\SearchBundle\EventListener;

use Doctrine\Common\EventSubscriber;
use Doctrine\ORM\Event\LifecycleEventArgs;
// for Doctrine 2.4: Doctrine\Common\Persistence\Event\LifecycleEventArgs;
use Acme\StoreBundle\Entity\Product;

class SearchIndexerSubscriber implements EventSubscriber
{
    public function getSubscribedEvents()
    {
        return array(
            'postPersist',
            'postUpdate',
        );
    }

    public function postUpdate(LifecycleEventArgs $args)
    {
        $this->index($args);
    }

    public function postPersist(LifecycleEventArgs $args)
    {
        $this->index($args);
    }

    public function index(LifecycleEventArgs $args)
    {
        $entity = $args->getEntity();
        $entityManager = $args->getEntityManager();

        // perhaps you only want to act on some "Product" entity
        if ($entity instanceof Product) {
            // ... do something with the Product
        }
    }
}
```

Doctrine 事件订阅不可以像 [Symfony 事件订阅](http://symfony.com/doc/current/components/event_dispatcher/introduction.html#event-dispatcher-using-event-subscribers)那样返回一系列灵活的方法来调用事件。Doctrine 事件订阅可以返回一系列简单的它们所订阅的事件的名称。Doctrine 期待当使用一个监听器的时，订阅上的方法同每一个订阅的事件有相同的名称。

想要完整的参考，参考 Doctrine 文档中的章节：[事件系统](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/events.html)。
