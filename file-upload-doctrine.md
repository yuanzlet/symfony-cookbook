# 如何用 Doctrine 上传文件

> 除了您自己上传文件，您或许考虑使用 [VichUploaderBundle](https://github.com/dustin10/VichUploaderBundle) 社区 bundle。这个 bundle 提供了所有常见的操作（例如文件重命名、保存和删除），并且它紧密地与 Doctrine ORM、MongoDB ODM、PHPCR ODM 和 Propel 组成为一个整体。

用 Doctrine 实体上传文件与上传任何其他文件无区别。换句话说，您可以在提交表单之后自由移动您控件中的文件。为了举例如何做这个，参见[文件类型引用](http://symfony.com/doc/current/reference/forms/types/file.html)页面。

如果您选择的话，您也可以整合上传文件到您的实体生命周期（例如，创建、更新和移除）。这种情况下，当您的实体被创建，更新或者是从 Doctrine 移除，上传文件和移除进程将会自动发生（不需要在您的控件中做任何事）。

要使这个奏效，您需要注意大量的细节，将会在这本教程条目中讲到。

## 基本设置

首先，创建一个简单的 Doctrine 实体类来使用：

```
// src/AppBundle/Entity/Document.php
namespace AppBundle\Entity;

use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Validator\Constraints as Assert;

/**
 * @ORM\Entity
 */
class Document
{
    /**
     * @ORM\Id
     * @ORM\Column(type="integer")
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    public $id;

    /**
     * @ORM\Column(type="string", length=255)
     * @Assert\NotBlank
     */
    public $name;

    /**
     * @ORM\Column(type="string", length=255, nullable=true)
     */
    public $path;

    public function getAbsolutePath()
    {
        return null === $this->path
            ? null
            : $this->getUploadRootDir().'/'.$this->path;
    }

    public function getWebPath()
    {
        return null === $this->path
            ? null
            : $this->getUploadDir().'/'.$this->path;
    }

    protected function getUploadRootDir()
    {
        // the absolute directory path where uploaded
        // documents should be saved
        return __DIR__.'/../../../../web/'.$this->getUploadDir();
    }

    protected function getUploadDir()
    {
        // get rid of the __DIR__ so it doesn't screw up
        // when displaying uploaded doc/image in the view.
        return 'uploads/documents';
    }
}
```

**Document** 实体有一个名称并且与一个文件相关联。**path** 属性储存相关的路径到文件，并且保存到数据库中。

**getAbsolutePath()** 是一个可以将绝对路径返回到文件的便捷方法，而 **getWebPath()** 是一个可以将网页路径返回，可用于模板链接上传文件的便捷方法。

> 如果您还未做完，您应该首先阅读[文件](http://symfony.com/doc/current/reference/forms/types/file.html)类型文档来了解基本的上传进程是如何运行的。  

> 如果您正在使用标注来指定您的验证规则（正如例子所示），确保您已经用标注启动了验证（参见[验证配置](http://symfony.com/doc/current/book/validation.html#book-validation-configuration)）。

> 如果您使用方法 **getUploadRootDir()**，注意这会保存根文件的内部文件，可以被所有人读取。要考虑把它放在根文件之外，并当您需要保护这些文件的时候添加自定义查看逻辑。

要上传表单中的实际文件，使用一个“虚拟” **file** 域。例如，如果您正在一个控件里直接构建您的表单，它看起来会像这样：

```
public function uploadAction()
{
    // ...

    $form = $this->createFormBuilder($document)
        ->add('name')
        ->add('file')
        ->getForm();

    // ...
}
```

接下来，在您的 **Document** 类里创建这个属性，并添加一些验证规则：

```
use Symfony\Component\HttpFoundation\File\UploadedFile;

// ...
class Document
{
    /**
     * @Assert\File(maxSize="6000000")
     */
    private $file;

    /**
     * Sets file.
     *
     * @param UploadedFile $file
     */
    public function setFile(UploadedFile $file = null)
    {
        $this->file = $file;
    }

    /**
     * Get file.
     *
     * @return UploadedFile
     */
    public function getFile()
    {
        return $this->file;
    }
}
```

Annotations

```
// src/AppBundle/Entity/Document.php
namespace AppBundle\Entity;

// ...
use Symfony\Component\Validator\Constraints as Assert;

class Document
{
    /**
     * @Assert\File(maxSize="6000000")
     */
    private $file;

    // ...
}
```

YAML:

```
# src/AppBundle/Resources/config/validation.yml
AppBundle\Entity\Document:
    properties:
        file:
            - File:
                maxSize: 6000000
```

XML:

```
<!-- src/AppBundle/Resources/config/validation.xml -->
<class name="AppBundle\Entity\Document">
    <property name="file">
        <constraint name="File">
            <option name="maxSize">6000000</option>
        </constraint>
    </property>
</class>
```

PHP:

```
// src/AppBundle/Entity/Document.php
namespace Acme\DemoBundle\Entity;

// ...
use Symfony\Component\Validator\Mapping\ClassMetadata;
use Symfony\Component\Validator\Constraints as Assert;

class Document
{
    // ...

    public static function loadValidatorMetadata(ClassMetadata $metadata)
    {
        $metadata->addPropertyConstraint('file', new Assert\File(array(
            'maxSize' => 6000000,
        )));
    }
}
```

> 当您正在使用 **File** 约束，Symfony 会自动猜测表单域是文件上传输入。这就是您为什么在创建上面的表单时（**->add('file')**）不需要做显示设置的原因。

以下控件展示了如何处理整个进程：

```
// ...
use AppBundle\Entity\Document;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;
use Symfony\Component\HttpFoundation\Request;
// ...

/**
 * @Template()
 */
public function uploadAction(Request $request)
{
    $document = new Document();
    $form = $this->createFormBuilder($document)
        ->add('name')
        ->add('file')
        ->getForm();

    $form->handleRequest($request);

    if ($form->isValid()) {
        $em = $this->getDoctrine()->getManager();

        $em->persist($document);
        $em->flush();

        return $this->redirectToRoute(...);
    }

    return array('form' => $form->createView());
}
```

之前的控件会用提交的名字自动保存 **Document** 实体，但是不会对文件做任何事情并且 **path** 属性为空白。

上传文件的一个简单的方法是在实体保存之前移动文件，然后相应地设置 **path** 属性。首先在 **Document** 类调用一个新的 **upload()** 方法，您就能立刻上传文件：

```
if ($form->isValid()) {
    $em = $this->getDoctrine()->getManager();

    $document->upload();

    $em->persist($document);
    $em->flush();

    return $this->redirectToRoute(...);
}
```

**upload()** 方法会利用 [UploadedFile](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/File/UploadedFile.html) 对象，在一个 **file** 域提交后会返回：

```
public function upload()
{
    // the file property can be empty if the field is not required
    if (null === $this->getFile()) {
        return;
    }

    // use the original file name here but you should
    // sanitize it at least to avoid any security issues

    // move takes the target directory and then the
    // target filename to move to
    $this->getFile()->move(
        $this->getUploadRootDir(),
        $this->getFile()->getClientOriginalName()
    );

    // set the path property to the filename where you've saved the file
    $this->path = $this->getFile()->getClientOriginalName();

    // clean up the file property as you won't need it anymore
    $this->file = null;
}
```

## 使用生命周期回呼

> 使用生命周期回呼是一个限制的技术，有一些缺陷。如果您想移除在 **Document::getUploadRootDir()** 方法内部的硬编码的 **__DIR__** 引用，最好的方法就是开始使用明确的 [doctrine 监听器](http://symfony.com/doc/current/cookbook/doctrine/event_listeners_subscribers.html)注入内核参数，比如 **kernel.root_dir** 来构建绝对路径。

尽管这个实现奏效，但是它有一个主要缺陷：如果实体保存的时候有问题怎么办？文件已经移动到了它的最终位置尽管实体的 **path** 属性未被正确保存。

为了避免这类问题，您应该改变实施从而使数据库操作和文件的移动具有原子性：如果在保存实体时有问题或者文件不能被移动，那么*没有事情*会发生。

要做到这一点，您需要正确移动文件因为 Doctrine 保存实体到数据库。这个可以通过挂钩一个实体生命周期回呼完成。

```
/**
 * @ORM\Entity
 * @ORM\HasLifecycleCallbacks
 */
class Document
{
}
```

接下来，重构 **Document** 类来利用这些回呼：

```
use Symfony\Component\HttpFoundation\File\UploadedFile;

/**
 * @ORM\Entity
 * @ORM\HasLifecycleCallbacks
 */
class Document
{
    private $temp;

    /**
     * Sets file.
     *
     * @param UploadedFile $file
     */
    public function setFile(UploadedFile $file = null)
    {
        $this->file = $file;
        // check if we have an old image path
        if (isset($this->path)) {
            // store the old name to delete after the update
            $this->temp = $this->path;
            $this->path = null;
        } else {
            $this->path = 'initial';
        }
    }

    /**
     * @ORM\PrePersist()
     * @ORM\PreUpdate()
     */
    public function preUpload()
    {
        if (null !== $this->getFile()) {
            // do whatever you want to generate a unique name
            $filename = sha1(uniqid(mt_rand(), true));
            $this->path = $filename.'.'.$this->getFile()->guessExtension();
        }
    }

    /**
     * @ORM\PostPersist()
     * @ORM\PostUpdate()
     */
    public function upload()
    {
        if (null === $this->getFile()) {
            return;
        }

        // if there is an error when moving the file, an exception will
        // be automatically thrown by move(). This will properly prevent
        // the entity from being persisted to the database on error
        $this->getFile()->move($this->getUploadRootDir(), $this->path);

        // check if we have an old image
        if (isset($this->temp)) {
            // delete the old image
            unlink($this->getUploadRootDir().'/'.$this->temp);
            // clear the temp image path
            $this->temp = null;
        }
        $this->file = null;
    }

    /**
     * @ORM\PostRemove()
     */
    public function removeUpload()
    {
        $file = $this->getAbsolutePath();
        if ($file) {
            unlink($file);
        }
    }
}
```

> 如果对你实体的改变被一个 Doctrine 事件监听器或者事件订阅者所处理，**preUpdate()** 回呼必须通知 Doctrine 所完成的变化。关于 preUpadate 事件限制的所有引用，在 Doctrine 事件文档中参见 [preUpdate](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/events.html#preupdate)。

类现在做一切您需要的事情：它会在保存之前产生一个独特的文件名，在保存之后移动文件，并且如果实体被删除的话就移除文件。

现在文件的移动是由实体自动处理的，**$document->upload()** 的调用应从控件中移除：

```
if ($form->isValid()) {
    $em = $this->getDoctrine()->getManager();

    $em->persist($document);
    $em->flush();

    return $this->redirectToRoute(...);
}
```

> **@ORM\PrePersist()** 和 **@ORM\PostPersist()** 事件回呼在实体保存到数据库前后被触发。在另一方面，当实体更新后，**@ORM\PreUpdate()** 和 **@ORM\PostUpdate()** 事件回呼被调用。
 
> 如果被保存的实体的字段其中之一有变化，**PreUpdate** 和 **PostUpdate** 回呼才会被激发。这意味着，默认情况下，如果您只调整 **$file** 属性，这些事件将不再被激发，因为属性本身不是直接通过 Doctrine 保存的。一个解决方案就是使用一个保存在 Doctrine 中的 **updated** 字段，然后当改变文件的时候手动调整。
 
 ## 使用 id 作为文件名称
 
 如果您想使用 **id** 作为文件的名称，操作和您需要在 **path** 属性下保存的扩展有轻微的不同，并不是实际的文件名称：
 
 ```
 use Symfony\Component\HttpFoundation\File\UploadedFile;

/**
 * @ORM\Entity
 * @ORM\HasLifecycleCallbacks
 */
class Document
{
    private $temp;

    /**
     * Sets file.
     *
     * @param UploadedFile $file
     */
    public function setFile(UploadedFile $file = null)
    {
        $this->file = $file;
        // check if we have an old image path
        if (is_file($this->getAbsolutePath())) {
            // store the old name to delete after the update
            $this->temp = $this->getAbsolutePath();
        } else {
            $this->path = 'initial';
        }
    }

    /**
     * @ORM\PrePersist()
     * @ORM\PreUpdate()
     */
    public function preUpload()
    {
        if (null !== $this->getFile()) {
            $this->path = $this->getFile()->guessExtension();
        }
    }

    /**
     * @ORM\PostPersist()
     * @ORM\PostUpdate()
     */
    public function upload()
    {
        if (null === $this->getFile()) {
            return;
        }

        // check if we have an old image
        if (isset($this->temp)) {
            // delete the old image
            unlink($this->temp);
            // clear the temp image path
            $this->temp = null;
        }

        // you must throw an exception here if the file cannot be moved
        // so that the entity is not persisted to the database
        // which the UploadedFile move() method does
        $this->getFile()->move(
            $this->getUploadRootDir(),
            $this->id.'.'.$this->getFile()->guessExtension()
        );

        $this->setFile(null);
    }

    /**
     * @ORM\PreRemove()
     */
    public function storeFilenameForRemove()
    {
        $this->temp = $this->getAbsolutePath();
    }

    /**
     * @ORM\PostRemove()
     */
    public function removeUpload()
    {
        if (isset($this->temp)) {
            unlink($this->temp);
        }
    }

    public function getAbsolutePath()
    {
        return null === $this->path
            ? null
            : $this->getUploadRootDir().'/'.$this->id.'.'.$this->path;
    }
}
```

您将会注意到在这种情况下，您需要再做一些工作来移除文件。在移除之前，您必须存储文件路径（因为它取决于 id）。然后，一旦对象已被完全从数据库移除，您可以安全地删除文件（在 **PostRemove** 中）。
