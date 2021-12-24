---
layout: post
permalink: /upgrade-to-https-with-certbot/
title: 用 Certbot 一键升级你的网站为 Https
date: 2017-08-15T11:26:00
lang: zh-CN
---

小站以前采用的是 [StartSSL](https://www.startcomca.com/) 的 https 证书，但是 Mozilla 封杀 StartSSL 后一直没有换。前几天终于被 Chrome 报不安全了，于是换成了 [Let’s Encrypt](https://letsencrypt.org/)。


但是 Let’s Encrypt 证书只有90天的有效期，有没有什么便捷的方法一键生成证书呢？答案是 [Certbot](https://certbot.eff.org/)。Certbot 真的是便捷，不用去 Let’s Encrypt 注册账号（它会自动帮你注册），不用手动修改配置服务器配置，一行命令搞定。。。

![file](https://static.lufficc.com/image/0LHo6SoE19a6cqBe0nbYtpZMtsvhR25vG02lvoYZ.png)

以我的服务器为例（Nginx on Ubuntu 14.04），首先安装 Certbot ：
```
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install python-certbot-nginx 
```

安装好之后，只需要运行一行代码：
```
$ sudo certbot --nginx
```
剩下的一切会自动完成。Certbot 会自动帮你注册账户，检测 Nginx 配置文件中的域名，询问你为哪些域名生成证书，是否将 Http 重定向到 Https 等等，最后帮你自动修改 Nginx 配置并重启，这时你的网站已经变成了 Https。另外由于证书只有三个月的有限期，你还可以运行 `sudo certbot renew --dry-run`，Certbot 会帮你启动一个定时任务，在证书过期时自动更新。

还有，如果你只想生成证书怎么办？比如七牛云加速域名的证书，Nginx 配置中并没有此域名。同样一行代码搞定：`certbot certonly --cert-name example.com`。Certbot 会启动一个临时服务器来完成验证（会占用80端口或443端口，因此需要暂时关闭 Web 服务器），然后 Certbot 会把证书以文件的形式保存，包括完整的证书链文件和私钥文件。

你用的是其他操作系统和 Web 服务器怎么办？官方有详细的文档，你可以在首页选择你的软件查看对应的文档。
![file](https://static.lufficc.com/image/16Aby8UHNpYqjr95xM03psGecHPmaV3NTEOM37hp.png)

既然这么简单就可以升级到 Https ,为什么不使用更加安全的 Https 呢？行动起来吧~
