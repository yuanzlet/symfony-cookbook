# 如何处理不同的错误级别

有时候，您想通过某些规则和标准来定义并且展示不同的限制验证错误。比如，您建立了一个给用户进行注册的表单，用户需要输入身份信息和用户凭据来完成注册。当用户进行注册的时候，输入用户名和一个安全的密码是必不可少的，但是是否输入用户的银行账户信息并不是强制要求的，用户具有选择权。尽管如此，如果用户在该字段输入了信息，那么您还是需要检查用户在该字段的输入的信息是否是正确有效的，如果输入出现错误，您就得给出不同的错误描述。  

实现该功能一共需要两个步骤：

1. 给限制验证类提供不同级别的错误描述。
2. 根据提供的错误级别来定义输入内容的错误级别。

## 1. 制定错误的级别

> 2.6 在 Symfony 2.6节中介绍了 **payload** 选项。

使用 **payload** 选项来配置每个约束的级别：

Annotations:

```
// src/AppBundle/Entity/User.php
namespace AppBundle\Entity;

use Symfony\Component\Validator\Constraints as Assert;

class User
{
    /**
     * @Assert\NotBlank(payload = {severity = "error"})
     */
    protected $username;

    /**
     * @Assert\NotBlank(payload = {severity = "error"})
     */
    protected $password;

    /**
     * @Assert\Iban(payload = {severity = "warning"})
     */
    protected $bankAccountNumber;
}
```

YAML:

```
# src/AppBundle/Resources/config/validation.yml
AppBundle\Entity\User:
    properties:
        username:
            - NotBlank:
                payload:
                    severity: error
        password:
            - NotBlank:
                payload:
                    severity: error
        bankAccountNumber:
            - Iban:
                payload:
                    severity: warning
```

XML:

```
<!-- src/AppBundle/Resources/config/validation.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<constraint-mapping xmlns="http://symfony.com/schema/dic/constraint-mapping"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/constraint-mapping http://symfony.com/schema/dic/constraint-mapping/constraint-mapping-1.0.xsd">

    <class name="AppBundle\Entity\User">
        <property name="username">
            <constraint name="NotBlank">
                <option name="payload">
                    <value key="severity">error</value>
                </option>
            </constraint>
        </property>
        <property name="password">
            <constraint name="NotBlank">
                <option name="payload">
                    <value key="severity">error</value>
                </option>
            </constraint>
        </property>
        <property name="bankAccountNumber">
            <constraint name="Iban">
                <option name="payload">
                    <value key="severity">warning</value>
                </option>
            </constraint>
        </property>
    </class>
</constraint-mapping>
```

PHP:

```
// src/AppBundle/Entity/User.php
namespace AppBundle\Entity;

use Symfony\Component\Validator\Mapping\ClassMetadata;
use Symfony\Component\Validator\Constraints as Assert;

class User
{
    public static function loadValidatorMetadata(ClassMetadata $metadata)
    {
        $metadata->addPropertyConstraint('username', new Assert\NotBlank(array(
            'payload' => array('severity' => 'error'),
        )));
        $metadata->addPropertyConstraint('password', new Assert\NotBlank(array(
            'payload' => array('severity' => 'error'),
        )));
        $metadata->addPropertyConstraint('bankAccountNumber', new Assert\Iban(array(
            'payload' => array('severity' => 'warning'),
        )));
    }
}
```

## 2. 根据错误级别模板定义用户错误级别

> 2.6 在 Symfony 2.6节中的 **ConstraintViolation** 类中介绍了 **getConstraint()** 方法。

当**用户**输入的对象在检验时失败，这时候就可以通过使用 [getConstraint()](http://api.symfony.com/2.7/Symfony/Component/Validator/ConstraintViolation.html#getConstraint() "getConstraint()") 方法来检索造成失败的限制。每个限制都会把 payload 选项作为一个公开属性进行展示。

```
// a constraint validation failure, instance of
// Symfony\Component\Validator\ConstraintViolation
$constraintViolation = ...;
$constraint = $constraintViolation->getConstraint();
$severity = isset($constraint->payload['severity']) ? $constraint->payload['severity'] : null;
```

例如，您可以利用这个功能把错误级别作为一个 HTML 类的一个附加项添加到表单的错误区域：

```
{%- block form_errors -%}
    {%- if errors|length > 0 -%}
    <ul>
        {%- for error in errors -%}
            {% if error.cause.constraint.payload.severity is defined %}
                {% set severity = error.cause.constraint.payload.severity %}
            {% endif %}
            <li{% if severity is defined %} class="{{ severity }}"{% endif %}>{{ error.message }}</li>
        {%- endfor -%}
    </ul>
    {%- endif -%}
{%- endblock form_errors -%}
```


