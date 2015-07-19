# 如何嵌入集合表单

在这一节中你将学习到如何创建一个嵌入了很多表单集合的表单。这个可能会很有用，举例来说，如果你有一个 **Task** 类并且你想编辑、创建、删除很多与 Task 相关的 **Tag** 对象，就在同一个表单之中。  

>在这一节中，将会假设你使用 Doctrine 作为你的数据库存储。但是如果你没有使用 Doctrine 的话（例如使用的是 Propel 或者只是数据库连接），这都很相似。本指导只有很少一部分真正关注“持续性”。

>如果你正在使用 Doctrine 的话，你需要添加 Doctrine 元数据，包括定义在 Task 的 **tags** 上的**多对多**的映射。  

首先假设每一个 **Task** 属于多重的 **Tag** 对象。我们由建立简单的 **Task** 类开始：  

```
// src/Acme/TaskBundle/Entity/Task.php
namespace Acme\TaskBundle\Entity;

use Doctrine\Common\Collections\ArrayCollection;

class Task
{
    protected $description;

    protected $tags;

    public function __construct()
    {
        $this->tags = new ArrayCollection();
    }

    public function getDescription()
    {
        return $this->description;
    }

    public function setDescription($description)
    {
        $this->description = $description;
    }

    public function getTags()
    {
        return $this->tags;
    }
}
```  

>**ArrayCollection** 是 Doctrine 特有的并且基本上和使用 **array** 一样（但是如果你使用 Doctrine 必须是 **ArrayCollection**）。  

现在，创建一个 **Tag** 类。就像你上面看到的一样，**Task** 可以有很多 **Tag** 对象：  

```
// src/Acme/TaskBundle/Entity/Tag.php
namespace Acme\TaskBundle\Entity;

class Tag
{
    public $name;
}
```

>**名称**属性在这里是公共的所以 **Tag** 对象才可以被用户修正：  

```
// src/Acme/TaskBundle/Form/Type/TagType.php
namespace Acme\TaskBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class TagType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('name');
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => 'Acme\TaskBundle\Entity\Tag',
        ));
    }

    public function getName()
    {
        return 'tag';
    }
}
```

有了这个，你有足够的力量让它自己渲染 tag 表单。但是由于最终目标是允许 **Task** 的 tag 可以在 task 表单中自己修正，为 **Task** 类创建一个表单。  

注意你使用[集合](http://symfony.com/doc/current/reference/forms/types/collection.html)字段类型来嵌入 **TagType** 表单的集合：  

```
// src/Acme/TaskBundle/Form/Type/TaskType.php
namespace Acme\TaskBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class TaskType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('description');

        $builder->add('tags', 'collection', array('type' => new TagType()));
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => 'Acme\TaskBundle\Entity\Task',
        ));
    }

    public function getName()
    {
        return 'task';
    }
}
```

在你的控制器中，你现在将要初始化一个新的 **TaskType** 实例：  

```
// src/Acme/TaskBundle/Controller/TaskController.php
namespace Acme\TaskBundle\Controller;

use Acme\TaskBundle\Entity\Task;
use Acme\TaskBundle\Entity\Tag;
use Acme\TaskBundle\Form\Type\TaskType;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class TaskController extends Controller
{
    public function newAction(Request $request)
    {
        $task = new Task();

        // dummy code - this is here just so that the Task has some tags
        // otherwise, this isn't an interesting example
        $tag1 = new Tag();
        $tag1->name = 'tag1';
        $task->getTags()->add($tag1);
        $tag2 = new Tag();
        $tag2->name = 'tag2';
        $task->getTags()->add($tag2);
        // end dummy code

        $form = $this->createForm(new TaskType(), $task);

        $form->handleRequest($request);

        if ($form->isValid()) {
            // ... maybe do some form processing, like saving the Task and Tag objects
        }

        return $this->render('AcmeTaskBundle:Task:new.html.twig', array(
            'form' => $form->createView(),
        ));
    }
}
```

现在相应的模板也能够渲染 task 的两个**描述**字段也能够渲染已经和 **Task** 有关联的 **TagType** 的任何标签。在上面的控制器中。我们添加了一些虚拟的代码这样你就可以看的清楚了（因为 **Task** 在最初创建时并没有 tag）。  

```Twig
{# src/Acme/TaskBundle/Resources/views/Task/new.html.twig #}

{# ... #}

{{ form_start(form) }}
    {# render the task's only field: description #}
    {{ form_row(form.description) }}

    <h3>Tags</h3>
    <ul class="tags">
        {# iterate over each existing tag and render its only field: name #}
        {% for tag in form.tags %}
            <li>{{ form_row(tag.name) }}</li>
        {% endfor %}
    </ul>
{{ form_end(form) }}

{# ... #}
```

```PHP
<!-- src/Acme/TaskBundle/Resources/views/Task/new.html.php -->

<!-- ... -->

<?php echo $view['form']->start($form) ?>
    <!-- render the task's only field: description -->
    <?php echo $view['form']->row($form['description']) ?>

    <h3>Tags</h3>
    <ul class="tags">
        <?php foreach($form['tags'] as $tag): ?>
            <li><?php echo $view['form']->row($tag['name']) ?></li>
        <?php endforeach ?>
    </ul>
<?php echo $view['form']->end($form) ?>

<!-- ... -->
```

当用户提交表单的时候，**tags** 字段的提交的数据将会用于构造 **Tag** 对象的 **ArrayCollection**，这个将会在之后设置 **Task** 实例的 **tag** 字段。  

**tags** 的集合可以通过 **$task->getTags()** 正常访问并且可以保存到数据库或者在你想用的时候就可以用。  

目前为止，这个都是很好用的，但是这不允许你动态添加新的 tag 或者删除已经存在的 tag。所以，当编辑已经存在的 tag 是非常好用，然而你的用户不能实际添加新的 tag。  

>在这一节，你只嵌入了一个集合，但是不会仅限于此。你也可以随意嵌入嵌入的集合只要你喜欢。但是如果你在你的开发设置中使用 Xdebug 的话，你就会可能收到 **Maximum function nesting level of '100' reached, aborting!** 的错误提示。这是由于 **xdebug.max_nesting_level** 的 PHP 设置，这个值默认是 100。

>这个直接将循环是限制到 100 可能会在模板中渲染表单时不足，如果你一次渲染整个表单（例如 **form_widget(form)**）。为了避免这个你可以将这个直接设置成一个较高的值（或者通过 **php.ini** 文件或者通过 [ini_set](http://php.net/manual/en/function.ini-set.php)，例如在 **app/autoload.php** 中）或者手动使用 **form_row** 渲染每一个表单。  

## 允许有“原型”的“新” Tags ##

允许用户动态添加新的 tag 就意味着你需要使用一些 JavaScript。之前你在你的表单中添加了两个 tag。现在让用户直接在浏览器中添加他们需要的 tag 表单。这将会通过一些 JavaScript 来完成。  

你需要做的第一件事就是让表单集合知道它将会受到不明数量的 tag。目前为止你已经添加了两个并且表单类型希望就是收到两个，否则将会出现错误提示：**This form should not contain extra fields**。为了使这个变得灵活，向你的集合表单中添加 **allow_add** 选项：  

```
// src/Acme/TaskBundle/Form/Type/TaskType.php

// ...
use Symfony\Component\Form\FormBuilderInterface;

public function buildForm(FormBuilderInterface $builder, array $options)
{
    $builder->add('description');

    $builder->add('tags', 'collection', array(
        'type'         => new TagType(),
        'allow_add'    => true,
    ));
}
```

除了高速字段接收任何数量的提交的对象之外，**allow_add** 也为你制造了一个“*原型*”变量。这个“原型”是一个小的“模板”包含了所有的 HTML 能够渲染任意的新的 “tag” 表单。为了渲染它，在你的模板中进行如下改变：  

```Twig
<ul class="tags" data-prototype="{{ form_widget(form.tags.vars.prototype)|e }}">
    ...
</ul>
```  

```PHP
<ul class="tags" data-prototype="<?php
    echo $view->escape($view['form']->row($form['tags']->vars['prototype']))
?>">
    ...
</ul>
```  

>如果你一次渲染你的整个 “tags” 子表单（例如 **form_row(form.tags)**），那么原型将会在外部的 **div** 作为 **data-prototype** 属性可用，和你上面看到的相似。  

>**form.tags.vars.prototype** 是表单元素这个看起来和感觉就是队里的 **form_widget(tag)** 元素在你的 **for** 循环中。这就意味着你可以调用 **form_widget**, **form_row** 或者 **form_label**。你甚至可以选择只渲染它的一个字段（例如**名称**字段）：  

>```
>{{ form_widget(form.tags.vars.prototype.name)|e }}
>```

在渲染页，结果将会像下面这样：  

```
<ul class="tags" data-prototype="&lt;div&gt;&lt;label class=&quot; required&quot;&gt;__name__&lt;/label&gt;&lt;div id=&quot;task_tags___name__&quot;&gt;&lt;div&gt;&lt;label for=&quot;task_tags___name___name&quot; class=&quot; required&quot;&gt;Name&lt;/label&gt;&lt;input type=&quot;text&quot; id=&quot;task_tags___name___name&quot; name=&quot;task[tags][__name__][name]&quot; required=&quot;required&quot; maxlength=&quot;255&quot; /&gt;&lt;/div&gt;&lt;/div&gt;&lt;/div&gt;">
```  

这一节的目标就是使用 JavaScript 读取这个属性并且动态添加新的 tag 表单当用户点击“添加一个 tag”链接的时候。为了使事情变得简单，这个例子使用了 jQuery 并且假设在你的包的某个位置包括它。  

在你的包的某个位置添加一个**脚本**标签这样你就可以开始写一些 JavaScript 了。  

首先，在“tags”列表的底部通过 JavaScript 添加一个链接。然后，将“单击”事件捆绑在链接上这样你就可以添加一个新的 tag 表单（**addTagForm** 将会在接下来展示）：  

```
var $collectionHolder;

// setup an "add a tag" link
var $addTagLink = $('<a href="#" class="add_tag_link">Add a tag</a>');
var $newLinkLi = $('<li></li>').append($addTagLink);

jQuery(document).ready(function() {
    // Get the ul that holds the collection of tags
    $collectionHolder = $('ul.tags');

    // add the "add a tag" anchor and li to the tags ul
    $collectionHolder.append($newLinkLi);

    // count the current form inputs we have (e.g. 2), use that as the new
    // index when inserting a new item (e.g. 2)
    $collectionHolder.data('index', $collectionHolder.find(':input').length);

    $addTagLink.on('click', function(e) {
        // prevent the link from creating a "#" on the URL
        e.preventDefault();

        // add a new tag form (see next code block)
        addTagForm($collectionHolder, $newLinkLi);
    });
});
```

**addTagForm** 功能的工作就是当链接被点击的时候使用 **data-prototype** 属性动态添加一个新的表单。**data-prototype** HTML 包含了名为 **task[tags][__name__][name]** 的 tag **文本**输入元素和 **task_tags___name___name** 的 id。**__name__ ** 是一个小的“占位符”，这个你可以使用一个独特的，增量的数字代替（例如 **task[tags][3][name]**）。  

使这些起作用的实际的代码可能很不同，但是下面有一个例子：  

```
function addTagForm($collectionHolder, $newLinkLi) {
    // Get the data-prototype explained earlier
    var prototype = $collectionHolder.data('prototype');

    // get the new index
    var index = $collectionHolder.data('index');

    // Replace '__name__' in the prototype's HTML to
    // instead be a number based on how many items we have
    var newForm = prototype.replace(/__name__/g, index);

    // increase the index with one for the next item
    $collectionHolder.data('index', index + 1);

    // Display the form in the page in an li, before the "Add a tag" link li
    var $newFormLi = $('<li></li>').append(newForm);
    $newLinkLi.before($newFormLi);
}
```  

>将你的 JavaScript 文件分开到几个真正的 JavaScript 文件中比在这里用 HTML 写要好。  

现在，每当用户点击**添加 tag** 的链接时，一个新的子表单都会出现在页面中。当表单被提交的时候，任何新的 tag 表单都会被转换成新的 **Tag** 对象并且添加到 **Task** 对象的 **tags** 属性中。  

>你可以在 [JSFiddle](http://jsfiddle.net/847Kf/4/) 中找到实例。  

为了使得处理这些新的 tag 更容易，为 **Task** 类中的 tags 添加一个“添加”和一个“移除”方法：  

```
// src/Acme/TaskBundle/Entity/Task.php
namespace Acme\TaskBundle\Entity;

// ...
class Task
{
    // ...

    public function addTag(Tag $tag)
    {
        $this->tags->add($tag);
    }

    public function removeTag(Tag $tag)
    {
        // ...
    }
}
```

接下来，添加一个 **by_reference** 选项到 **tags** 字段并且将其设置为 **false**：  

```
// src/Acme/TaskBundle/Form/Type/TaskType.php

// ...
public function buildForm(FormBuilderInterface $builder, array $options)
{
    // ...

    $builder->add('tags', 'collection', array(
        // ...
        'by_reference' => false,
    ));
}
```

由于这两个更改，当表单被提交时，每一个新的 **Tag** 都是通过调用 **addTag** 方法添加到 **Task** 类的。在做这个改变之前，他们是通过调用 **$task->getTags()->add($tag)** 方法由表单内部添加的。这样也很好，但是使用“adder”方法使得处理这些新的 **Tag** 对象更容易了（尤其是如果你是用的是你接下来将会学习的 Doctrine！）。  

>你已经建立了 **addTag** 和 **removeTag** **两个**方法，否则表单将会继续使用 **setTag** 即使 **by_reference** 是 **false**。你将会在本文的后面详细学习 **removeTag** 方法。  

>**Doctrine**:**串联关系并且保留“颠倒”的一边**

>为了使用 Doctrine 来保存新的 tags，你需要考虑多一点事情。首先除非迭代绑定所有的新的 **Tag** 对象并且在每一个上调用 **$em->persist($tag)**，你将会从 Doctrine 收到一个错误提示：  

>*一个新的实例已经通过关系发现了* **Acme\TaskBundle\Entity\Task#tags** *那是未配置的叠加持续实例操作*...  

>为了解决这个问题，你可能需要自动选择“叠加”持续的操作从 **Task** 对象到任何相关的 tags。为了完成这个，需要向你的**多对多**元数据中添加 **cascade** 选项：  

>```Annotations
>// src/Acme/TaskBundle/Entity/Task.php
// ...
/**
 * @ORM\ManyToMany(targetEntity="Tag", cascade={"persist"})
 */
protected $tags;
>```

>```YAML
># src/Acme/TaskBundle/Resources/config/doctrine/Task.orm.yml
Acme\TaskBundle\Entity\Task:
    type: entity
    # ...
    oneToMany:
        tags:
            targetEntity: Tag
            cascade:      [persist]
>```

>```XML
><!-- src/Acme/TaskBundle/Resources/config/doctrine/Task.orm.xml -->
<doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
                    http://doctrine-project.org/schemas/orm/doctrine-mapping.xsd">
    <entity name="Acme\TaskBundle\Entity\Task">
        <!-- ... -->
        <one-to-many field="tags" target-entity="Tag">
            <cascade>
                <cascade-persist />
            </cascade>
        </one-to-many>
    </entity>
</doctrine-mapping>
>```

>第二个潜在的问题就是处理 Doctrine 的[所有方与反向方](http://docs.doctrine-project.org/en/latest/reference/unitofwork-associations.html)。在本例子中，如果“所有”方的关系是 “Task”，那么持续性就会很好的工作由于 tags 已经正确添加到 Task。然而，如果所有方在 “Tag” 上，那么你就需要多做一点工作来确保关系的正确方得到修正。  

>这个窍门就是为了确保单一的 “Task” 设置在每一个 “Tag” 上。一个简单的方法是添加一些额外的逻辑到 **addTag()**，这个被表单的类型调用由于 **by_reference** 设置成了 **false**：  

>```
>// src/Acme/TaskBundle/Entity/Task.php
// ...
public function addTag(Tag $tag)
{
    $tag->addTask($this);
    $this->tags->add($tag);
}
>```

>在 **Tag** 中，只需要确定你有 **addTask** 方法：  

>```
>// src/Acme/TaskBundle/Entity/Tag.php
// ...
public function addTask(Task $task)
{
    if (!$this->tasks->contains($task)) {
        $this->tasks->add($task);
    }
}
>```  

>如果你拥有一对多的关系，那么工作区就很相似，除非你能仅仅从 **addTag** 中调用 **setTask**。  

## 允许 Tags 被移除 ##

接下来的一步就是允许删除集合中的特定条目。这个方法和允许 Tags 被添加差不多。  

从在表单类型中添加 **allow_delete** 选项开始：  

```
// src/Acme/TaskBundle/Form/Type/TaskType.php
// ...
public function buildForm(FormBuilderInterface $builder, array $options)
{
    // ...
    $builder->add('tags', 'collection', array(
        // ...
        'allow_delete' => true,
    ));
}
```  

现在，你需要在 **Task** 的 **removeTag** 方法中加入一些代码：  

```
// src/Acme/TaskBundle/Entity/Task.php

// ...
class Task
{
    // ...

    public function removeTag(Tag $tag)
    {
        $this->tags->removeElement($tag);
    }
}
```

### 模板修正 ###

**allow_delete** 选项有一个后果：如果一个结合的条目没有在提交时发送，相关的数据就会从服务器的集合中移除。因此解决办法就是从表单组件中移除 DOM。  

首先，给每一个 tag 表单添加“删除这个 tag” 的链接：  

```
jQuery(document).ready(function() {
    // Get the ul that holds the collection of tags
    $collectionHolder = $('ul.tags');

    // add a delete link to all of the existing tag form li elements
    $collectionHolder.find('li').each(function() {
        addTagFormDeleteLink($(this));
    });

    // ... the rest of the block from above
});

function addTagForm() {
    // ...

    // add a delete link to the new form
    addTagFormDeleteLink($newFormLi);
}
```

**addTagFormDeleteLink** 功能将会如下所示：  

```
function addTagFormDeleteLink($tagFormLi) {
    var $removeFormA = $('<a href="#">delete this tag</a>');
    $tagFormLi.append($removeFormA);

    $removeFormA.on('click', function(e) {
        // prevent the link from creating a "#" on the URL
        e.preventDefault();

        // remove the li for the tag form
        $tagFormLi.remove();
    });
}
```

当 tag 表单从 DOM 移除并且提交，移除的 **Tag** 对象将不会包含在传递到 **setTags** 的集合中。基于你的持续层，这可能或者可能不足以实际移除被移除的 **Tag** 和 **Task** 对象间的关系。  

>**Doctrine：保证数据库的持续性**

>当这样移除对象之后，你可能需要多做一些工作来保证 **Task** 以及被移除的 **Tag** 之间的关系被适当移除。  

>在 Doctrine 之中，你有两种关系：所有方以及反方。正常情况下你将会有多对多的关系并且删除的 tags 将会消失并且一直正确（添加新的 tags 也会有效果）。  

>但是如果你有一对多的关系或者在 Task 实体上有多对多的  **mappedBy**关系（意味着 Task  是“反”向的），你就需要移除 tags 来保持正确的一致性。  

>在这种情况下，你可以通过修正控制器来移除已经移除的 tag 的关系。这个假设你有一些 **editAction**  这个处理你的 Task 的“更新”：  

>```
>// src/Acme/TaskBundle/Controller/TaskController.php
use Doctrine\Common\Collections\ArrayCollection;
// ...
public function editAction($id, Request $request)
{
    $em = $this->getDoctrine()->getManager();
    $task = $em->getRepository('AcmeTaskBundle:Task')->find($id);
    if (!$task) {
        throw $this->createNotFoundException('No task found for id '.$id);
    }
    $originalTags = new ArrayCollection();
    // Create an ArrayCollection of the current Tag objects in the database
    foreach ($task->getTags() as $tag) {
        $originalTags->add($tag);
    }
    $editForm = $this->createForm(new TaskType(), $task);
    $editForm->handleRequest($request);
    if ($editForm->isValid()) {
        // remove the relationship between the tag and the Task
        foreach ($originalTags as $tag) {
            if (false === $task->getTags()->contains($tag)) {
                // remove the Task from the Tag
                $tag->getTasks()->removeElement($task);
                // if it was a many-to-one relationship, remove the relationship like this
                // $tag->setTask(null);
                $em->persist($tag);
                // if you wanted to delete the Tag entirely, you can also do that
                // $em->remove($tag);
            }
        }
        $em->persist($task);
        $em->flush();
        // redirect back to some edit page
        return $this->redirectToRoute('task_edit', array('id' => $id));
    }
    // render some form template
}
>```

>正如你所见，正确地添加或者移除元素是很微妙的。除非你有多对多的关系在那里 Task 在 “拥有”方，你将需要做额外的工作来确保每一个 Tag 对象自己的关系正确地更新（不论你是添加还是删除已经存在的 tags）。  


