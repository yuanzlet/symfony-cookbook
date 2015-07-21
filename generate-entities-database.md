# 如何从已存在的数据库中生成实体

当开始使用一个工作于一个全新的项目的数据库，自然而然就有两个不同的结果。大部分情况下，数据库模型的设计和建立从零开始。然而有些时候，您将是从一个已存在且不变的模型上开始。幸运的是，Doctrine 有一大堆的工具来帮助从您已存在的数据中生成模型类。

正如 [Doctrine 工具文档](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/tools.html#reverse-engineering)所说的，逆向工程是开始一个项目的一次性过程。Doctrine 能够转换 70 - 80% 基于领域、表单和外检约束的必要映射信息。Doctrine 不能够发现逆关联、继承类型、作为主键的外键实体或者语义操作关联例如级联或者生命周期事件。在生成实体之后还有一些必要的额外工作来设计每一个适合您的域模型特性。

本教程假设您正在使用一个有以下两个表格的简单的博客应用程序：**blog_post** 和 **blog_comment**。由于外键约束的原因，评论记录与后续记录相链接。

```
CREATE TABLE `blog_post` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `title` varchar(100) COLLATE utf8_unicode_ci NOT NULL,
  `content` longtext COLLATE utf8_unicode_ci NOT NULL,
  `created_at` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

CREATE TABLE `blog_comment` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `post_id` bigint(20) NOT NULL,
  `author` varchar(20) COLLATE utf8_unicode_ci NOT NULL,
  `content` longtext COLLATE utf8_unicode_ci NOT NULL,
  `created_at` datetime NOT NULL,
  PRIMARY KEY (`id`),
  KEY `blog_comment_post_id_idx` (`post_id`),
  CONSTRAINT `blog_post_id` FOREIGN KEY (`post_id`) REFERENCES `blog_post` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```

在投入到教程之前，确保您的数据库连接参数在 **app/config/parameters.yml** 文件中正确设置（或者不管您的数据库配置在哪里），并确保您已经初始化了一个将要群集您实体类的包。在本教程中假设存在并位于 **src/Acme/BlogBundle** 文件夹。

从已存在数据库创建实体类的第一步是让 Doctrine 内省数据库并生成相应的元数据文件。元数据文件描述了在表字段生成的实体类。

```
$ php app/console doctrine:mapping:import --force AcmeBlogBundle xml
```

这个命令行工具让 Doctrine 来内省数据库并在包的 **src/Acme/BlogBundle/Resources/config/doctrine** 文件夹中生成 XML 元数据文件。这生成两个文件：**BlogPost.orm.xml** 和 **BlogComment.orm.xml**。

通过把最后一个参数改为 **yml** 来生成 YAML 格式的元数据文件也是可能的。

生成的 **BlogPost.orm.xml** 元数据文件看起来如下：

```
<?xml version="1.0" encoding="utf-8"?>
<doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping http://doctrine-project.org/schemas/orm/doctrine-mapping.xsd">
  <entity name="Acme\BlogBundle\Entity\BlogPost" table="blog_post">
    <id name="id" type="bigint" column="id">
      <generator strategy="IDENTITY"/>
    </id>
    <field name="title" type="string" column="title" length="100" nullable="false"/>
    <field name="content" type="text" column="content" nullable="false"/>
    <field name="createdAt" type="datetime" column="created_at" nullable="false"/>
  </entity>
</doctrine-mapping>
```

一旦元数据文件生成，您可以通过执行以下两个命令让 Doctrine 创建相关的实体类。

```
$ php app/console doctrine:mapping:convert annotation ./src
$ php app/console doctrine:generate:entities AcmeBlogBundle
```

第一个命令生成注释映射的实体类。但是如果您想使用 YAML 或者 XML 而不是注释，您应该只执行第二个命令。

如果您想使用注释，您可以安全地在运行了这两个命令后删除 XML（或 YAML）文件。

例如，新创建的 **BlogComment** 实体类看起来如下：

```
// src/Acme/BlogBundle/Entity/BlogComment.php
namespace Acme\BlogBundle\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * Acme\BlogBundle\Entity\BlogComment
 *
 * @ORM\Table(name="blog_comment")
 * @ORM\Entity
 */
class BlogComment
{
    /**
     * @var integer $id
     *
     * @ORM\Column(name="id", type="bigint")
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="IDENTITY")
     */
    private $id;

    /**
     * @var string $author
     *
     * @ORM\Column(name="author", type="string", length=100, nullable=false)
     */
    private $author;

    /**
     * @var text $content
     *
     * @ORM\Column(name="content", type="text", nullable=false)
     */
    private $content;

    /**
     * @var datetime $createdAt
     *
     * @ORM\Column(name="created_at", type="datetime", nullable=false)
     */
    private $createdAt;

    /**
     * @var BlogPost
     *
     * @ORM\ManyToOne(targetEntity="BlogPost")
     * @ORM\JoinColumn(name="post_id", referencedColumnName="id")
     */
    private $post;
}
```

正如您可以看到的，Doctrine 将所有的表字段转化成为纯私有和带注释的类属性。最令人印象深刻的是它同样发现了与基于外键约束的 **BlogPost** 实体类的关系。因此，您可以在 **$post** 实体类中找到用 **BlogPost** 实体映射的一个私有的 **$post** 属性。

如果您想有一对多的关系，您需要手动将其添加到实体或者生成的 XML 或 YAML 文件。在具体的实体上添加一个区段使一对多定义 **inversedBy** 和 **mappedBy** 块。

现在可以使用生成的实体了。玩得开心！
