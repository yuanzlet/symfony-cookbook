# 如何测试 Doctrine 仓库


一个 Symfony 项目中的 Doctrine 仓库（repositories）测试是不被推荐的。当您处理一个仓库（repository）的时候，您真的在处理一些面对真正的数据库连接的东西。

幸运的是，您可以很容易地测试您对真实数据库的查询（queries），如下所述。

## 功能测试

如果您需要确实地执行一个查询，您需要启动（boot）内核（kernel）以获得一个有效的链接。在这种情况下您会继承（extend）**KernelTestCase**，这可以使一切变得简单：

```PHP
// src/Acme/StoreBundle/Tests/Entity/ProductRepositoryFunctionalTest.php
namespace Acme\StoreBundle\Tests\Entity;

use Symfony\Bundle\FrameworkBundle\Test\KernelTestCase;

class ProductRepositoryFunctionalTest extends KernelTestCase
{
    /**
     * @var \Doctrine\ORM\EntityManager
     */
    private $em;

    /**
     * {@inheritDoc}
     */
    public function setUp()
    {
        self::bootKernel();
        $this->em = static::$kernel->getContainer()
            ->get('doctrine')
            ->getManager()
        ;
    }

    public function testSearchByCategoryName()
    {
        $products = $this->em
            ->getRepository('AcmeStoreBundle:Product')
            ->searchByCategoryName('foo')
        ;

        $this->assertCount(1, $products);
    }

    /**
     * {@inheritDoc}
     */
    protected function tearDown()
    {
        parent::tearDown();
        $this->em->close();
    }
}
```
