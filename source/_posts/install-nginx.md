---
title: centos7安装nginx
top: false
cover: false
toc: true
mathjax: true
tags:
  - nginx
  - centos7
categories:
  - 服务器配置
date: 2020-07-13 21:57:48
password:
summary:
---

# 前言
nginx和apache作为现在非常流行的两大web服务器软件，毋庸置疑都拥有非常优越的性能。
nginx是一个高性能的web服务器软件，它相对于apache更加的灵活轻量；这篇文章将告诉你如何在centos7上安装并启动nginx。

# 准备
我认为你已通过terminal登录到服务器。安装nginx需要用户有root权限，可以按照[初始化centos7指南](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-centos-7)教程中第三步和第四步来进行设置

# 第一步-增加nginx源
增加CentOS 7 EPEL源，在terminal中输入以下命令

```shell
sudo yum install epel-release
```

# 第二步-安装nginx
通过第一步nginx源已经安装到服务器，现在可以通过yum install来进行安装

```shell
sudo yum install nginx
```
在安装过程中会有一次确认，你回答yes即可。这样nginx将安装到你的vps服务器上

# 第三步-启动nginx
nginx安装后不会自动启动，需要通过以下命令来启动

```shell
sudo systemctl start nginx
```
通过一下命令停止和重启nginx服务

```shell
## 停止nginx服务
sudo systemctl stop nginx

## 重启nginx服务
sudo systemctl restart nginx
```

可以通过

```shell
system status nginx.service
```
命令来查看nginx运行状态
![运行状态](http://img95.699pic.com/element/40102/6907.png_300.png!/clip/0x300a0a0)

启动后如果服务器中有安装防火墙，需要通过以下命令来允许http和https的通信

```shell
sudo firewall-cmd --permanent --zone=public --add-service=http 
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```

你可以通过访问域名或ip来验证服务器apache是否已经安装完毕且运行正常

```shell
http://server_domain_name_or_IP/
```

如果返回如下页面则代表运行正常
![默认页面](http://assets.digitalocean.com/articles/centos/nginx/centos-7-nginx.png)

接下来就是设置nginx在系统重启的时候自启动，可以通过一下命令来实现

```shell
sudo systemctl enable nginx
```

通过

```shell
systemctl is-enabled nginx.service
```
来验证自启动设置是否生效
![自启动](http://img95.699pic.com/element/40102/6907.png_300.png!/clip/0x300a0a0)

恭喜你！nginx服务已经在你的服务器成功安装、运行并且设置服务器自启动

# 如何查看服务器的公网ip
如果你是通过阿里云这种平台购买的服务器，可以通过控制中心查询到公网ip。你还可以通过以下命令来进行查询

```shell
ip addr
```
![ip地址](http://img95.699pic.com/element/40102/6907.png_300.png!/clip/0x300a0a0)

# nginx配置
## 默认根目录
nginx默认的根目录是`/var/www/html`，在这个文件夹中的文件将被作为web页面。这个路径的默认配置是放在`/etc/nginx/conf.d/default.conf`中的。

## 服务器块设置
可增加的服务器块可以在`/etc/nginx/conf.d`文件夹中进行添加。在这个文件夹中文件名是以`.conf`结尾的文件都将在nginx启动后加载

## nginx全局配置
nginx的主要配置信息都是配置在`/etc/nginx/nginx.conf`文件中。在这个文件中可以配置 运行nginx保护进程的用户、nginx运行时产生的工作进程数量 等配置

## 反向代理配置
nginx默认监听的是`80`端口，一般web服务器都会将`80`端口预留给nginx。因为nginx作为网络代理可以进行负载均衡，通过配置将不同请求分发到不同服务器或者同一个服务器的不同端口。可以参照[nginx常用代理配置](https://www.cnblogs.com/fanzhidongyzby/p/5194895.html)来对nginx进行配置

# References
[1] [how to install nginx on centos](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-7)