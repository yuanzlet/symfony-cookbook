# 如何用 &quot;inherit-data&quot; 减少代码冗余

>**inherit_data** 选项是在 Symfony 2.3 中引入的。在之前的版本中，它叫做 **virtual**。  

在你有些从不同的实体中重复的字段时 **inherit_data** 表单字段选项可以很有用的。举例来说，假设你有两个实体，一个是**公司**另一个是**顾客**：  

```
// src/AppBundle/Entity/Company.php
namespace AppBundle\Entity;

class Company
{
    private $name;
    private $website;

    private $address;
    private $zipcode;
    private $city;
    private $country;
}
```

```
// src/AppBundle/Entity/Customer.php
namespace AppBundle\Entity;

class Customer
{
    private $firstName;
    private $lastName;

    private $address;
    private $zipcode;
    private $city;
    private $country;
}
```

正如你所看到的，每个实体共享了一些字段：**地址**，**邮编**，**城市**，**乡村**。  

从给这些实体建立表单开始，**CompanyType** 和 **CustomerType**：  

```
// src/AppBundle/Form/Type/CompanyType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;

class CompanyType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('name', 'text')
            ->add('website', 'text');
    }
}
```

```
// src/AppBundle/Form/Type/CustomerType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Form\AbstractType;

class CustomerType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('firstName', 'text')
            ->add('lastName', 'text');
    }
}
```

代替在两个表单中包含这些**地址**，**邮编**，**城市**，**乡村**的重复字段，你可以创建一个名为**位置类型**的第三个表单：  

```
// src/AppBundle/Form/Type/LocationType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class LocationType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('address', 'textarea')
            ->add('zipcode', 'text')
            ->add('city', 'text')
            ->add('country', 'text');
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'inherit_data' => true
        ));
    }

    public function getName()
    {
        return 'location';
    }
}
```

位置表单有一个很有趣的选项设置，也就是 **inherit_data**。这个选项使得表单可以从它的父表单那里继承数据。如果嵌入到公司表单中，位置表单的字段将会访问**公司**实体的属性。如果嵌入到顾客表单中，位置表单的字段将会访问**顾客**实体的属性。很简单，有木有？  

>代替在**位置类型**中设置 **inherit_data** 选项，你也可以（就像其他选项一样）将它传递到 **$builder->add()** 的第三变元中。  

最后，通过将位置表单添加到你的原始表单中来完成这个工作：  

```
// src/AppBundle/Form/Type/CompanyType.php
public function buildForm(FormBuilderInterface $builder, array $options)
{
    // ...

    $builder->add('foo', new LocationType(), array(
        'data_class' => 'AppBundle\Entity\Company'
    ));
}
```

```
// src/AppBundle/Form/Type/CustomerType.php
public function buildForm(FormBuilderInterface $builder, array $options)
{
    // ...

    $builder->add('bar', new LocationType(), array(
        'data_class' => 'AppBundle\Entity\Customer'
    ));
}
```

就是这样！你已经提取重复的字段并且将它定义到一个分离位置的当你需要用的时候就能重复利用的表单中了。  

>具有 **inherit_data** 选项设置的表单不能有 **\*_SET_DATA** 事件监听器。  


