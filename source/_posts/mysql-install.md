---
title: centos7安装MySql8
top: false
cover: false
toc: true
mathjax: true
tags:
  - MySql8
  - centos7
categories:
  - 服务器配置
date: 2020-07-09 20:26:29
password:
summary:
---

# 前言
在centos服务器安装MySql数据库
+ 操作系统：centos7 x86
+ 型号（规格）：1核 2GB
+ 硬盘：40GB 高IO
+ MySql版本：8.0.20（据说比5.7快2倍）

# 准备
通过ssh登陆服务器

```key
ssh root@124.70.141.52
```
然后输入root对应的密码，登陆服务器

![登陆服务器](login.png)

因为centos7预安装了mariadb，需要先卸载mariadb。

查看是否安装了mariadb
```key
rpm -qa | grep mariadb
```

然后卸载mariadb
```key
rpm -e mariadb-libs-5.5.64-1.el7.x86_64 --nodeps
```

![卸载mariadb](mariadb.png)

# 卸载历史版本MySql服务
## 查看是否拥有历史版本
```shell
rpm -qa|grep mysql
```

## 查看MySql服务状态
```shell
service mysqld status
```

## 暂停MySql服务
```shell
service mysqld stop
```

## 卸载历史版本的MySql服务
```shell
rpm ev [需要移除组件的名称]
或者

# 强制卸载
rpm -e --nodeps [需要移除组件的名称]
```

# 安装MySql服务

可以从[MySql repo源仓库中的yum类型源](http://repo.mysql.com/yum/)中查找指定版本的数据库
```shell
wget http://repo.mysql.com/mysql80-community-release-el7-1.noarch.rpm
```

![yum包](yum.png)
安装rpm包
```shell
sudo rpm -vih mysql80-community-release-el7-1.noarch.rpm
```

yum安装mysql服务
```shell
sudo yum install mysql-server
```

如果显示以下情形则表示安装成功，安装过程中会有两次Y/N的询问，输入Y就行
![成功图片](install.png)
此时默认已经设置为开机启动MySql服务

# 初始化MySql
初始化服务
```shell
mysqld --initialize
```

查看MySql生成的默认密码
```shell
grep 'temporary password' /var/log/mysqld.log
```
![默认密码](password.png)
可以复制root@localhost：后面的密码然后登陆mysql
```shell
mysql -u root -p
```

然后输入之前复制的密码登陆，此时将会显示
```shell
ERROR 2002 (HY000): cant connet to local MySQL server through socket '/var/lib/mysql/mysql.sock'
```

这是因为MySql服务没有安装/运行，或者/var/lib/mysql/mysql.sock这个文件不存在。如果是因为服务没有运行，则开启服务运行

```shell
service mysqld start
```

现在这种情形是因为/var/lib/mysql/mysql.sock文件不存在。需要编辑my.cnf，首先找到my.cnf（一般在/etc/）然后编辑文件，修改为如下
```shell
[mysqld]

datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
user=mysql
## 为了防止安全风险，建议关闭symbolic-links
symbolic-links=0

[mysqld]

socket=/var/lib/mysql/mysql.sock

[client]
socket=/var/lib/mysql/mysql.sock
```
然后重启MySql服务
```shell
systemctl restart mysqld.service
```
这时候报mysqld.service entered failed state这个错误。可以通过
```shell
systemctl status mysqld.service
```
查看错误详情

也可以通过
```shell
journalctl -xe
```
查看完整的日志，但是并不能提供服务启动失败的原因
此时可以通过以下几种方式来进行排查

1、查看/var/lib/mysql文件夹的拥有者和用户组必须是mysql:mysql，而且文件夹的权限必须是700
```shell
ls -ld /var/lib/mysql/
```

2、查看/var/lib/mysql文件夹下面的所有文件夹的拥有者和用户组必须是mysql:mysql，
```shell
ls -lh /var/lib/mysql/
```

3、查看网络tcp端口的监听情况
```shell
netstate -ntlp
```

4、查看mysql的日志文件，查看错误
```shell
cat /var/log/mysql/mysqld.log
```

5、尝试使用以下命令来启动mysql
```shell
mysqld_safe --defaults-file=/etc/my.cf
```

当看到如下场景时说明MySql已经安装完成!
![mysql](success.png)

然后通过以下命令来重置密码
```shell
alter user 'root'@'localhost' identified by 'your_password'
```

最后开启远程连接
```shell
use mysql;
# 修改root账户权限
update user set host = '%' where host = 'root';
# 刷新权限
flush privileges;
```

查看MySql安装在哪个目录
```shell
whereis mysql
```
![whereis](whereis.png)

# References
[1] [CentOS 7安装MySQL8.0](https://www.jianshu.com/p/224a891932d8)
[2] [job for mysqld service failed in centos 7](https://stackoverflow.com/questions/34113689/job-for-mysqld-service-failed-in-centos-7)