---
layout: master
title: "Mac下使用MAMP搭建WordPress使用环境"
categories: PHP
---

整理了一下这几天在 Mac 下使用 Wordpress 的折腾过程, 感谢神奇的 Homebrew ...

***

## 1. 安装 mySQL5.6+, php7.2+

* 用 Homebrew 安装 mySQL

~~~
brew install mysql
~~~

安装完成后会提示没有设置 root 密码, 需要按照终端提示设置密码, 注意权限问题

* 用 Homebrew 安装 PHP

由于 Mac 系统自带低版本 PHP, 则这一步实为升级

~~~
brew install php@7.2
~~~

安装完成后在终端输入 `php -v` 命令发现还是低版本
这时注意终端的提示的文字

~~~
To enable PHP in Apache add the following to httpd.conf and restart Apache:
    LoadModule php7_module /usr/local/opt/php/lib/httpd/modules/libphp7.so

    <FilesMatch \.php$>
        SetHandler application/x-httpd-php
    </FilesMatch>

Finally, check DirectoryIndex includes index.php
    DirectoryIndex index.php index.html

The php.ini and php-fpm.ini file can be found in:
    /usr/local/etc/php/7.2/

To have launchd start php now and restart at login:
  brew services start php
Or, if you don't want/need a background service you can just run:
  php-fpm

~~~

接下来的操作为:

终端输入

~~~
cd /etc/apache2/
~~~

编辑 httpd.conf 文件

~~~
sudo vim httpd.conf
~~~

在文件末尾粘贴终端提示的两部分

~~~
LoadModule php7_module /usr/local/opt/php/lib/httpd/modules/libphp7.so

<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>
~~~

这时, 在终端输入 `php -v` 命令看到已经升级

***

## 2. 安装 MAMP

* 安装完成后打开 MAMP, 点击 Start Servers, 可以看到 Apache Server 和 MySQL Server 绿灯亮起

* 点击 Open WebStart Page, 跳转到浏览器, 并打开了开始页面

***

## 3. 安装 Wordpress

* 下载并解压 Wordpress 的 zip 包

* 复制解压之后得到的 Wordpress 文件夹到 Apache 默认发布目录 `/Applications/MAMP/htdocs` , 并重命名为你想要的名字, 例如 'banana'

* 这时在浏览器中打开 `localhost:8888/banana` 就可以看到 Wordpress 的安装页面

***

## 4. 配置 Apache 虚拟主机

这里暂停一下 Wordpress 的安装, 为了使用方便, 我们配置一下虚拟主机

* 首先更改 hosts, 在终端输入

~~~
sudo vim /etc/hosts
~~~

* 在后面追加 (其中的 banana 为主机名)

~~~
127.0.0.1    banana
~~~

* 再打开 Apache 同目录下的配置文件

~~~
sudo vim /Applications/MAMP/conf/apache/httpd.conf
~~~

* 末尾追加虚拟主机配置

~~~
# apache 虚拟主机配置文件

NameVirtualHost *

<VirtualHost *>
  ServerName localhost
  DocumentRoot "/Applications/MAMP/htdocs/"
  <Directory "Applications/MAMP/htdocs/">
    Options Indexes FollowSymLinks
    AllowOverride All
    Order allow,deny
    Allow from all
  </Directory>
</VirtualHost>

<VirtualHost *>
  ServerName banana
  DocumentRoot "/Applications/MAMP/htdocs/banana"
  <Directory "Applications/MAMP/htdocs/banana">
    Options Indexes FollowSymLinks
    AllowOverride All
    Order allow,deny
    Allow from all
  </Directory>
</VirtualHost>
~~~

* 此时重启 MAMP 的 Servers , 在浏览中打开 `banana:8888` 就能看到和 `localhost:8888/banana` 相同的页面了

***

## 5. 继续安装 Wordpress

* 在 `banana:8888` 中我们开始安装, 按照提示我们需要填写一个数据库名, 以及能够访问该数据库的用户名和密码, 那么我们需要创建一个数据库

* 在 MAMP 的界面点击 **Open WebStart Page** 将会打开 MAMP 开始页面， 我们在导航中找到 **phpMyAdmin** , 新建一个数据库, 例如 'banana'

* 接下来可以点击该数据库, 在右边的 Privileges 选项卡下创建一个管理员用户, 如果你选择用超级用户(root账户, 密码为root)登陆, 则可以不新建

* 回到 Wordpress 安装页面, 输入数据库名, 用户名和密码, 后面的选项里的 localhost 指代本机, 'wp-' 前缀在一个数据库对应多个 Wordpress 时作区分

* 提交后, 按照提示设置站点标题和网站管理员信息

* 使用刚才注册的管理员账户登陆站点

***


