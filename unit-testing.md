# (form)如何对表单单元测试

表单组件包含三个核心的对象：一个是表单类型（实现 [FormTypeInterface](http://api.symfony.com/2.7/Symfony/Component/Form/FormTypeInterface.html)），[Form](http://api.symfony.com/2.7/Symfony/Component/Form/Form.html) 以及 the [FormView](http://api.symfony.com/2.7/Symfony/Component/Form/FormView.html)。  

经常被程序员操作的唯一的类是表单类型类，这个类作为表单的蓝图。它被用来生成 **Form** 以及 **FormView**。你可以通过模拟它和工厂的交互作用来直接测试它，但是这个会很复杂。最好的办法就是将它传递到 FormFactory 这样就会像真正在应用程序中使用一样。这对于 bootstrap 很简单并且你可以信任 Symfony 组件测试这个足够用了。  

这里有一个类你可以直接用它来进行简单的 FormTypes 测试：[TypeTestCase](http://api.symfony.com/2.7/Symfony/Component/Form/Test/TypeTestCase.html)。它是用来测试核心类型的同时你也可以用它测试你的类型。  

>在 2.3 中 **TypeTestCase** 已经转移到 **Symfony\Component\Form\Test** 命名空间中了。在之前的版本中，这个类位于 **Symfony\Component\Form\Tests\Extension\Core\Type**。

>取决于你安装你的 Symfony 的方法或者 Symfony 表单测试组件可能没有被下载。这种情况下可以使用 Composer 的 **--prefer-source** 选项。  

## 基础

最简单的 **TypeTestCase** 启用如下所示：  

```
// src/Acme/TestBundle/Tests/Form/Type/TestedTypeTest.php
namespace Acme\TestBundle\Tests\Form\Type;

use Acme\TestBundle\Form\Type\TestedType;
use Acme\TestBundle\Model\TestObject;
use Symfony\Component\Form\Test\TypeTestCase;

class TestedTypeTest extends TypeTestCase
{
    public function testSubmitValidData()
    {
        $formData = array(
            'test' => 'test',
            'test2' => 'test2',
        );

        $type = new TestedType();
        $form = $this->factory->create($type);

        $object = TestObject::fromArray($formData);

        // submit the data to the form directly
        $form->submit($formData);

        $this->assertTrue($form->isSynchronized());
        $this->assertEquals($object, $form->getData());

        $view = $form->createView();
        $children = $view->children;

        foreach (array_keys($formData) as $key) {
            $this->assertArrayHasKey($key, $children);
        }
    }
}
```

那么，它怎么测试呢？下面就来详细讲解。  

首先你需要区分 **FormType** 是否编制。这包括基本的类的继承，**buildForm** 功能以及选项解决方案。这应该是你写的第一个测试：  

```
$type = new TestedType();
$form = $this->factory->create($type);
```  

这个测试检查了你的表单使用的数据翻译器没有失败的。[isSynchronized()](http://api.symfony.com/2.7/Symfony/Component/Form/FormInterface.html#isSynchronized()) 方法只是设置成 **false** 如果数据转换器出现例外：  

```
$form->submit($formData);
$this->assertTrue($form->isSynchronized());
```

>不要测试验证：这个被监听器实施，这个在测试环境不活跃并且它依赖于验证配置。作为替代直接对你的定制的限制进行单元测试。  

接下来，核实表单的提交和映射。下列的测试检查了所有的字段是否正确被指定：  

```
$this->assertEquals($object, $form->getData());
```

最后，检查 **FormView** 的创建。你应当检查是否你想要展示的所有的控件是否在子属性上有用：  

```
$view = $form->createView();
$children = $view->children;

foreach (array_keys($formData) as $key) {
    $this->assertArrayHasKey($key, $children);
}
```

## 添加你的表单依靠的类型

你的表单可能依赖于其他被定义为服务的类型。可能像下面这样：  

```
// src/Acme/TestBundle/Form/Type/TestedType.php

// ... the buildForm method
$builder->add('acme_test_child_type');
```

为了正确的建立你的表单，你需要在你的测试中使得类型对于你的表单工厂可用。最简单的方法就是在创建父表单时使用 **PreloadedExtension** 类手动注册：  

```
// src/Acme/TestBundle/Tests/Form/Type/TestedTypeTests.php
namespace Acme\TestBundle\Tests\Form\Type;

use Acme\TestBundle\Form\Type\TestedType;
use Acme\TestBundle\Model\TestObject;
use Symfony\Component\Form\Test\TypeTestCase;
use Symfony\Component\Form\PreloadedExtension;

class TestedTypeTest extends TypeTestCase
{
    protected function getExtensions()
    {
        $childType = new TestChildType();
        return array(new PreloadedExtension(array(
            $childType->getName() => $childType,
        ), array()));
    }

    public function testSubmitValidData()
    {
        $type = new TestedType();
        $form = $this->factory->create($type);

        // ... your test
    }
}
```

>确保你添加的子类型被很好地测试。否则你正在测试的表单的子表单将会出现错误。  

## 添加自定义扩展

你使用由[表单扩展](http://symfony.com/doc/current/cookbook/form/create_form_type_extension.html)添加的一些选项这种情况检查出现。其中的一种情况就是 **ValidatorExtension** 的 **invalid_message** 选项。**TypeTestCase** 只加载核心表单扩展所以一个“不可用的选项”例外将会被释放如果你想要使用它测试基于其他扩展的类。你需要将这些扩展添加到工厂对象：  

```
// src/Acme/TestBundle/Tests/Form/Type/TestedTypeTests.php
namespace Acme\TestBundle\Tests\Form\Type;

use Acme\TestBundle\Form\Type\TestedType;
use Acme\TestBundle\Model\TestObject;
use Symfony\Component\Form\Test\TypeTestCase;
use Symfony\Component\Form\Forms;
use Symfony\Component\Form\FormBuilder;
use Symfony\Component\Form\Extension\Validator\Type\FormTypeValidatorExtension;
use Symfony\Component\Validator\ConstraintViolationList;

class TestedTypeTest extends TypeTestCase
{
    protected function setUp()
    {
        parent::setUp();

        $validator = $this->getMock('\Symfony\Component\Validator\Validator\ValidatorInterface');
        $validator->method('validate')->will($this->returnValue(new ConstraintViolationList()));

        $this->factory = Forms::createFormFactoryBuilder()
            ->addExtensions($this->getExtensions())
            ->addTypeExtension(
                new FormTypeValidatorExtension(
                    $validator
                )
            )
            ->addTypeGuesser(
                $this->getMockBuilder(
                    'Symfony\Component\Form\Extension\Validator\ValidatorTypeGuesser'
                )
                    ->disableOriginalConstructor()
                    ->getMock()
            )
            ->getFormFactory();

        $this->dispatcher = $this->getMock('Symfony\Component\EventDispatcher\EventDispatcherInterface');
        $this->builder = new FormBuilder(null, null, $this->dispatcher, $this->factory);
    }

    // ... your tests
}
```  

## 测试不同集合的数据

如果你不熟悉 PHPUnit 的 [数据提供](http://www.phpunit.de/manual/current/en/writing-tests-for-phpunit.html#writing-tests-for-phpunit.data-providers)，这可能是使用它们的好机会：  

```
// src/Acme/TestBundle/Tests/Form/Type/TestedTypeTests.php
namespace Acme\TestBundle\Tests\Form\Type;

use Acme\TestBundle\Form\Type\TestedType;
use Acme\TestBundle\Model\TestObject;
use Symfony\Component\Form\Test\TypeTestCase;

class TestedTypeTest extends TypeTestCase
{

    /**
     * @dataProvider getValidTestData
     */
    public function testForm($data)
    {
        // ... your test
    }

    public function getValidTestData()
    {
        return array(
            array(
                'data' => array(
                    'test' => 'test',
                    'test2' => 'test2',
                ),
            ),
            array(
                'data' => array(),
            ),
            array(
                'data' => array(
                    'test' => null,
                    'test2' => null,
                ),
            ),
        );
    }
}
```

上述代码将以三种不同的数据集合运行你的测试三次。这就允许固定的测试去耦合并且很容易测试多重数据集合。  

你也可以传递另外一个变元，例如 boolean 如果表单必须给定的数据集合同步或者不同步表单等等。  


