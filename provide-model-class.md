# 如何为多个 Doctrine 的实现提供模型类

当创建一个 bundlie不仅可以用于 Doctrine ORM，也可以用于 CouchDB ODM，MongoDB ODM 或者 PHPCR ODM，您应该是仍然只能使用一种模型类。Doctrine bundles 提供了一个编译器为您的模型类注册映射。

对于不可再使用的 bundle，最简单的选择就是将您的模型类放在默认的位置 **Entity** 在 Doctrine ORM 中或者将 **Document** 放在 ODM 之一。对于可以再使用的 bundle，复制模型类不只是为了获取自动映射，而是使用扫描编译器。

2.3 基本映射扫描编译器在 Symfony2.3中介绍。Doctrine bundles 在 DoctrineBundle >= 1.3.0, MongoDBBundle >= 3.0.0, PHPCRBundle >= 1.0.0 支持它，并且当 [CouchDB Mapping Compiler Pass pull request](https://github.com/doctrine/DoctrineCouchDBBundle/pull/27) 合并了，（无版本） CouchDBBundle 支持扫描编译器。

2.6 Symfony2.6中介绍了支持定义命名空间别名。用老版本的 Symfony 来定义别名是安全的，因为别名是 **createXmlMappingDriver** 的最后一个参数，并且如果参数不存在的话，就会被 PHP 忽略。

在您的 bundles 类中，编写以下代码来注册扫描编译器。这是为 CmfRoutingBundle 编写的，所以其中部分需要按照您的情况进行调整：

```
use Doctrine\Bundle\DoctrineBundle\DependencyInjection\Compiler\DoctrineOrmMappingsPass;
use Doctrine\Bundle\MongoDBBundle\DependencyInjection\Compiler\DoctrineMongoDBMappingsPass;
use Doctrine\Bundle\CouchDBBundle\DependencyInjection\Compiler\DoctrineCouchDBMappingsPass;
use Doctrine\Bundle\PHPCRBundle\DependencyInjection\Compiler\DoctrinePhpcrMappingsPass;

class CmfRoutingBundle extends Bundle
{
    public function build(ContainerBuilder $container)
    {
        parent::build($container);
        // ...

        $modelDir = realpath(__DIR__.'/Resources/config/doctrine/model');
        $mappings = array(
            $modelDir => 'Symfony\Cmf\RoutingBundle\Model',
        );

        $ormCompilerClass = 'Doctrine\Bundle\DoctrineBundle\DependencyInjection\Compiler\DoctrineOrmMappingsPass';
        if (class_exists($ormCompilerClass)) {
            $container->addCompilerPass(
                DoctrineOrmMappingsPass::createXmlMappingDriver(
                    $mappings,
                    array('cmf_routing.model_manager_name'),
                    'cmf_routing.backend_type_orm',
                    array('CmfRoutingBundle' => 'Symfony\Cmf\RoutingBundle\Model')
            ));
        }

        $mongoCompilerClass = 'Doctrine\Bundle\MongoDBBundle\DependencyInjection\Compiler\DoctrineMongoDBMappingsPass';
        if (class_exists($mongoCompilerClass)) {
            $container->addCompilerPass(
                DoctrineMongoDBMappingsPass::createXmlMappingDriver(
                    $mappings,
                    array('cmf_routing.model_manager_name'),
                    'cmf_routing.backend_type_mongodb',
                    array('CmfRoutingBundle' => 'Symfony\Cmf\RoutingBundle\Model')
            ));
        }

        $couchCompilerClass = 'Doctrine\Bundle\CouchDBBundle\DependencyInjection\Compiler\DoctrineCouchDBMappingsPass';
        if (class_exists($couchCompilerClass)) {
            $container->addCompilerPass(
                DoctrineCouchDBMappingsPass::createXmlMappingDriver(
                    $mappings,
                    array('cmf_routing.model_manager_name'),
                    'cmf_routing.backend_type_couchdb',
                    array('CmfRoutingBundle' => 'Symfony\Cmf\RoutingBundle\Model')
            ));
        }

        $phpcrCompilerClass = 'Doctrine\Bundle\PHPCRBundle\DependencyInjection\Compiler\DoctrinePhpcrMappingsPass';
        if (class_exists($phpcrCompilerClass)) {
            $container->addCompilerPass(
                DoctrinePhpcrMappingsPass::createXmlMappingDriver(
                    $mappings,
                    array('cmf_routing.model_manager_name'),
                    'cmf_routing.backend_type_phpcr',
                    array('CmfRoutingBundle' => 'Symfony\Cmf\RoutingBundle\Model')
            ));
        }
    }
}
```

注意 [class_exists](http://php.net/manual/en/function.class-exists.php) 的核对。这是很关键的，因为您不想您的 bundle 在所有的 Doctrine bundles中有一个很困难的依赖，而是让用户决定使用哪一个。

扫描编译器为 Doctrine 提供的所有驱动提供工厂方法：Annotations, XML, Yaml, PH和 StaticPHP。参数是：

•	一个到命名空间的绝对路径的映射/散列；

•	一组容器参数，您的 bundle 用以指定其使用的原则管理器的名称。在以上的例子中，CmfRoutingBundle 存储在 **cmf_routing.model_manager_name** 参数下使用的管理器名字。扫描编译器将会附加 Doctrine 使用的参数来指定默认管理器的名称。使用找到的第一个参数，并且管理器注册该映射；

•	一个可选择的容器参数名称将由扫描编译器使用来决定此 Doctrine 类型是否被使用。如果您的用户安装了不止一种 Doctrine bundle，那这就很相关了，但是您的bundle 只能在一种 Doctrine 情况下使用。

•	一个命名控件的映射/散列别名。这应该是 Doctrine 自动映射使用的同一惯例。在以上的例子中，这允许用户来调用 **$om->getRepository('CmfRoutingBundle:Route')**。

工厂方法使用 Doctrine 的 **SymfonyFileLocator**，意味着如果它们不包含像文件名称那样的完整的命名空间，它将只能看到 XML 和 YML 映射文件。这是由设计 **SymfonyFileLocator** 简化了事情，假定文件只是作为文件名的“简短”版本（例如 **BlogPost.orm.xml**）。

如果您也需要映射一个基本类，您可以注册一个像这个的 **DefaultFileLocator** 扫描编译器。代码从 **DoctrineOrmMappingsPass** 中获取，并且适用于 **DefaultFileLocator** 而不是 **SymfonyFileLocator**：

```
private function buildMappingCompilerPass()
{
    $arguments = array(array(realpath(__DIR__ . '/Resources/config/doctrine-base')), '.orm.xml');
    $locator = new Definition('Doctrine\Common\Persistence\Mapping\Driver\DefaultFileLocator', $arguments);
    $driver = new Definition('Doctrine\ORM\Mapping\Driver\XmlDriver', array($locator));

    return new DoctrineOrmMappingsPass(
        $driver,
        array('Full\Namespace'),
        array('your_bundle.manager_name'),
        'your_bundle.orm_enabled'
    );
}
```

记住您不需要提供一个命名空间别名除非您的用户希望访问 Doctrine 的基本类。

现在将您的映射文件以有效的类名称放入 **/Resources/config/doctrine-base** 中，由 **.** 而不是 **/** 分开，例如 **Other.Namespace.Model.Name.orm.xml**。您不可以混淆两者，要不然 **SymfonyFileLocator** 就会搞糊涂了。



