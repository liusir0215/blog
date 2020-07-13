---
title: centos7安装apache
top: false
cover: false
toc: true
mathjax: true
tags:
  - apache
  - centos7
categories:
  - 服务器配置
date: 2020-07-13 21:57:55
password:
summary:
---

# 前言
nginx和Apache作为现在非常流行的两大web服务器软件，毋庸置疑都拥有非常优越的性能。
Apache http服务器是使用最广泛的web服务器，它提供了很多优秀的特性，包括动态加载模块，强大的媒体支持以及针对其他流行软件的广泛支持

# 准备
在安装前你需要提前准备一下步骤

+ 服务器上需要有一个拥有sudo权限的非root角色，可以参照[初始化centos设置](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-centos-7)进行设置
+ 针对防火墙进行简单的配置，可以参照[Additional Recommended Steps for New CentOS 7 Servers](https://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-centos-7-servers#configuring-a-basic-firewall)进行配置

# 第一步-安装Apache
centos系统中默认就有Apache的`yum`源，可以通过`yum`包管理器进行安装。

先更新本地Apache `httpd`包到最新版本

```shell
sudo yum update httpd
```

更新完成后安装Apache

```shell
sudo yum install httpd
```

在安装过程中需要进行一次确认，输入yes即可。`yum`将安装Apache和需要的依赖

如果你已经按照上面的步骤配置了防火墙，需要通过以下命令来允许http和https的通信

```shell
sudo firewall-cmd --permanent --zone=public --add-service=http 
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```

# 第二步-验证web服务器
安装完成后Apache不会自动启动，需要手动启动Apache服务

```shell
sudo systemctl start httpd
```

通过以下命令来查看Apache服务状态

```shell
sudo systemctl status httpd.service
```

```shell
Output
Redirecting to /bin/systemctl status httpd.service
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2019-02-20 01:29:08 UTC; 5s ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 1290 (httpd)
   Status: "Processing requests..."
   CGroup: /system.slice/httpd.service
           ├─1290 /usr/sbin/httpd -DFOREGROUND
           ├─1291 /usr/sbin/httpd -DFOREGROUND
           ├─1292 /usr/sbin/httpd -DFOREGROUND
           ├─1293 /usr/sbin/httpd -DFOREGROUND
           ├─1294 /usr/sbin/httpd -DFOREGROUND
           └─1295 /usr/sbin/httpd -DFOREGROUND
...
```

如果Active是`active (running)`则代表是启动状态。当然最好的方法是从浏览器访问域名或者ip地址来进行验证

如果Apache成功安装并且正常运行，在浏览器中通过ip地址访问的时候将会看到以下Apache默认页面

![Apache默认页面](https://assets.digitalocean.com/articles/CART-65406/apache_default_page.png)

如果你不知道当前服务器的公网ip地址，你可以通过以下几种方法来获得

```shell
hostname -I
```

这个命令会把所有的网络地址展示出来，所以你会拿到几个不同分区的地址，你可以在浏览器中对它们分别进行尝试，看看是否有成功的。

另外，你还可以用`curl`请求`icanhazip.com`来拿到当前服务器从公网其他地址访问时的IPv4地址

```bash
curl -4 icanhazip.com
```

# 第三步-管理Apache进程
可以通过以下命令来停止Apache服务

```shell
sudo systemctl stop httpd
```
通过以下命令来重启Apache服务

```shell
sudo systemctl restart httpd
```

为了让以上命令生效，需要重新加载Apache服务。重新加载不会导致连接被丢弃

```shell
sudo systemctl reload httpd
```

当Apache服务安装后会设置为重启自启动，你可以通过以下命令来开启/关闭自启动

```shell
# 关闭自启动
sudo systemctl disable httpd

# 开启自启动
sudo systemctl enable httpd
```

Apache默认配置允许服务器host一个站点，如果想要host多个域名的时候，你需要配置虚拟hosts(virtul hosts)

# 第四步-配置虚拟hosts
你可以通过配置虚拟hosts封装配置细节来host多个站点，在这一步你将配置`example.com`的站点，在你配置的时候需要替换成你真实的域名。

centos上的Apache服务默认就有配置将`/var/www/html`文件夹作为根目录的服务器块。对于单个站点的情况这已经足够，不过当需要host多个站点的时候就不满足了。你可以在`/var/www`文件夹中新建一个`example.com`

# References
[1] [how to install the apache web server on centos7](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-centos-7)