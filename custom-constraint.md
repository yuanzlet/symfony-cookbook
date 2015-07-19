# 如何创建一个自定义的验证限制

您可以通过继承一个限制基类 [Constraint](http://api.symfony.com/2.7/Symfony/Component/Validator/Constraint.html) 来创建一个自定义的验证限制，比如，您可以创建一个简单的验证器来检查一个字符串是否只包含字母和数字。  
 
##创建限制类    

首先，您可以通过继承 [Constraint](http://api.symfony.com/2.7/Symfony/Component/Validator/Constraint.html)  来创建一个验证：    

```  
// src/AppBundle/Validator/Constraints/  ContainsAlphanumeric.php  
namespace AppBundle\Validator\Constraints;  

use Symfony\Component\Validator\Constraint;  

/**  
 * @Annotation 
 */  
class ContainsAlphanumeric extends Constraint  
{  
    public $message = 'The string "%string%" contains an illegal character: it can only contain letters or numbers.';  
}  
```  

> 当创建一个新类时，很有必要为新创建的限制类添加上 **@Annotation** 文档注释，通过注释可以增加新建类代码的可读性，使其更好的被使用。您可以在新创建的类中选择公有属性进行注释与说明。

## 创建验证器  

正如您看到一样，一个验证限制类是十分简洁的，并不是通过该限制验证类直接执行验证，而是通过另一个限制验证类 "constraint validator" 中指定的 **validatedBy()** 方法进行验证，在该方法中存在一些默认的简单算法逻辑：

```
// in the base Symfony\Component\Validator\Constraint class  
public function validatedBy()  
{  
    return get_class($this).'Validator';  
} 
```

换句话说，当您创建一个自定义的限制验证类的时候，（例如：**MyConstraint**）当实际执行验证的时候 Symfony 会自动调用另一个类 **MyConstraintValidator** 进行验证。

这个验证类也很简洁，只包括一个 **validate()** 方法：

```
// src/AppBundle/Validator/Constraints/ContainsAlphanumericValidator.php  
namespace AppBundle\Validator\Constraints;  

use Symfony\Component\Validator\Constraint;
use Symfony\Component\Validator\ConstraintValidator;

class ContainsAlphanumericValidator extends ConstraintValidator
{
    public function validate($value, Constraint $constraint)
    {
        if (!preg_match('/^[a-zA-Z0-9]+$/', $value, $matches)) {
            // If you're using the new 2.5 validation API (you probably are!)
            $this->context->buildViolation($constraint->message)
                ->setParameter('%string%', $value)
                ->addViolation();

            // If you're using the old 2.4 validation API
            /*
            $this->context->addViolation(
                $constraint->message,
                array('%string%' => $value)
            );
            */
        }
    }
}
```

在这个验证类中，您并不需要设定一个返回值。相反的是，如果您需要验证的内容是合法的，那么该内容将会通过验证，如果该内容不能被检验通过，那么 ```buildViolation``` 方法会把错误信息作为参数传递给 [ConstraintViolationBuilderInterface](http://api.symfony.com/2.7/Symfony/Component/Validator/Constraint.html) 作为一个实例进行调用，然后 ```addViolation``` 方法会把不合法的部分标注到您需要检测的内容中。

## 使用新创建的限制验证 

就和使用 Symfony 本身提供的接口一样，使用一个自定义的验证限制类也同样很简单：

Annotations:

```  
// src/AppBundle/Entity/AcmeEntity.php
use Symfony\Component\Validator\Constraints as Assert;
use AppBundle\Validator\Constraints as AcmeAssert;

class AcmeEntity
{
    // ...

    /**
     * @Assert\NotBlank
     * @AcmeAssert\ContainsAlphanumeric
     */
    protected $name;

    // ...
}
``` 

YAML:

```
# src/AppBundle/Resources/config/validation.yml
AppBundle\Entity\AcmeEntity:
    properties:
        name:
            - NotBlank: ~
            - AppBundle\Validator\Constraints\ContainsAlphanumeric: ~
```

XML:

```
<!-- src/AppBundle/Resources/config/validation.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<constraint-mapping xmlns="http://symfony.com/schema/dic/constraint-mapping"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/constraint-mapping http://symfony.com/schema/dic/constraint-mapping/constraint-mapping-1.0.xsd">

    <class name="AppBundle\Entity\AcmeEntity">
        <property name="name">
            <constraint name="NotBlank" />
            <constraint name="AppBundle\Validator\Constraints\ContainsAlphanumeric" />
        </property>
    </class>
</constraint-mapping>
```

PHP:

```：
// src/AppBundle/Entity/AcmeEntity.php
use Symfony\Component\Validator\Mapping\ClassMetadata;
use Symfony\Component\Validator\Constraints\NotBlank;
use AppBundle\Validator\Constraints\ContainsAlphanumeric;

class AcmeEntity
{
    public $name;

    public static function loadValidatorMetadata(ClassMetadata $metadata)
    {
        $metadata->addPropertyConstraint('name', new NotBlank());
        $metadata->addPropertyConstraint('name', new ContainsAlphanumeric());
    }
}
```

如果在您定义的类中含有可选择的属性时，您应该在创建自定义类的时候就以公有的方式声明这些属性，那么这些可选择的属性就可以像使用 核心 Symfony 约束类中的属性一样使用它们。   

## 带有依赖关系的限制验证 

如果您的限制验证具有依赖关系，就如同一个数据库的连接操作，那么它将被视为依赖注入与服务定位器中的一个服务项，那么这个服务项必须包含 **validator.constraint_validator** 标签和 **alias** 属性。

YAML:

```
# app/config/services.yml
services:
    validator.unique.your_validator_name:
        class: Fully\Qualified\Validator\Class\Name
        tags:
            - { name: validator.constraint_validator, alias: alias_name }
```

XML:

```
public function validatedBy()
{
    return 'alias_name';
}
```

PHP:

```
// app/config/services.php
$container
    ->register('validator.unique.your_validator_name', 'Fully\Qualified\Validator\Class\Name')
    ->addTag('validator.constraint_validator', array('alias' => 'alias_name'));
```

这时候您新建的类就可以用此别名去引用相应的限制类了:

```
public function validatedBy()
{
    return 'alias_name';
}
```

就如同上文提到的，Symfony 会自动查找以 constraint 命名并且添加了验证的类。如果您的约束验证程序被定义为一种服务项，那么您应该覆写 **validatedBy()** 方法来返回您定义该服务项时使用的别名，否则 Symfony 不会使用这个限制验证类的服务项，使得该限制验证类被实例化的时候不会有任何依赖项被注入。

## 限制验证类

一个验证类可以返回一个类作用域的对象属性:

```
public function getTargets()
{
    return self::CLASS_CONSTRAINT;
}
```

验证类中的 **validate()** 方法把这个对象作为它的第一个参数:

```
class ProtocolClassValidator extends ConstraintValidator
{
    public function validate($protocol, Constraint $constraint)
    {
        if ($protocol->getFoo() != $protocol->getBar()) {
            // If you're using the new 2.5 validation API (you probably are!)
            $this->context->buildViolation($constraint->message)
                ->atPath('foo')
                ->addViolation();

            // If you're using the old 2.4 validation API
            /*
            $this->context->addViolationAt(
                'foo',
                $constraint->message,
                array(),
                null
            );
            */
        }
    }
}
```

注意，一个限制验证类是作用于其本身，而不是一个属性:

Annotations:

```
/**
 * @AcmeAssert\ContainsAlphanumeric
 */
class AcmeEntity
{
    // ...
}
```

YAML:

```
# src/AppBundle/Resources/config/validation.yml
AppBundle\Entity\AcmeEntity:
    constraints:
        - AppBundle\Validator\Constraints\ContainsAlphanumeric: ~
```

XML:

```
<!-- src/AppBundle/Resources/config/validation.xml -->
<class name="AppBundle\Entity\AcmeEntity">
    <constraint name="AppBundle\Validator\Constraints\ContainsAlphanumeric" />
</class>
```