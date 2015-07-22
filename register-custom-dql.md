# 如何注册自定义 DQL 函数

Doctrine 允许您指定自定义 DQL 函数。要想知道关于这个话题的更多信息，阅读 Doctrine 的教程文章“[DQL 用户定义函数](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/cookbook/dql-user-defined-functions.html)”。

在 Symfony 中，您可以按照如下注册自定义 DQL 函数：

YAML:

```
# app/config/config.yml
doctrine:
    orm:
        # ...
        dql:
            string_functions:
                test_string: AppBundle\DQL\StringFunction
                second_string: AppBundle\DQL\SecondStringFunction
            numeric_functions:
                test_numeric: AppBundle\DQL\NumericFunction
            datetime_functions:
                test_datetime: AppBundle\DQL\DatetimeFunction
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
        <doctrine:orm>
            <!-- ... -->
            <doctrine:dql>
                <doctrine:string-function name="test_string">AppBundle\DQL\StringFunction</doctrine:string-function>
                <doctrine:string-function name="second_string">AppBundle\DQL\SecondStringFunction</doctrine:string-function>
                <doctrine:numeric-function name="test_numeric">AppBundle\DQL\NumericFunction</doctrine:numeric-function>
                <doctrine:datetime-function name="test_datetime">AppBundle\DQL\DatetimeFunction</doctrine:datetime-function>
            </doctrine:dql>
        </doctrine:orm>
    </doctrine:config>
</container>
```

PHP:

```
// app/config/config.php
$container->loadFromExtension('doctrine', array(
    'orm' => array(
        // ...
        'dql' => array(
            'string_functions' => array(
                'test_string'   => 'AppBundle\DQL\StringFunction',
                'second_string' => 'AppBundle\DQL\SecondStringFunction',
            ),
            'numeric_functions' => array(
                'test_numeric' => 'AppBundle\DQL\NumericFunction',
            ),
            'datetime_functions' => array(
                'test_datetime' => 'AppBundle\DQL\DatetimeFunction',
            ),
        ),
    ),
));
```
