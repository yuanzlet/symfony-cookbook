# 如何为表单类配置空数据

**empty_data** 允许你为你的表单类指定一个空的数据集合。如果你提交表单就会用到空的数据集合，但是没有在你的表单中调用 **setData()** 或者当你建立表单的时候传入数据。举例来说：  

```
public function indexAction()
{
    $blog = ...;

    // $blog is passed in as the data, so the empty_data
    // option is not needed
    $form = $this->createForm(new BlogType(), $blog);

    // no data is passed in, so empty_data is
    // used to get the "starting data"
    $form = $this->createForm(new BlogType());
}
```

默认情况下，**empty_data** 设置为 **null**。或者，如果你为你的表单类指定了 **data_class** 选项，它将会默认一个新的实例的类。这个实例将会通过调用构造函数创建并且没有参数。  

如果你想要重写这个默认的行为，有两种方法可以用。  

## 选择 1：实例化一个新的类

你可能使用这个方法的原因就是如果你想要使用具有参数的构造函数的话。记住，默认的 **data_class** 选项调用构造函数没有参数：  

```
// src/AppBundle/Form/Type/BlogType.php

// ...
use Symfony\Component\Form\AbstractType;
use AppBundle\Entity\Blog;
use Symfony\Component\OptionsResolver\OptionsResolver;

class BlogType extends AbstractType
{
    private $someDependency;

    public function __construct($someDependency)
    {
        $this->someDependency = $someDependency;
    }
    // ...

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'empty_data' => new Blog($this->someDependency),
        ));
    }
}
```

你可以使用你想使用的任何方法来实例化你的类。在这个例子中，当我们实例化 **BlogType** 的时候，我们向它传递一些依赖性，然后使用它将 **Blog** 类实例化。要点就是，你能够将 **empty_data** 设置到你想使用的全“新的”对象中。  

## 选择 2：提供一个闭包

使用闭包是一个更好的选择，因为它只有在对象需要的时候才会被创建。  

闭包必须接受 **FormInterface** 实例为第一变元：  

```
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Form\FormInterface;
// ...

public function configureOptions(OptionsResolver $resolver)
{
    $resolver->setDefaults(array(
        'empty_data' => function (FormInterface $form) {
            return new Blog($form->get('title')->getData());
        },
    ));
}
```