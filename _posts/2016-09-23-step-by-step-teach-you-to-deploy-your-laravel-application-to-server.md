---
layout: post
permalink: /step-by-step-teach-you-to-deploy-your-laravel-application-to-server/
title: 一步一步教你部署自己的 Laravel 应用程序到服务器
date: 2016-09-23T23:18:00
lang: zh-CN
---

在部署自己的博客到 LEMP 环境的时候，遇到了一些小挫折，现在把经验分享出来，让大家少走弯路。包括Php7.1安装与下载，SSL证书申请与配置，Mysql升级到5.7，Nginx服务器的简单配置。`Let's start`.

> *注：本教程环境：Ubuntu 14.04.4 LTS (GNU/Linux 3.13.0-86-generic x86_64)*

## 安装 Php 7.1


首先用SSH连接到你的服务器，Windows 用户建议使用 SecureCRT 来连接，这是一个收费软件，请自行搜索 ~~破解版~~ 。

#### 1. 准备

安装 Php7.1 之前，要先安装`language-pack-en-base`这个包，运行：

```
sudo apt-get update
sudo apt-get install -y language-pack-en-base
```

这个包是为了解决系统不同语言之间可能发生的冲突，安装之后可以减少许多因语言编码带来的问题。其中`-y`参数表明直接安装，无需确认。

安装完成之后，运行：
```
locale-gen en_US.UTF-8
```
设定语言编码为`UTF-8`。


接下来安装Git，Vim，运行：
```
sudo apt-get install git vim
```
Git 是必备的，你可以很方便的使用Git来拉取远程仓库的代码，vim是一款强大的编辑器，可以帮助你修改一些配置文件，如`.env`文件，如果的你的服务器已经安装了vim编辑器，可以忽略。


进入正题，安装Php7.1，本教程采用ppa方式安装php7.1，运行：
```
sudo apt-get install software-properties-common
```
`software-properties-common`是`add-apt-repository`所依赖的包，安装成功后，运行：
```
sudo LC_ALL=en_US.UTF-8 add-apt-repository ppa:ondrej/php
```
来添加php7的ppa，注意`LC_ALL=en_US.UTF-8`参数告诉我们系统语言为UTF-8，如果没有，可能会出现错误，如阿里云的服务器。

安装完成之后，运行`sudo apt-get update`更新安装包，把刚才添加的包拉取下来。
运行`apt-cache search php7.1`搜索php7.1开头的包检验是否安装成功，输出如下：

```
root@demo:~# apt-cache search php7.1
php-yaml - YAML-1.1 parser and emitter for PHP
php-apcu - APC User Cache for PHP
php-ssh2 - Bindings for the libssh2 library
php-igbinary - igbinary PHP serializer
php-mailparse - Email message manipulation for PHP
php-libsodium - PHP wrapper for the Sodium cryptographic library
php-propro - propro module for PHP
php-raphf - raphf module for PHP

....................省略...........................
```


#### 2. 安装：

安装php7.1:
```
sudo apt-get -y install php7.1
```
安装成功后运行`php -v`查看是否安装成功，成功的话会输出类似如下信息：

```
PHP 7.1.0beta2 (cli) ( NTS )
Copyright (c) 1997-2016 The PHP Group
Zend Engine v3.1.0-dev, Copyright (c) 1998-2016 Zend Technologies
    with Zend OPcache v7.1.0beta2, Copyright (c) 1999-2016, by Zend Technologies
```

安装php7.1-mysql，这是 Php7.1 与 mysql 的通信模块：

```
sudo apt-get -y install php7.1-mysql
```

安装 fpm，这是Nginx 用来解析php文件的：


```
sudo apt-get install php7.1-fpm
```

安装其他必备模块：

```
apt-get install php7.1-curl php7.1-xml php7.1-mcrypt php7.1-json php7.1-gd php7.1-mbstring
```

至此与php相关的模块安装安装完成。



##   安装Mysql

直接安装Mysql5.7吧，5.7 可以说是里程碑式的版本，提高了性能，并增加了很多新的特性。特别是新增加的`json`字段，用过之后你会爱上她的！！

> MySQL 开发团队于 9.12 日宣布 MySQL 8.0.0 开发里程碑版本（DMR）发布！但是目前 8.0.0 还是开发版本，如果你希望体验和测试最新特性，可以从 http://dev.mysql.com/downloads/mysql/  下载各个平台的安装包。不过，MySQL 软件包是越来越大了，Linux 平台上的二进制打包后就将近有 1 GB。如果在产品环境中使用，在 8.0 没有进入稳定版本之前，请继续使用 5.7 系列，当前最新的版本是 5.7.15 GA 版本——这只有 600 M 多。

#### 1. 下载`.deb`包到你的服务器：

```
wget http://dev.mysql.com/get/mysql-apt-config_0.5.3-1_all.deb
```

#### 2. 然后使用`dpkg`命令添加Mysql的源：

```
sudo dpkg -i mysql-apt-config_0.5.3-1_all.deb
```
注意在添加源的时候，会叫你选择安装 MySQL 哪个应用，这里选择 Server 即可，再选择 MySQL 5.7 后又会回到选择应用的那个界面，此时选择 Apply 即可。

#### 3. 安装
```
sudo apt-get update
sudo apt-get install mysql-server
```
安装完成之后运行`mysql -V`查看版本：

```
root@demo:~# mysql -V
mysql  Ver 14.14 Distrib 5.7.15, for Linux (x86_64) using  EditLine wrapper
```

#### 4. `注意`
如果你已经通过 ppa 的方式安装了 MySQL 5.6，首先得去掉这个源。

```
sudo apt-add-repository --remove ppa:ondrej/mysql-5.6
# 如果没有 apt-add-repository 先安装上
# sudo apt-get install software-properties-common
```
然后其它和上面一样，**但最后要运行`sudo mysql_upgrade -u root -p `升级数据库，运行`sudo service mysql restart`重启数据库，这样你的数据会完好无缺（不出意外的话）**。


##  安装Nginx

简单，运行：
```
sudo apt-get -y install nginx
```
即可，然后运行`curl localhost`查看是否运行成功。你也可以直接访问你的IP地址，如下：

![file](https://static.lufficc.com/image/0374bd6a93468c4ac2d3a6429be1e25b.png)



##   配置

#### 1. 配置php：

```
sudo vim /etc/php/7.1/fpm/php.ini
```

输入`/fix_pathinfo`搜索，将`cgi.fix_pathinfo=1`改为`cgi.fix_pathinfo=0`：

![file](https://static.lufficc.com/image/49e0a5f1ddcec2cfab9c1cf6d804a468.png)

> Why ? 假设有如下的 URL：http://phpvim.net/foo.jpg，当访问 http://phpvim.net/foo.jpg/a.php 时，foo.jpg 将会被执行，如果 foo.jpg 是一个普通文件，那么 foo.jpg 的内容会被直接显示出来，但是如果把一段 php 代码保存为 foo.jpg，那么问题就来了，这段代码就会被直接执行。这对一个 Web 应用来说，所造成的后果无疑是毁灭性的。具体参看：http://www.cnblogs.com/buffer/archive/2011/07/24/2115552.html

#### 2. 编辑fpm的配置文件：

运行：

```
sudo vim /etc/php/7.1/fpm/pool.d/www.conf
```

找到`listen = /run/php/php7.1-fpm.sock`修改为`listen = /var/run/php/php7.1-fpm.sock`。当然，你也可以不修改，但必须前后一致，后面会用到这个配置。

![file](https://static.lufficc.com/image/10e90495b7435cf51d31c62806638a4d.png)

#### 3. 配置Nginx：

运行：
```
sudo vim /etc/nginx/sites-available/default
```

主要是配置server这一部分，最终配置如下:
```

server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;

        root /var/www/your-project-name/public;
        index index.php index.html index.htm;

        # Make site accessible from http://localhost/
        server_name lufficc.com www.lufficc.com;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ /index.php?$query_string;
                # Uncomment to enable naxsi on this location
                # include /etc/nginx/naxsi.rules
        }
				
        location ~ \.php$ {
                try_files $uri /index.php =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass unix:/var/run/php/php7.1-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
        }
}
```

解释：

* `root`：是你的项目的public目录，也就是网站的入口
* `index`：添加了，`index.php`，告诉Nginx先解析`index.php`文件
* `server_name`：你的域名，没有的话填写`localhost`
* `location / try_files`修改为了`try_files $uri $uri/ /index.php?$query_string;`
* ` location ~ \.php$`部分告诉Nginx怎么解析Php，原封不动复制即可，**但注意：`fastcgi_pass unix:/var/run/php/php7.1-fpm.sock;`的目录要和fpm的配置文件中的listen一致**。


#### 4. 创建网站目录，Laravel项目：


如果你还没有`/var/www`目录，运行`mkdir  /var/www`，然后将Nginx的用户名和用户组`www-data`分配给它：

```
sudo chown -R www-data:www-data 
```

然后进入到/var/www/目录，`cd /var/www/`。这个www目录就是放置你的网站文件的目录（可以多个）。

下面以laravel-blog为例，先`clone`我的laravel-blog,在`/var/www/`目录下运行：

```
git clone https://github.com/lufficc/laravel-blog.git
```

这会生成在`/var/www/`目录下生成laravel-blog目录，注意此时上面的Nginx配置的`root`要和这个一致，当前配置应该是`root /var/www/laravel-blog/public;`，而不再是~~`root /var/www/your-project-name/public;`~~。

然后`composer update`，配置`.env`，数据库连接，没有安装Redis的话安装Redis:`apt-get install redis-server`。

再次给予目录权限，运行（在`/var/www/laravel-blog`下面）：

```
sudo chmod -R 775 storage/
sudo chown -R www-data:www-data /var/www/laravel-blog
```

运行`ll`，结果如下：
![file](https://static.lufficc.com/image/1f46d3b81d69792bf2f1387748860198.png)


最后！！重启nginx，fpm，访问你的ip地址，不出意外，安装成功，部署完成！
```
sudo service nginx restart
sudo service php7.1-fpm restart
```

##  小结

啰嗦了那么长，其实实际配置起来很简单，把下面的命令复制，依次运行即可完成大部分的配置，然后再简单修改一下配置文件即可：
```

# 安装php7.1
sudo apt-get update
sudo apt-get install -y language-pack-en-base
locale-gen en_US.UTF-8

sudo apt-get install software-properties-common
sudo LC_ALL=en_US.UTF-8 add-apt-repository ppa:ondrej/php
sudo apt-get update

sudo apt-get -y install php7.1
sudo apt-get -y install php7.1-mysql
sudo apt-get install php7.1-fpm

apt-get install php7.1-curl php7.1-xml php7.1-mcrypt php7.1-json php7.1-gd php7.1-mbstring

# 安装mysql5.7
wget http://dev.mysql.com/get/mysql-apt-config_0.5.3-1_all.deb
sudo dpkg -i mysql-apt-config_0.5.3-1_all.deb
sudo apt-get update
sudo apt-get install mysql-server

# 安装nginx
sudo apt-get -y install nginx
```


##   Https（~~对这个没要求的可以不看~~）

> 如果你熟悉Let’s Encrypt，可以跳过了，本教程使用的是免费的，配置更简单的startssl。

去官网注册：https://www.startssl.com/，  startssl免费还是很诱惑人的哈：
![file](https://static.lufficc.com/image/52b2793ee1785c31d0f2a14d55edfeee.png)

#### 1. 验证

登录后之后进去主页，点击Validations Wizard，选择第一项Domain Validation (for SSL certificate)，点击Continue验证你是否是域名的主人：

![file](https://static.lufficc.com/image/5ce05b0c5522097cb349cd9421a4f7b8.png)

![file](https://static.lufficc.com/image/893e9b68b3965679ed9de12d38a9797f.png)

然后点击将验证信息发送到你的邮箱，输入验证码继续

![file](https://static.lufficc.com/image/462470472364caed8ae0c3ecbe56116a.png)

验证成功，startssl认为你就是域名的所有者，此时进行下一步操作：
![file](https://static.lufficc.com/image/d13e7060eb29dcc366d02e5cc2a5f509.png)

#### 2. 选择域名

点击Certificates Wizard，选择第一项：

![file](https://static.lufficc.com/image/d3c080ddb363da9af76578f8e887eef1.png)

输入所有你想要ssl的域名，例如`static`子域名，`blog`子域名:

![file](https://static.lufficc.com/image/bdfe1f06e72b03595b1efb53cdd8b828.png)

#### 3.生成 CSR (Certificate Signing Request ) key：

复制`openssl req -newkey rsa:2048 -keyout yourname.key -out yourname.csr`到你的服务器命令行，：

![file](https://static.lufficc.com/image/fb2bc7300aaa3507d0c4ee09304deaea.png)

填写你的信息：

![file](https://static.lufficc.com/image/142533a60e6d9fe7254fb3a5f9ffb88c.png)

然后会生成两个文件：

![file](https://static.lufficc.com/image/df8c09a26e0ddeb4c6a345737863c27d.png)

运行`cat lufficc.csr`复制`lufficc.csr`的内容到上面的输入框，点击Submit：

![file](https://static.lufficc.com/image/e4528a29ac8ce6c6369364bd481fbea0.png)

此时你的证书已经生成功，点击下载即可下载到本地。

![file](https://static.lufficc.com/image/a5224732514dc52ad98e5495e16aa354.png)

下载完成后解压，会有四个文件，对应不同软件，这里我们只需要Nginx的就行：

![file](https://static.lufficc.com/image/9750c592608d4650e177a59c570c82f5.png)

![file](https://static.lufficc.com/image/73454dec9d07f899348543254de2e757.png)

#### 4. 解密私钥

点击startssl的Tool Box下的Decrypt Private Key，输入`lufficc.key`（刚才命令行生成的.key结尾的文件）文件内容和你刚才设置的密码（刚才命令行设置的密码），点击解密

![file](https://static.lufficc.com/image/ea39a32dd29cdfd1d919d692002186da.png)

复制解压后的内容，运行`vim lufficc_de.key`生成一个新文件，复制进去解密后的key,保存。
然后在新建一个目录`/etc/nginx/ssl`，将刚才的文件放进去，便于统一管理：
![file](https://static.lufficc.com/image/850993ac5555609bc2c10a480d8ff0e0.png)

其中红色框框的是要用到文件。

#### 5. 配置Nginx

运行`sudo vim /etc/nginx/sites-available/default`，翻到最下面注释掉的HTTPS server，最终配置如下：

```
server {
        listen 443;
        server_name lufficc.com www.lufficc.com;

        root /var/www/your-project-name/public;
        index index.php index.html index.htm;

        ssl on;
        ssl_certificate /etc/nginx/ssl/1_lufficc.com_bundle.crt;
        ssl_certificate_key /etc/nginx/ssl/lufficc_de.key;

        ssl_protocols              TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers  on;
        ssl_session_cache    shared:SSL:10m;
        ssl_session_timeout  24h;


        location / {
                try_files $uri $uri/ /index.php?$query_string;
        }

        location ~ \.php$ {
                try_files $uri /index.php =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass unix:/var/run/php/php7.1-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
        }
```

和http配置基本一样，区别是

```
        ssl on;
        ssl_certificate /etc/nginx/ssl/1_lufficc.com_bundle.crt;
        ssl_certificate_key /etc/nginx/ssl/lufficc_de.key;
```
要和你的目录一致。接下来在80端口的http server中添加跳转，`return      301 https://lufficc.com;`这样当别人用http 协议访问你的站点时，会自动调转到https，如下：
```

server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;

        root /var/www/laravel/public;
        index index.php index.html index.htm;

        # Make site accessible from http://localhost/
        server_name lufficc.com www.lufficc.com *.lufficc.com;
        return      301 https://lufficc.com;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ /index.php?$query_string;
								
 ....................省略...........................
```

最后，测试一行nginx的配置是否正确，运行`sudo service nginx configtest`：

![file](https://static.lufficc.com/image/d24eff42a7ded1b64b5a5e064a0537e6.png)

重新加载并重启nginx：

```
sudo service nginx reload
sudo service nginx restart
```


测试通过，访问你的站点，看有没有加上那把别致的小绿锁？

![file](https://static.lufficc.com/image/354d9b4c7cdc46b7aac0572a7630b5e4.png)

至此，教程结束。最终效果如本站所示。

## 结语

写了那么长的教程，看起来很负责，但只是我啰嗦而已，你把上面的命令一敲，软件安装都很简单，在进行一些简单的配置即可，如果你熟悉整个流程的话，其实没有那么复杂。有问题欢迎在评论指出。

`Have a nice day!`


参考:

* 感谢laravist系列教程以及jellybool对laravel在中国发展的贡献:
https://laravist.com/series/deploy-laravel-app-on-vps

感谢下列教程给予的启发和帮助:

 * php7.1:
http://www.jianshu.com/p/9d4565afa756

* 配置:
http://www.cnblogs.com/buffer/archive/2011/07/24/2115552.html

* MySQL:
https://iyaozhen.com/ubuntu-upgrade-mysql-to-5-7.html
https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-14-04

* startssl:
https://www.freehao123.com/startssl-ssl-apache-ngnix/
