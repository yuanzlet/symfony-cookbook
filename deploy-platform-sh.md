# 部署在 Platform.sh

此按部就班的指导书描述如何将 Symfony 网页应用程序部署到 [Platform.sh](https://platform.sh/)。你可以在官方的 [Platform.sh 文件](https://docs.platform.sh/)中阅读更多关于在Platform.sh上使用 Symfony 的说明。

## 部署已存在的网站

本指南中已假设您的代码库的版本中包含 Git。

### 获取一个 Platform.sh 项目

您需要订阅一个 [Platform.sh 项目](https://marketplace.commerceguys.com/platform/buy-now)。选择发展计划并完成校验过程。一旦您的项目准备好了，为其命名并选择: **Import an existing site**。

### 准备应用程序

若要在 Platform.sh 上部署 Symfony 应用程序，您只需在 Git 存储库的根目录里添加一个 **platform.app.yamlat**，存储库会使 Platform.sh 部署您的应用程序 (阅读更多关于 [Platform.sh 配置文件](https://docs.platform.sh/))

```
# .platform.app.yaml

# This file describes an application. You can have multiple applications
# in the same project.

# The name of this app. Must be unique within a project.
name: myphpproject

# The toolstack used to build the application.
toolstack: "php:symfony"

# The relationships of the application with services or other applications.
# The left-hand side is the name of the relationship as it will be exposed
# to the application in the PLATFORM_RELATIONSHIPS variable. The right-hand
# side is in the form `<service name>:<endpoint name>`.
relationships:
    database: "mysql:mysql"

# The configuration of app when it is exposed to the web.
web:
    # The public directory of the app, relative to its root.
    document_root: "/web"
    # The front-controller script to send non-static requests to.
    passthru: "/app.php"

# The size of the persistent disk of the application (in MB).
disk: 2048

# The mounts that will be performed when the package is deployed.
mounts:
    "/app/cache": "shared:files/cache"
    "/app/logs": "shared:files/logs"

# The hooks that will be performed when the package is deployed.
hooks:
    build: |
      rm web/app_dev.php
      app/console --env=prod assetic:dump --no-debug
    deploy: |
      app/console --env=prod cache:clear
```

最佳的做法是，您应该在 Git 存储库的根目录下面添加一个包含 **.platform** 文件的 文件夹：

```
# .platform/routes.yaml
"http://{default}/":
    type: upstream
    # the first part should be your project name
    upstream: "myphpproject:php"

# .platform/services.yaml
mysql:
    type: mysql
    disk: 2048

```

您可以在 [GitHub](https://github.com/platformsh/platformsh-examples) 上找到此类配置的示例。 Platform.sh 文件中包含有[可用服务](https://docs.platform.sh/#configure-services)列表：

### 配置数据库入口

Platform.sh 将通过导入以下文件重写您数据库的特定配置 (您需要自主添加下列文件到您的代码库)：

```
// app/config/parameters_platform.php
<?php
$relationships = getenv("PLATFORM_RELATIONSHIPS");
if (!$relationships) {
    return;
}

$relationships = json_decode(base64_decode($relationships), true);

foreach ($relationships['database'] as $endpoint) {
    if (empty($endpoint['query']['is_master'])) {
      continue;
    }

    $container->setParameter('database_driver', 'pdo_' . $endpoint['scheme']);
    $container->setParameter('database_host', $endpoint['host']);
    $container->setParameter('database_port', $endpoint['port']);
    $container->setParameter('database_name', $endpoint['path']);
    $container->setParameter('database_user', $endpoint['username']);
    $container->setParameter('database_password', $endpoint['password']);
    $container->setParameter('database_path', '');
}

# Store session into /tmp.
ini_set('session.save_path', '/tmp/sessions');
```

请确保您输入了下列文件：

```
# app/config/config.yml
imports:
    - { resource: parameters_platform.php }
```

### 部署您的应用程序

现在您需要在 Git 代码库中添加一个 Platform.sh 的远程指令(复制您在 Platform.sh web UI 上看到的指令)：

```
$ git remote add platform [PROJECT-ID]@git.[CLUSTER].platform.sh:[PROJECT-ID].git
```

**PROJECT-ID**

给您的项目添加唯一标识符。就像 **kjh43kbobssae** 一样。

**CLUSTER**

部署您项目所在的服务器位置。它可以是 **eu** 或 **us**。执行前一节中创建的 Platform.sh 特定文件：

```
$ git add .platform.app.yaml .platform/*
$ git add app/config/config.yml app/config/parameters_platform.php
$ git commit -m "Adding Platform.sh configuration files."
```

将您的代码库推送给新添加的远程指令：

```
$ git push platform master
```

就是这样！您的应用程序正在被部署到 Platform.sh 上，您很快就能够在您的浏览器中访问它了。

redeploy your environment on Platform.sh.
从现在起，您做出的每一个代码变更都将会推送到 Git，以便重新调配您的 Platform.sh 环境。

有关[迁移数据库和文件](https://docs.platform.sh/)的详细信息可在 Platform.sh 文件中查看。

## 部署一个新站点

您可以开始一个新的 [Platform.sh 项目](https://marketplace.commerceguys.com/platform/buy-now)。选择发展计划并完成校验过程。

一旦您的项目准备就绪，为其命名并选择:：**Create a new site**。选择*Symfonystack* 和一个类似 *Standard* 的起始点。

就是这样！您的 Symfony 应用程序将自主运行并进行配置。您很快就能够在您的浏览器中看到它。



