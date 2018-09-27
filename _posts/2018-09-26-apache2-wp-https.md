---
layout: post
title:  "apache2 wordpress https"
date:   2018-09-26 15:50:11 +0800
categories: web
---

### 域名访问 wordpress (http://www.domain.com)

wordpress放到 /var/wwww/html/wordpress 目录下面

- 修改 /etc/apache2/apache2.conf 增加配置

``` shell
    <Directory /var/www/html/wordpress/>
            Options Indexes FollowSymLinks
            AllowOverride All
            Require all granted
    </Directory>
```

- 修改 /etc/apache2/sites-available/000-default.conf

``` shell
    DocumentRoot /var/www/html/wordpress/
```

- wordpress数据修改，进入数据库 修改 home 和 siteurl的内容

`update wp_options set option_value='http://www.domain.com' where option_name="home";`

`update wp_options set option_value='http://www.domain.com' where option_name="siteurl";`

配置完后直接用域名访问wordpress站点，不需要再加上目录名称。


### apache2 wordpress开启https

#### 1. apache2 配置

- 开启ssl模块  `sudo a2enmod ssl`

- 启用SSL站点  `sudo a2ensite /etc/apache2/sites-available/default-ssl.conf`

- /etc/apache2/ 目前下面新建cert文件夹，存放证书文件。如果使用了阿里云服务，证书在阿里云上面下载。

- 修改 /etc/apache2/apache2.conf 增加配置

首先 find /. -name 'mod_ssl.so' 全局搜索有没有

``` shell
    LoadModule ssl_module /usr/lib/apache2/modules/mod_ssl.so
```


- 修改 /etc/apache2/ports.conf 增加443端口

``` shell
    Listen 80 443
```


- 修改 /etc/apache2/sites-available/default-ssl.conf

``` shell
    ServerName www.domain.com
    DocumentRoot /var/www/html/wordpress/

    #证书公钥配置
    SSLCertificateFile /etc/apache2/cert/mypublic.pem
    #证书私钥配置
    SSLCertificateKeyFile /etc/apache2/cert/myprivate.key
    #证书链配置，如果该属性开头有 '#'字符，请删除掉
    SSLCertificateChainFile /etc/apache2/cert/mychain.pem
```

- 修改 /etc/apache2/mods-available/ssl.conf

``` shell
    # 修改加密套件如下
    SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
    SSLHonorCipherOrder on
```

#### 2. wordpress 配置

 进入wordpress安装目录 /var/www/html/wordpress/

 - 修改 /var/www/html/wordpress/.htaccess

 注意重定向配置的顺序

 ``` shell
    # BEGIN WordPress
    <IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteBase /
        RewriteRule ^index\.php$ - [L]
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteRule . /index.php [L]
    </IfModule>
    # END WordPress

        RewriteEngine On
        RewriteCond %{SERVER_PORT} !^443$
        RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
        #将 https 访问强制重定向至 http，代码如下：
        #RewriteCond %{SERVER_PORT} !^80$
        #RewriteRule ^.*$ http://%{SERVER_NAME}%{REQUEST_URI} [L,R=301]
 ```

 - 修改 /var/www/html/wordpress/wp-config.php

 注意 `$_SERVER['HTTPS'] = 'on';`配置的顺序。

 ``` shell
    $_SERVER['HTTPS'] = 'on';

    /** Absolute path to the WordPress directory. */
    if ( !defined('ABSPATH') )
        define('ABSPATH', dirname(__FILE__) . '/');

    /** Sets up WordPress vars and included files. */
    require_once(ABSPATH . 'wp-settings.php');

    /* 强制后台和登录使用 SSL */
    define('FORCE_SSL_LOGIN', true);
    define('FORCE_SSL_ADMIN', true);
 ```

 - wordpress数据修改，进入数据库 修改 home 和 siteurl的内容

`update wp_options set option_value='https://www.domain.com' where option_name="home";`

`update wp_options set option_value='https://www.domain.com' where option_name="siteurl";`

 - wordpress数据修改，进入数据库替换post_content中http的链接为https，此处是网站中导航、资源文件的链接。

 `update wp_posts set post_content = replace(post_content, 'http://www.domain.com','https://www.domain.com');`


 #### 3. 重启apache2
 
 `/etc/init.d/apache2 restart` 或者 `service apache2 restart`