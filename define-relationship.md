# 如何定义虚拟类和接口之间的关系

Bundles 的目标之一是创建一个不具有多个（如果有的话）依赖关系的谨慎的功能 bundles，允许您在其他应用程序中使用该功能，而不包括不必要的项目。

Doctrine2.2 包括一个新的实用工具，称为 **ResolveTargetEntityListener**，通过截取 Doctrine 内部一定的调用并且在运行时重写您元数据映射的 **targetEntity** 参数。这意思是在您的 bundle 里，您可以在您的映射中使用一个接口或者虚拟类，并期望在运行时正确映射到一个具体的实体。

这个功能允许您在定义不同实体间的关系，而不让它们很难依赖。

## 背景

假设您有一个 invoiceBundle 提供进销存功能并且 CustomerBundle 包含客户管理工具。您想保持它们分开的状态，因为它们可以在分开的状态下用于其他系统，但对于您的应用程序，您想将其放在一起使用。

在这种情况下，您有一个与一个不存在对象相关的 **Invoice** 实体 **InvoiceSubjectInterface**。目标是让 **ResolveTargetEntityListener** 来取代任何提及与实体对象实现该接口的接口。

## 设置

本文章使用以下两种基本实体（简洁但不完整）来解释如何设置并使用 **ResolveTargetEntityListener**。

Customer 实体：

```
// src/Acme/AppBundle/Entity/Customer.php

namespace Acme\AppBundle\Entity;

use Doctrine\ORM\Mapping as ORM;
use Acme\CustomerBundle\Entity\Customer as BaseCustomer;
use Acme\InvoiceBundle\Model\InvoiceSubjectInterface;

/**
 * @ORM\Entity
 * @ORM\Table(name="customer")
 */
class Customer extends BaseCustomer implements InvoiceSubjectInterface
{
    // In this example, any methods defined in the InvoiceSubjectInterface
    // are already implemented in the BaseCustomer
}
```

Invoice 实体：

```
// src/Acme/InvoiceBundle/Entity/Invoice.php

namespace Acme\InvoiceBundle\Entity;

use Doctrine\ORM\Mapping AS ORM;
use Acme\InvoiceBundle\Model\InvoiceSubjectInterface;

/**
 * Represents an Invoice.
 *
 * @ORM\Entity
 * @ORM\Table(name="invoice")
 */
class Invoice
{
    /**
     * @ORM\ManyToOne(targetEntity="Acme\InvoiceBundle\Model\InvoiceSubjectInterface")
     * @var InvoiceSubjectInterface
     */
    protected $subject;
}
```

InvoiceSubjectInterface：

```
// src/Acme/InvoiceBundle/Model/InvoiceSubjectInterface.php

namespace Acme\InvoiceBundle\Model;

/**
 * An interface that the invoice Subject object should implement.
 * In most circumstances, only a single object should implement
 * this interface as the ResolveTargetEntityListener can only
 * change the target to a single object.
 */
interface InvoiceSubjectInterface
{
    // List any additional methods that your InvoiceBundle
    // will need to access on the subject so that you can
    // be sure that you have access to those methods.

    /**
     * @return string
     */
    public function getName();
}
```

接下来，您需要配置监听器，告知DoctrineBundle 关于配置的事情：

YAML

```
# app/config/config.yml
doctrine:
    # ...
    orm:
        # ...
        resolve_target_entities:
            Acme\InvoiceBundle\Model\InvoiceSubjectInterface: Acme\AppBundle\Entity\Customer
```

XML

```
<!-- app/config/config.xml -->
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:doctrine="http://symfony.com/schema/dic/doctrine"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                        http://symfony.com/schema/dic/doctrine http://symfony.com/schema/dic/doctrine/doctrine-1.0.xsd">

    <doctrine:config>
        <doctrine:orm>
            <!-- ... -->
            <doctrine:resolve-target-entity interface="Acme\InvoiceBundle\Model\InvoiceSubjectInterface">Acme\AppBundle\Entity\Customer</doctrine:resolve-target-entity>
        </doctrine:orm>
    </doctrine:config>
</container>
```

PHP

```
// app/config/config.php
$container->loadFromExtension('doctrine', array(
    'orm' => array(
        // ...
        'resolve_target_entities' => array(
            'Acme\InvoiceBundle\Model\InvoiceSubjectInterface' => 'Acme\AppBundle\Entity\Customer',
        ),
    ),
));
```

## 结语

有了 **ResolveTargetEntityListener**，您可以分离您的 bundles，让它们自己保持合用的状态，但是仍能够定义不同对象之间的关系。通过使用这种方法，您的 bundles 会更容易保持独立。

