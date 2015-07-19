# (configuration)如何在数据库中使用 PdoSessionHandler 存储 Sessions

> 在 Symfony 2.6 中有一个后向兼容：数据库模式稍作改变。更多细节请见 [Symfony 2.6 的改变](http://symfony.com/doc/current/cookbook/configuration/pdo_session_storage.html#pdo-session-handle-26-changes)。  

默认的 Symfony 的 session 存储将 session 信息写进文件。大多数的中到大型的网页使用一个数据库储存 session 的值而不是文件，因为数据库很好用并且适应多线程网页服务器环境。  

Symfony 有一个内建的数据库 session 的存储解决方案名为 [PdoSessionHandler](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/Session/Storage/Handler/PdoSessionHandler.html)。使用这个，你只需要在主配置文件中改变一些参数：  

YAML:

```YAML
# app/config/config.yml
framework:
    session:
        # ...
        handler_id: session.handler.pdo

services:
    session.handler.pdo:
        class:     Symfony\Component\HttpFoundation\Session\Storage\Handler\PdoSessionHandler
        public:    false
        arguments:
            - "mysql:dbname=mydatabase"
            - { db_username: myuser, db_password: mypassword }
```  

XML:

```XML
<!-- app/config/config.xml -->
<framework:config>
    <framework:session handler-id="session.handler.pdo" cookie-lifetime="3600" auto-start="true"/>
</framework:config>

<services>
    <service id="session.handler.pdo" class="Symfony\Component\HttpFoundation\Session\Storage\Handler\PdoSessionHandler" public="false">
        <argument>mysql:dbname=mydatabase</agruement>
        <argument type="collection">
            <argument key="db_username">myuser</argument>
            <argument key="db_password">mypassword</argument>
        </argument>
    </service>
</services>
```  

PHP:

```PHP
// app/config/config.php
use Symfony\Component\DependencyInjection\Definition;
use Symfony\Component\DependencyInjection\Reference;

$container->loadFromExtension('framework', array(
    ...,
    'session' => array(
        // ...,
        'handler_id' => 'session.handler.pdo',
    ),
));

$storageDefinition = new Definition('Symfony\Component\HttpFoundation\Session\Storage\Handler\PdoSessionHandler', array(
    'mysql:dbname=mydatabase',
    array('db_username' => 'myuser', 'db_password' => 'mypassword')
));
$container->setDefinition('session.handler.pdo', $storageDefinition);
```  

## 设置表和列名称

这将会产生一个有着很多不同列的 **sessions** 表。表的名称以及所有的列名称，可以通过向 **PdoSessionHandler** 传递一个第二数组参数的方式设置：  

YAML:

```YAML
# app/config/config.yml
services:
    # ...
    session.handler.pdo:
        class:     Symfony\Component\HttpFoundation\Session\Storage\Handler\PdoSessionHandler
        public:    false
        arguments:
            - "mysql:dbname=mydatabase"
            - { db_table: sessions, db_username: myuser, db_password: mypassword }
```  

XML:

```XML
<!-- app/config/config.xml -->
<services>
    <service id="session.handler.pdo" class="Symfony\Component\HttpFoundation\Session\Storage\Handler\PdoSessionHandler" public="false">
        <argument>mysql:dbname=mydatabase</agruement>
        <argument type="collection">
            <argument key="db_table">sessions</argument>
            <argument key="db_username">myuser</argument>
            <argument key="db_password">mypassword</argument>
        </argument>
    </service>
</services>
```  

PHP:

```PHP
// app/config/config.php

use Symfony\Component\DependencyInjection\Definition;
// ...

$storageDefinition = new Definition('Symfony\Component\HttpFoundation\Session\Storage\Handler\PdoSessionHandler', array(
    'mysql:dbname=mydatabase',
    array('db_table' => 'sessions', 'db_username' => 'myuser', 'db_password' => 'mypassword')
));
$container->setDefinition('session.handler.pdo', $storageDefinition);
```  

> **db\_lifetime\_col** 是在 Symfony 2.6 中被引进的。2.6 之前的版本并不存在。  

下列这些参数你必须设置：  

**db_table** (默认为 **sessions**)：  
你的数据库中的 session 表的名称；  

**db_id_col** (默认为 **sess_id**)：  
你的 session 表的 id 列的名称（文本类型（128））；  

**db_data_col** (默认为 **sess_data**):  
你的 session 表的 value 列的名称 (二进制大对象);  

**db_time_col** (默认为 **sess_time**):
你的 session 表的 time 列的名称(整型);  

**db_lifetime_col** (默认为 **sess_lifetime**):
T你的 session 表的 lifetime 列的名称(整型).  

## 共享你的数据库连接信息

根据给定的设置，数据库的连接只是为了 session 存储连接而设置的。当你为 session 数据使用分离的数据库时这个是可以的。  

但是如果你想要将 session 数据像你的工程的其它的数据一样储存在同一个数据库中，你可以使用通过引用数据库的 **parameters.yml** 文件的连接设置——相关的参数在如下定义：  

YAML:

```YAML
services:
    session.handler.pdo:
        class:     Symfony\Component\HttpFoundation\Session\Storage\Handler\PdoSessionHandler
        public:    false
        arguments:
            - "mysql:host=%database_host%;port=%database_port%;dbname=%database_name%"
            - { db_username: %database_user%, db_password: %database_password% }
```  

XML:

```XML
<service id="session.handler.pdo" class="Symfony\Component\HttpFoundation\Session\Storage\Handler\PdoSessionHandler" public="false">
    <argument>mysql:host=%database_host%;port=%database_port%;dbname=%database_name%</agruement>
    <argument type="collection">
        <argument key="db_username">%database_user%</argument>
        <argument key="db_password">%database_password%</argument>
    </argument>
</service>
```  

PHP:

```PHP
$storageDefinition = new Definition('Symfony\Component\HttpFoundation\Session\Storage\Handler\PdoSessionHandler', array(
    'mysql:host=%database_host%;port=%database_port%;dbname=%database_name%',
    array('db_username' => '%database_user%', 'db_password' => '%database_password%')
));
``` 

## SQL 语句案例

>## 当升级到 Symfony 2.6 时模式就需要改变了 
>如果你使用 PdoSessionHandler 是 Symfony 2.6 之前的版本然后进行了升级，你的 session 表需要做出一些改变：  
>- 需要添加新的 session lifetime (**sess_lifetime** 默认)整型列；
>- data 列(**sess_data** 默认)需要改成二进制大对象型。  

>更多细节详见下面的 SQL 语句。  

>为了保存以前（2.5 以及更早的）版本的功能，将你的类的名称由 **PdoSessionHandler** 改成 **LegacyPdoSessionHandler**（Symfony 2.6.2 中添加的旧的类）。  

### MySQL

创建新的数据库的表的 SQL 语句如下所示（MySQL）：  

```
CREATE TABLE `sessions` (
    `sess_id` VARBINARY(128) NOT NULL PRIMARY KEY,
    `sess_data` BLOB NOT NULL,
    `sess_time` INTEGER UNSIGNED NOT NULL,
    `sess_lifetime` MEDIUMINT NOT NULL
) COLLATE utf8_bin, ENGINE = InnoDB;
```  

> **二进制大对象**类型的栏目只能储存到 64 kb。如果存储的用户的 session 数据超过这个值，可能就会出现异常或者它们的 session 会被重置。如果你需要更多的存储空间可以考虑使用 **MEDIUMBLOB**。

### PostgreSQL

对于 PostgreSQL，代码如下所示：  

```
CREATE TABLE sessions (
    sess_id VARCHAR(128) NOT NULL PRIMARY KEY,
    sess_data BYTEA NOT NULL,
    sess_time INTEGER NOT NULL,
    sess_lifetime INTEGER NOT NULL
);
```  

### 微软的 SQL Server

对于微软的 SQL Server，代码如下所示：  

```
CREATE TABLE [dbo].[sessions](
    [sess_id] [nvarchar](255) NOT NULL,
    [sess_data] [ntext] NOT NULL,
    [sess_time] [int] NOT NULL,
    [sess_lifetime] [int] NOT NULL,
    PRIMARY KEY CLUSTERED(
        [sess_id] ASC
    ) WITH (
        PAD_INDEX  = OFF,
        STATISTICS_NORECOMPUTE  = OFF,
        IGNORE_DUP_KEY = OFF,
        ALLOW_ROW_LOCKS  = ON,
        ALLOW_PAGE_LOCKS  = ON
    ) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
``` 