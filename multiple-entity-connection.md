# 如何使用多个实体管理器和连接

您可以在一个 Symfony 应用程序中使用多个 Doctrine 实体管理器或者连接。这是必要的，如果您使用的是不同的数据库，甚至是完全不同的实体的供应商。换句话说，连接到一个数据库的一个实体管理器处理一些实体的时候，连接到另一个数据库的另一个实体管理器将会处理剩余的实体。

使用多个实体管理器相当简单，但是更为高级并且不是经常需要。在添加这复杂的层之前，确保您是真正需要多个实体管理器。

以下的配置代码展示了如何配置两个实体管理器：

YAML

```
doctrine:
    dbal:
        default_connection: default
        connections:
            default:
                driver:   "%database_driver%"
                host:     "%database_host%"
                port:     "%database_port%"
                dbname:   "%database_name%"
                user:     "%database_user%"
                password: "%database_password%"
                charset:  UTF8
            customer:
                driver:   "%database_driver2%"
                host:     "%database_host2%"
                port:     "%database_port2%"
                dbname:   "%database_name2%"
                user:     "%database_user2%"
                password: "%database_password2%"
                charset:  UTF8

    orm:
        default_entity_manager: default
        entity_managers:
            default:
                connection: default
                mappings:
                    AppBundle:  ~
                    AcmeStoreBundle: ~
            customer:
                connection: customer
                mappings:
                    AcmeCustomerBundle: ~
```

XML

```
<?xml version="1.0" encoding="UTF-8"?>
<srv:container xmlns="http://symfony.com/schema/dic/doctrine"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:srv="http://symfony.com/schema/dic/services"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                        http://symfony.com/schema/dic/doctrine http://symfony.com/schema/dic/doctrine/doctrine-1.0.xsd">

    <config>
        <dbal default-connection="default">
            <connection name="default"
                driver="%database_driver%"
                host="%database_host%"
                port="%database_port%"
                dbname="%database_name%"
                user="%database_user%"
                password="%database_password%"
                charset="UTF8"
            />

            <connection name="customer"
                driver="%database_driver2%"
                host="%database_host2%"
                port="%database_port2%"
                dbname="%database_name2%"
                user="%database_user2%"
                password="%database_password2%"
                charset="UTF8"
            />
        </dbal>

        <orm default-entity-manager="default">
            <entity-manager name="default" connection="default">
                <mapping name="AppBundle" />
                <mapping name="AcmeStoreBundle" />
            </entity-manager>

            <entity-manager name="customer" connection="customer">
                <mapping name="AcmeCustomerBundle" />
            </entity-manager>
        </orm>
    </config>
</srv:container>
```

PHP

```
$container->loadFromExtension('doctrine', array(
    'dbal' => array(
        'default_connection' => 'default',
        'connections' => array(
            'default' => array(
                'driver'   => '%database_driver%',
                'host'     => '%database_host%',
                'port'     => '%database_port%',
                'dbname'   => '%database_name%',
                'user'     => '%database_user%',
                'password' => '%database_password%',
                'charset'  => 'UTF8',
            ),
            'customer' => array(
                'driver'   => '%database_driver2%',
                'host'     => '%database_host2%',
                'port'     => '%database_port2%',
                'dbname'   => '%database_name2%',
                'user'     => '%database_user2%',
                'password' => '%database_password2%',
                'charset'  => 'UTF8',
            ),
        ),
    ),

    'orm' => array(
        'default_entity_manager' => 'default',
        'entity_managers' => array(
            'default' => array(
                'connection' => 'default',
                'mappings'   => array(
                    'AppBundle'  => null,
                    'AcmeStoreBundle' => null,
                ),
            ),
            'customer' => array(
                'connection' => 'customer',
                'mappings'   => array(
                    'AcmeCustomerBundle' => null,
                ),
            ),
        ),
    ),
));
```

这种情况下，您已经定义了两个实体管理器，称之为 **default** 和 **customer**。**default** 管理器管理在 AppBundle 和 AcmeStoreBundle 中的实体，而 **customer** 实体管理器管理在 AcmeCustomerBundle 中的实体。您也定义了两个连接，用于每个实体管理器。

当使用多个连接和实体管理器的时候，您应该对您想要的配置很明确。如果您*的确*遗漏了连接或者实体管理器的名称，就使用默认情况（例如 **default**）。

当使用多个连接来创建您的数据库时：

```
# Play only with "default" connection
$ php app/console doctrine:database:create

# Play only with "customer" connection
$ php app/console doctrine:database:create --connection=customer
```

当使用多个实体管理器来更新您的模式：

```
# Play only with "default" mappings
$ php app/console doctrine:schema:update --force

# Play only with "customer" mappings
$ php app/console doctrine:schema:update --force --em=customer
```

如果您在请求的时候*的确*遗漏了实体管理器的名称，返回到默认实体管理器（例如 **default**）

```
class UserController extends Controller
{
    public function indexAction()
    {
        // All three return the "default" entity manager
        $em = $this->get('doctrine')->getManager();
        $em = $this->get('doctrine')->getManager('default');
        $em = $this->get('doctrine.orm.default_entity_manager');

        // Both of these return the "customer" entity manager
        $customerEm = $this->get('doctrine')->getManager('customer');
        $customerEm = $this->get('doctrine.orm.customer_entity_manager');
    }
}
```

现在您可以像之前那样使用 Doctrine了—使用 **default** 实体来保存和读取它所管理的实体，**customer** 实体管理器来保存和读取它的实体。

同样适用于存储库调用：

```
class UserController extends Controller
{
    public function indexAction()
    {
        // Retrieves a repository managed by the "default" em
        $products = $this->get('doctrine')
            ->getRepository('AcmeStoreBundle:Product')
            ->findAll()
        ;

        // Explicit way to deal with the "default" em
        $products = $this->get('doctrine')
            ->getRepository('AcmeStoreBundle:Product', 'default')
            ->findAll()
        ;

        // Retrieves a repository managed by the "customer" em
        $customers = $this->get('doctrine')
            ->getRepository('AcmeCustomerBundle:Customer', 'customer')
            ->findAll()
        ;
    }
}
```

