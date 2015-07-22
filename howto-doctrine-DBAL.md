# 如何使用 Doctrine DBAL

> 这篇文章关于 Doctrine DBAL。一般来说，你将会使用更高级别的 Doctrine ORM 层次，它仅使用幕后的 DBAL 真正地与数据库交流。要阅读更多关于 Doctrine ORM，参见“[数据库与 Doctrine](http://symfony.com/doc/current/book/doctrine.html)”。

[Doctrine](http://www.doctrine-project.org/) 数据库抽象层（DBAL）是一个抽象层面，位于 [PDO](http://php.net/pdo) 的顶端，并且提供一个直观的和灵活的 API，用于与最流行的关系数据库进行通信。换句话说，DBAL 库使执行查询和执行其他数据库操作很容易。

> 阅读官方 Doctrine [DBAL 文档](http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/index.html)来学习更多关于 Doctrine DBAL 库的细节和功能。

一开始，配置数据库连接参数：

YAML:

```
# app/config/config.yml
doctrine:
    dbal:
        driver:   pdo_mysql
        dbname:   Symfony
        user:     root
        password: null
        charset:  UTF8
        server_version: 5.6
```

XML:

```
<!-- app/config/config.xml -->
<doctrine:config>
    <doctrine:dbal
        name="default"
        dbname="Symfony"
        user="root"
        password="null"
        charset="UTF8"
        server-version="5.6"
        driver="pdo_mysql"
    />
</doctrine:config>
```

PHP:

```
// app/config/config.php
$container->loadFromExtension('doctrine', array(
    'dbal' => array(
        'driver'    => 'pdo_mysql',
        'dbname'    => 'Symfony',
        'user'      => 'root',
        'password'  => null,
        'charset'   => 'UTF8',
        'server_version' => '5.6',
    ),
));
```

想要完整的 DBAL 配置选项或者学习如何配置多种连接，参见 [Doctrine DBAL 配置](http://symfony.com/doc/current/reference/configuration/doctrine.html#reference-dbal-configuration)。

然后你可以通过访问 **database_connection** 服务来访问 Doctrine DBAL 连接：

```
class UserController extends Controller
{
    public function indexAction()
    {
        $conn = $this->get('database_connection');
        $users = $conn->fetchAll('SELECT * FROM users');

        // ...
    }
}
```

## 注册自定义映射类型

你可以通过 Symfony 配置来注册自定义映射类型。它们将会被添加到所有的配置连接中。想知道更多关于自定义映射类型的信息，阅读它们文档的 Doctrine [自定义映射类型](http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/types.html#custom-mapping-types)部分。

YAML:

```
# app/config/config.yml
doctrine:
    dbal:
        types:
            custom_first:  AppBundle\Type\CustomFirst
            custom_second: AppBundle\Type\CustomSecond
```

XML:

```
<!-- app/config/config.xml -->
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:doctrine="http://symfony.com/schema/dic/doctrine"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                        http://symfony.com/schema/dic/doctrine http://symfony.com/schema/dic/doctrine/doctrine-1.0.xsd">

    <doctrine:config>
        <doctrine:dbal>
            <doctrine:type name="custom_first" class="AppBundle\Type\CustomFirst" />
            <doctrine:type name="custom_second" class="AppBundle\Type\CustomSecond" />
        </doctrine:dbal>
    </doctrine:config>
</container>
```

PHP:

```
// app/config/config.php
$container->loadFromExtension('doctrine', array(
    'dbal' => array(
        'types' => array(
            'custom_first'  => 'AppBundle\Type\CustomFirst',
            'custom_second' => 'AppBundle\Type\CustomSecond',
        ),
    ),
));
```

## 在 SchemaTool 中注册自定义映射类型

SchemaTool 被用于检测数据库来比较模式。为了完成这个任务，需要知道每一个数据库需要的映射类型是什么。注册新的可以通过配置来完成。

现在，将 ENUM 类型（默认情况下不由 DBAL 支持）映射到 **string** 映射类型：

YAML:

```
# app/config/config.yml
doctrine:
    dbal:
       mapping_types:
          enum: string
```

XML:

```

<!-- app/config/config.xml -->
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:doctrine="http://symfony.com/schema/dic/doctrine"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                        http://symfony.com/schema/dic/doctrine http://symfony.com/schema/dic/doctrine/doctrine-1.0.xsd">

    <doctrine:config>
        <doctrine:dbal>
             <doctrine:mapping-type name="enum">string</doctrine:mapping-type>
        </doctrine:dbal>
    </doctrine:config>
</container>
```

PHP:

```
// app/config/config.php
$container->loadFromExtension('doctrine', array(
    'dbal' => array(
       'mapping_types' => array(
          'enum'  => 'string',
       ),
    ),
));

```
