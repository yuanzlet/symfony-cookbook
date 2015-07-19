# 如何上传文件

> 代替你自己处理文件上传，你可以考虑使用 [VichUploaderBundle](https://github.com/dustin10/VichUploaderBundle) bundle。这个 bundle 提供所有的一般选项（例如文件重命名，保存和删除）并且整合了 Doctrine ORM, MongoDB ODM, PHPCR ODM 和 Propel。  

试想在你的应用程序中有一个 **Product** 实体并且你想为你的每一个产品添加一个 PDF 格式的小册子。为了完成这个，在你的 **Product** 实体中添加一个名为 **brochure** （小册子）的属性：  

```
// src/AppBundle/Entity/Product.php
namespace AppBundle\Entity;

use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Validator\Constraints as Assert;

class Product
{
    // ...

    /**
     * @ORM\Column(type="string")
     *
     * @Assert\NotBlank(message="Please, upload the product brochure as a PDF file.")
     * @Assert\File(mimeTypes={ "application/pdf" })
     */
    private $brochure;

    public function getBrochure()
    {
        return $this->brochure;
    }

    public function setBrochure($brochure)
    {
        $this->brochure = $brochure;

        return $this;
    }
}
```  

记住 **brochure** 列的类型是**字符串**而不是**二进制**或者**二进制大对象**因为这个只是储存 PDF 的文件名并不是储存文件内容。  

然后向管理 **Product** 实体的表中添加一个新的 **brochure** 字段：  

```
// src/AppBundle/Form/ProductType.php
namespace AppBundle\Form;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class ProductType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            // ...
            ->add('brochure', 'file', array('label' => 'Brochure (PDF file)'))
            // ...
        ;
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => 'AppBundle\Entity\Product',
        ));
    }

    public function getName()
    {
        return 'product';
    }
}
```  

现在，更新你的模板从而渲染表格来显示新的 **brochure** 字段（添加确切的模板代码依赖于你的应用程序以个性化[表格渲染](http://symfony.com/doc/current/cookbook/form/form_customization.html)所用的方法）：  

```
{# app/Resources/views/product/new.html.twig #}
<h1>Adding a new product</h1>

{{ form_start() }}
    {# ... #}

    {{ form_row(form.brochure) }}
{{ form_end() }}
```  

最后，你需要更新处理表格的 controller 的代码：  

```
// src/AppBundle/Controller/ProductController.php
namespace AppBundle\ProductController;

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;
use AppBundle\Entity\Product;
use AppBundle\Form\ProductType;

class ProductController extends Controller
{
    /**
     * @Route("/product/new", name="app_product_new")
     */
    public function newAction(Request $request)
    {
        $product = new Product();
        $form = $this->createForm(new ProductType(), $product);
        $form->handleRequest($request);

        if ($form->isValid()) {
            // $file stores the uploaded PDF file
            /** @var Symfony\Component\HttpFoundation\File\UploadedFile $file */
            $file = $product->getBrochure()

            // Generate a unique name for the file before saving it
            $fileName = md5(uniqid()).'.'.$file->guessExtension();

            // Move the file to the directory where brochures are stored
            $brochuresDir = $this->container->getParameter('kernel.root_dir').'/../web/uploads/brochures';
            $file->move($brochuresDir, $fileName);

            // Update the 'brochure' property to store the PDF file name
            // instead of its contents
            $product->setBrochure($filename);

            // persist the $product variable or any other work...

            return $this->redirect($this->generateUrl('app_product_list'));
        }

        return $this->render('product/new.html.twig', array(
            'form' => $form->createView()
        ));
    }
}
```  

下面是一些有关于上述代码的注意事项：

1. 当表格上传时，**brochure** 属性包含了整个 PDF 文件的内容。因为这个属性仅仅储存了文件名，所以在保存实体更改前你必须设置新的值。    

2. 在 Symfony 应用程序中，上传文件是 [UploadedFile](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/File/UploadedFile.html) 对象的类，这个对象提供了处理上传文件时大多数操作的方法。  

3. 一个好的安全实践就是不要信任用户输入的东西。这也适用于访问者上传的文件。**Uploaded** 类提供了获取原始文件扩展名（[getExtension()()](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/File/UploadedFile.html#getExtension()())），原始文件大小（[getSize()()](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/File/UploadedFile.html#getSize()())）以及原始文件名（[getClientOriginalName()()](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/File/UploadedFile.html#getClientOriginalName()())）的方法。然而，这些方法并*不是*安全的，因为恶意使用者会篡改那些信息。这就是为什么最好产生一个特定的名称然后使用 [guessExtension()()](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/File/UploadedFile.html#guessExtension()()) 方法让 Symfony 根据文件的 MIME 类型去猜测正确的扩展名。  

4. **UploadedFile** 类也提供 [move()()](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/File/UploadedFile.html#move()()) 方法来储存在预先的目录下的文件。将这个目录路径定义为一个应用程序的配置选项是很好的选择并且这也简化了代码：**$this->container->getParameter('brochures_dir')**。

现在你可以使用下列代码来为你的产品链接一个 PDF 小册子了：  

```
<a href="{{ asset('uploads/brochures' ~ product.brochure) }}">View brochure (PDF)</a>
```