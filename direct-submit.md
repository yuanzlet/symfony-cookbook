# 如何使用 submit() 函数处理表单提交

> [handleRequest()](http://api.symfony.com/2.7/Symfony/Component/Form/FormInterface.html#handleRequest()) 方法是在 Symfony 2.3 中引进的。  

有了 **handleRequest()** 方法，处理表单提交就简单多了：  

```
use Symfony\Component\HttpFoundation\Request;
// ...

public function newAction(Request $request)
{
    $form = $this->createFormBuilder()
        // ...
        ->getForm();

    $form->handleRequest($request);

    if ($form->isValid()) {
        // perform some action...

        return $this->redirectToRoute('task_success');
    }

    return $this->render('AcmeTaskBundle:Default:new.html.twig', array(
        'form' => $form->createView(),
    ));
}
```

> 有关这个方法的更多细节详见[处理表单提交](http://symfony.com/doc/current/book/forms.html#book-form-handling-form-submissions)。  

## 手动调用 Form::submit()

> 在 Symfony 2.3 之前，**submit()** 方法叫做 **bind()**。  

在某些情况下，你可能想要对何时你的表单被提交以及什么数据传递到它有更好的控制。代替使用 [handleRequest()](http://api.symfony.com/2.7/Symfony/Component/Form/FormInterface.html#handleRequest()) 方法，直接将提交的数据传递到 [submit()](http://api.symfony.com/2.7/Symfony/Component/Form/FormInterface.html#submit())：  

```
use Symfony\Component\HttpFoundation\Request;
// ...

public function newAction(Request $request)
{
    $form = $this->createFormBuilder()
        // ...
        ->getForm();

    if ($request->isMethod('POST')) {
        $form->submit($request->request->get($form->getName()));

        if ($form->isValid()) {
            // perform some action...

            return $this->redirectToRoute('task_success');
        }
    }

    return $this->render('AcmeTaskBundle:Default:new.html.twig', array(
        'form' => $form->createView(),
    ));
}
```

> 包含嵌套字段的表单期望在 [submit()](http://api.symfony.com/2.7/Symfony/Component/Form/FormInterface.html#submit()) 中的一个数组。你也可以同直接在字段上调用 [submit()](http://api.symfony.com/2.7/Symfony/Component/Form/FormInterface.html#submit()) 提交独立的字段：  

>```
>$form->get('firstName')->submit('Fabien');
>```

## 向 Form::submit() 传递一个请求（不赞成）

> 在 Symfony 2.3 之前，**submit** 方法叫做 **bind**。  

在 Symfony 2.3 之前，[submit()](http://api.symfony.com/2.7/Symfony/Component/Form/FormInterface.html#submit()) 方法接受[请求](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/Request.html)对象作为方便的捷径到前一个例子：  

```
use Symfony\Component\HttpFoundation\Request;
// ...

public function newAction(Request $request)
{
    $form = $this->createFormBuilder()
        // ...
        ->getForm();

    if ($request->isMethod('POST')) {
        $form->submit($request);

        if ($form->isValid()) {
            // perform some action...

            return $this->redirectToRoute('task_success');
        }
    }

    return $this->render('AcmeTaskBundle:Default:new.html.twig', array(
        'form' => $form->createView(),
    ));
}
```

将[请求](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/Request.html)直接传递到 [submit()](http://api.symfony.com/2.7/Symfony/Component/Form/FormInterface.html#submit()) 依然有效，但是我们不推荐并且这将会在 Symfony 3.0 中移除。作为替代你应当看看 [handleRequest()](http://api.symfony.com/2.7/Symfony/Component/Form/FormInterface.html#handleRequest()) 方法。