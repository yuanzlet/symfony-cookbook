# (configuration) 配置一个 Web 服务器


最好的开发你的 Symfony 应用程序的方法就是使用 [PHP 的内部网页服务器](http://symfony.com/doc/current/cookbook/web_server/built_in.html)。然而，当使用老版本的 PHP 时或者当在开发环境运行应用程序时，你就会需要一个全功能的网页服务器。这篇文章介绍了在 Symfony 中使用 Apache 或者 Nginx 的一些方法。  

当使用 Apache 时。你可以将 PHP 设置成 [Apache module](http://symfony.com/doc/current/cookbook/configuration/web_server_configuration.html#web-server-apache-mod-php) 或者使用 [PHP FPM](http://symfony.com/doc/current/cookbook/configuration/web_server_configuration.html#web-server-apache-fpm) FastCGI。FastCGI 也是[用 Nginx](http://symfony.com/doc/current/cookbook/configuration/web_server_configuration.html#web-server-nginx) 使用 PHP 的最好的方法。  

>**网页目录**

>网页目录是你的所有应用程序的公共以及静态文件的家，包括图片，样式表和 JavaScript 文件。它也是前端控制器（**app.php** 和 **app_dev.php**）住的地方。  

>当你配置你的网页目录时，网页目录作为文档的根。在下面的例子中，**web/** 目录将会是文档的根。这个目录是 **/var/www/project/web/**。  

>如果你的虚拟主机请求你将 **web/** 目录更改到另外的位置（例如 **public_html/**）确保你[重写了 web/ 的位置](http://symfony.com/doc/current/cookbook/configuration/override_dir_structure.html#override-web-dir)。  

## 使用 mod_php/PHP-CGI 的 Apache ##

使得你的应用程序在 Apache 下运行的**最小配置**是：  

```
<VirtualHost *:80>
    ServerName domain.tld
    ServerAlias www.domain.tld

    DocumentRoot /var/www/project/web
    <Directory /var/www/project/web>
        AllowOverride All
        Order Allow,Deny
        Allow from All
    </Directory>

    # uncomment the following lines if you install assets as symlinks
    # or run into problems when compiling LESS/Sass/CoffeScript assets
    # <Directory /var/www/project>
    #     Options FollowSymlinks
    # </Directory>

    ErrorLog /var/log/apache2/project_error.log
    CustomLog /var/log/apache2/project_access.log combined
</VirtualHost>
```  

>如果你的系统支持 **APACHE_LOG_DIR** 变量，你就可能想要使用 **${APACHE_LOG_DIR}/** 来代替硬编码 **/var/log/apache2/**。  

使用下面的**可选配置**来禁用 **.htaccess** 支持从而增加网页服务器的效率：  

```
<VirtualHost *:80>
    ServerName domain.tld
    ServerAlias www.domain.tld

    DocumentRoot /var/www/project/web
    <Directory /var/www/project/web>
        AllowOverride None
        Order Allow,Deny
        Allow from All

        <IfModule mod_rewrite.c>
            Options -MultiViews
            RewriteEngine On
            RewriteCond %{REQUEST_FILENAME} !-f
            RewriteRule ^(.*)$ app.php [QSA,L]
        </IfModule>
    </Directory>

    # uncomment the following lines if you install assets as symlinks
    # or run into problems when compiling LESS/Sass/CoffeScript assets
    # <Directory /var/www/project>
    #     Options FollowSymlinks
    # </Directory>

    ErrorLog /var/log/apache2/project_error.log
    CustomLog /var/log/apache2/project_access.log combined
</VirtualHost>
```  

>如果你使用 **php-cgi**，Apache 将默认不会通过 HTTP 基本的 PHP 的用户名和密码。为了取消这个限制，你应当使用下面这一小段配置代码：  
>```
>RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
>```  

### 在 Apache 2.4 下使用 mod_php/PHP-CGI ###

在 Apache 2.4 下，**Order Allow**,**Deny** 已经被 **Require all granted** 所取代。因此，你需要像下面这样修正你的**目录**权限设置：  

```
<Directory /var/www/project/web>
    Require all granted
    # ...
</Directory>
```  

获取更多的有关于 Apache 配置的选项，阅读官方的 [Apache 文档](http://httpd.apache.org/docs/)。  

## Apache 的 PHP-FPM ##

为了使用 Apache 的 PHP5-FPM，首先你必须确定你有安装二进制的 **php-fpm** FastCGI 进程管理器以及 Apache 的 FastCGI 模块（例如，在基于 Debian 的系统中你以及已经安装 **libapache2-mod-fastcgi** 和 **php5-fpm** 包）。  

PHP-FPM 使用一种叫做 *pools* 的东西处理输入的 FastCGI 请求。你可以在 FPM 中设置任意的 pools 的数量。在 pool 中你可以配置监听 TCP 套接字(IP 和 端口)或者 Unix 主机套接字。每一个 pool 也可以在不同的 UID 和 GID 下运行：  

```
; a pool called www
[www]
user = www-data
group = www-data

; use a unix domain socket
listen = /var/run/php5-fpm.sock

; or listen on a TCP socket
listen = 127.0.0.1:9000
```  

### Apache 2.4 下使用 mod_proxy_fcgi ###

如果你使用的是 Apache 2.4，你可以轻松应用 **mod_proxy_fcgi** 来向 PHP-FPM 传递内部请求。配置 PHP-FPM 监听 TCP 套接字（**mod_proxy** 目前[暂时不支持 Unix 套接字](https://issues.apache.org/bugzilla/show_bug.cgi?id=54101)），在你的 Apache 配置中启用 **mod_proxy** 和 **mod_proxy_fcgi** 并且使用 SetHandler 直接将 PHP 文件的请求传递给 PHP FPM：  

```
<VirtualHost *:80>
    ServerName domain.tld
    ServerAlias www.domain.tld

    # Uncomment the following line to force Apache to pass the Authorization
    # header to PHP: required for "basic_auth" under PHP-FPM and FastCGI
    #
    # SetEnvIfNoCase ^Authorization$ "(.+)" HTTP_AUTHORIZATION=$1

    # For Apache 2.4.9 or higher
    # Using SetHandler avoids issues with using ProxyPassMatch in combination
    # with mod_rewrite or mod_autoindex
    <FilesMatch \.php$>
        SetHandler proxy:fcgi://127.0.0.1:9000
    </FilesMatch>

    # If you use Apache version below 2.4.9 you must consider update or use this instead
    # ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://127.0.0.1:9000/var/www/project/web/$1

    # If you run your Symfony application on a subpath of your document root, the
    # regular expression must be changed accordingly:
    # ProxyPassMatch ^/path-to-app/(.*\.php(/.*)?)$ fcgi://127.0.0.1:9000/var/www/project/web/$1

    DocumentRoot /var/www/project/web
    <Directory /var/www/project/web>
        # enable the .htaccess rewrites
        AllowOverride All
        Require all granted
    </Directory>

    # uncomment the following lines if you install assets as symlinks
    # or run into problems when compiling LESS/Sass/CoffeScript assets
    # <Directory /var/www/project>
    #     Options FollowSymlinks
    # </Directory>

    ErrorLog /var/log/apache2/project_error.log
    CustomLog /var/log/apache2/project_access.log combined
</VirtualHost>
```  

### Apache 2.2 的 PHP-FPM ###

在 Apache 2.2 或者更低的版本中，你不能使用 **mod_proxy_fcgi**。你必须使用 [FastCgiExternalServer](http://www.fastcgi.com/mod_fastcgi/docs/mod_fastcgi.html#FastCgiExternalServer) 来代替这个。因此，你的 Apache 配置应当像下面所示：  

```
<VirtualHost *:80>
    ServerName domain.tld
    ServerAlias www.domain.tld

    AddHandler php5-fcgi .php
    Action php5-fcgi /php5-fcgi
    Alias /php5-fcgi /usr/lib/cgi-bin/php5-fcgi
    FastCgiExternalServer /usr/lib/cgi-bin/php5-fcgi -host 127.0.0.1:9000 -pass-header Authorization

    DocumentRoot /var/www/project/web
    <Directory /var/www/project/web>
        # enable the .htaccess rewrites
        AllowOverride All
        Order Allow,Deny
        Allow from all
    </Directory>

    # uncomment the following lines if you install assets as symlinks
    # or run into problems when compiling LESS/Sass/CoffeScript assets
    # <Directory /var/www/project>
    #     Options FollowSymlinks
    # </Directory>

    ErrorLog /var/log/apache2/project_error.log
    CustomLog /var/log/apache2/project_access.log combined
</VirtualHost>
```  

如果你更喜欢使用 Unix 套接字，你必须使用 **-socket** 作为替代：  

```
FastCgiExternalServer /usr/lib/cgi-bin/php5-fcgi -socket /var/run/php5-fpm.sock -pass-header Authorization
```  

## Nginx ##

使得你的应用程序相似 Nginx 下运行的**最小配置**如下所示：  

```
server {
    server_name domain.tld www.domain.tld;
    root /var/www/project/web;

    location / {
        # try to serve file directly, fallback to app.php
        try_files $uri /app.php$is_args$args;
    }
    # DEV
    # This rule should only be placed on your development environment
    # In production, don't include this and don't deploy app_dev.php or config.php
    location ~ ^/(app_dev|config)\.php(/|$) {
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTPS off;
    }
    # PROD
    location ~ ^/app\.php(/|$) {
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTPS off;
        # Prevents URIs that include the front controller. This will 404:
        # http://domain.tld/app.php/some-path
        # Remove the internal directive to allow URIs like this
        internal;
    }

    error_log /var/log/nginx/project_error.log;
    access_log /var/log/nginx/project_access.log;
}
```  

>依赖于你的 PHP-FPM 配置，**fastcgi_pass** 也可以是 **fastcgi_pass 127.0.0.1:9000**。

>这个**只是**在网页目录下执行 **app.php**, **app_dev.php** 和 **config.php**。所有其他的文件都会以文本形式存在。你**必须**确保如果你*确实*配置 **app_dev.php** 或者 **config.php** 使得这些文件安全且不能被任何外界使用者使用（每个文件顶部的 IP 地址检查码默认完成此项工作）。  

>如果在你的网页目录下你有其他的 PHP 文件需要被执行，确保他们包含在上述的 location 区域。  

获取更多的 Nginx 配置选项，阅读官方的 [Nginx 文档](http://wiki.nginx.org/Symfony)。  

