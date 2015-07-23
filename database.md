# 如何测试与数据库交互的代码



如果您的代码和数据库交互，例如从中读取数据或者写入数据，您需要把这个考虑进去来调整测试。有很多方法可以解决这个问题。您可以创建一个 Repository 的仿制品，然后用它来返回预期的对象。在功能测试中，您可能需要准备一个有预定值的测试数据库来确保您的测试始终有相同的数据来使用。

> 如果您想直接测试您的查询（queries），看[如何测试 Doctrine 仓库](http://symfony.com/doc/current/cookbook/testing/doctrine.html)。

## 在单元测试中模拟 Repository

如果您想测试一个独立的依赖于一个 Doctrine repository 的代码，您需要模拟 **Repository**。通常，您将 **EntityManager** 注入的您的类中然后使用它获得 repository。这使事情变得有一些困难，因为您需要同时模拟 **EntityManager** 和您的 repository 类。

> 通过将您的 repository 作为 [factory service](http://symfony.com/doc/current/components/dependency_injection/factories.html) 注册来直接注入 repository 是可能的（也是一个好主意）。这使设置需要多一些工作，但使测试更加容易，因为您只需要模拟 repository。

假设您要测试的类看起来像这样：

```PHP
namespace AppBundle\Salary;

use Doctrine\Common\Persistence\ObjectManager;

class SalaryCalculator
{
    private $entityManager;

    public function __construct(ObjectManager $entityManager)
    {
        $this->entityManager = $entityManager;
    }

    public function calculateTotalSalary($id)
    {
        $employeeRepository = $this->entityManager
            ->getRepository('AppBundle:Employee');
        $employee = $employeeRepository->find($id);

        return $employee->getSalary() + $employee->getBonus();
    }
}
```

由于 **ObjectManager** 通过构造函数被注入类里面，在一个测试内很容易传递（pass）一个模拟类：

```PHP
use AppBundle\Salary\SalaryCalculator;

class SalaryCalculatorTest extends \PHPUnit_Framework_TestCase
{
    public function testCalculateTotalSalary()
    {
        // First, mock the object to be used in the test
        $employee = $this->getMock('\AppBundle\Entity\Employee');
        $employee->expects($this->once())
            ->method('getSalary')
            ->will($this->returnValue(1000));
        $employee->expects($this->once())
            ->method('getBonus')
            ->will($this->returnValue(1100));

        // Now, mock the repository so it returns the mock of the employee
        $employeeRepository = $this
            ->getMockBuilder('\Doctrine\ORM\EntityRepository')
            ->disableOriginalConstructor()
            ->getMock();
        $employeeRepository->expects($this->once())
            ->method('find')
            ->will($this->returnValue($employee));

        // Last, mock the EntityManager to return the mock of the repository
        $entityManager = $this
            ->getMockBuilder('\Doctrine\Common\Persistence\ObjectManager')
            ->disableOriginalConstructor()
            ->getMock();
        $entityManager->expects($this->once())
            ->method('getRepository')
            ->will($this->returnValue($employeeRepository));

        $salaryCalculator = new SalaryCalculator($entityManager);
        $this->assertEquals(2100, $salaryCalculator->calculateTotalSalary(1));
    }
}
```

在这个例子中，您由内而外的构造模拟，首先创建由 **Repository** 返回的 employee，它本身被 **EntityManager** 返回。这种方式中，没有真正的类参加到测试中。

## 为功能测试更改数据库设置

如果您有功能测试，您希望他们与一个真实的数据库进行交互。大多数时候您需要用专用的数据库连接来保证当您开发应用程序时不会重写您输入的数据，也可以在每个测试之前清空数据库。

要做到这一点，您可以指定一个数据库配置，覆盖默认的配置：

```YAML
# app/config/config_test.yml
doctrine:
    # ...
    dbal:
        host:     localhost
        dbname:   testdb
        user:     testdb
        password: testdb
```

```XML
<!-- app/config/config_test.xml -->
<doctrine:config>
    <doctrine:dbal
        host="localhost"
        dbname="testdb"
        user="testdb"
        password="testdb"
    />
</doctrine:config>
```

```PHP
// app/config/config_test.php
$configuration->loadFromExtension('doctrine', array(
    'dbal' => array(
        'host'     => 'localhost',
        'dbname'   => 'testdb',
        'user'     => 'testdb',
        'password' => 'testdb',
    ),
));
```

确保您的数据库在 localhost 上运行，并且有数据库和用户凭证设置（database and user credentials set up）的定义。
