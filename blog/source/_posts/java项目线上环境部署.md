---
title: Java项目线上环境部署
date: 2019-08-26
categories: java
tags: [java,mmall,部署]
---

## 系统环境

* 腾讯云 CentOS Linux release 7.6.1810（不能使用阿里云镜像源）

## 用户创建

### 1、登录服务器、输入密码

```shell
[xiaomian@~]# ssh root@49.235.152.58
root@49.235.152.58's password:
```

### 2、切换阿里云软件配置（可选）

> 阿里云软件源配置说明，本教程所用centos：https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.3e221b11LV3jvx

配置方法

1. 备份
```shell
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```
2. 下载新的 CentOS-Base.repo 到 /etc/yum.repos.d/

CentOS 6
```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
```
或者
```
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
```

CentOS 7

```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```
或者
```
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

CentOS 8
```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-8.repo
```
或者
```
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-8.repo
```

3. 运行 yum makecache 生成缓存

4. 其他

非阿里云ECS用户会出现 Couldn't resolve host 'mirrors.cloud.aliyuncs.com' 信息，不影响使用。用户也可自行修改相关配置: eg:
```
sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
```

### 创建用户

1、创建带有sudo权限的用户（-d和-m是为用户bestmian创建一个主目录）
```
useradd -d /usr/bestmian -m bestmian

cd /usr/bestmian/
```

2、重置bestmian密码
```
passwd bestmian
```

3、编辑权限
```
sudo vim /etc/sudoers
```
找到root（通过:noh 取消高亮），通过(:wq!)保存退出
```
root ALL=(ALL) ALL
bestmian ALL=(ALL) ALL
```

4、退出root用户，登录bestmian用户
```
exit

ssh bestmian@49.235.152.58
```

## JDK

### 1、查看当前环境有没有安装jdk、如果有安装，使用 yum remove 删除即可

```shell
[root@VM_0_14_centos ~]# sudo rpm -qa | grep jdk
[root@VM_0_14_centos ~]# 
```

### 2、下载安装包

根目录创建文件夹
```
sudo mkdir develop

wget http://learning.happymmall.com/jdk/jdk-7u80-linux-x64.rpm
```

## Maven

### 1、下载apache-maven-3.0.5到/opt目录

```
[root@VM_0_14_centos opt]# wget http://learning.happymmall.com/maven/apache-maven-3.0.5-bin.tar.gz
```

### 2、添加环境变量

```
[root@VM_0_14_centos opt]# vim /etc/profile
```
编辑/etc/profile
```
export JAVA_HOME=/usr/java/jdk1.8.0_11
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin
export MAVEN_HOME=/opt/apache-maven-3.0.5
export LIBJLI_PATH=/usr/java/jdk1.8.0_11/jre/lib/amd64/jli
export CATALINA_HOME=/opt/apache-tomcat-7.0.73
export PATH=$PATH:$JAVA_PATH:$MAVEN_HOME/bin:/usr/local/nginx/sbin
```
使/etc/profile生效
```
[root@VM_0_14_centos opt]# source /etc/profile
```

## Mysql

### 1、查看当前环境有没有安装mysql

```shell
[root@VM_0_14_centos ~]# sudo rpm -qa | grep mysql-server
[root@VM_0_14_centos ~]# 
```

### 2、安装mysql，mysql5.7参考链接

https://www.cnblogs.com/nicknailo/articles/8563737.html

```shell
[root@VM_0_14_centos ~]# yum install -y mysql-server
[root@VM_0_14_centos ~]#
```

### 3、编辑my.cnf

```
sudo vim /etc/my.cnf
```
增加character-set-server=utf8、default-character-set=utf8
```
[mysqld]


datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
character-set-server=utf8
default-character-set=utf8
```

### 4、mysqld自启动

```
chkconfig mysql on
```
service和chkconfig命令的功能好像都被阉割了，而且好像已经被systemctl命令取代了。
https://blog.csdn.net/cds86333774/article/details/51165361

```
[root@master ~]# systemctl is-enabled mysqld.service
disabled
```

```
mysql -u root -p

[mysql]

show databases;

create databases mmall;

use mmall;

show tables;

source /app/mmall/data/mmall_learn.sql;
```

## Git

### 1、git安装依赖

```shell
sudo yum -y install zlib-devel openssl-devel cpio expat-devel gettext-devel curl-devel perl-ExtUtils-CBuilder perl-ExtUtils- MakeMaker 
```

## Nginx

### 1、nginx安装依赖命令

```shell
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel 
```

## 防火墙配置

```
cd /etc/sysconfig/

[root@VM_0_14_centos sysconfig]# ll | grep ipt
-rw-------. 1 root root 2374 Nov  5  2018 iptables-config
drwxr-xr-x. 2 root root 4096 Sep  2  2019 network-scripts
[root@VM_0_14_centos sysconfig]# iptables -P OUTPUT ACCEPT
[root@VM_0_14_centos sysconfig]# service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]
[root@VM_0_14_centos sysconfig]# mv iptables iptables.bak
[root@VM_0_14_centos sysconfig]# wget http://learning.happymmall.com/env/iptables

[root@VM_0_14_centos sysconfig]# systemctl restart iptables
```

>关于centos 7 中service iptables save 指令使用失败的结局方案 

在刚买的ceno 7服务器中安装vsftpd之后想打开防火墙端口  
结果/etc/sysconfig/目录下没有iptables文件  
这时候就需要自己写一个iptables文件并且写入相关指令  
然后使用 service iptables save 时显示 The service command supports only basic LSB actions (start, stop, restart, try-restart, reload, force-reload, status). For other actions, please try to use systemctl.
百思不得其解，然后上网百度之后，找到了解决方法：
首先不管防火墙有没有关 都使用 `systemctl stop firewalld` 关闭防火墙
然后使用 `yum install iptables-services` 安装或更新服务
再使用 `systemctl enable iptables` 启动iptables
最后 `systemctl start iptables` 打开iptables
大功告成
试试 `service iptables save`

### linux下执行防火墙相关指令报错
```
Redirecting to /bin/systemctl restart iptables.service
```
1、安装systemctl:
```
yum install iptables-services
```
2、设置开机启动
```
systemctl enable iptables.service
```
3、然后可以执行以下指令
```
systemctl stop iptables
systemctl start iptables
systemctl restart iptables
systemctl reload iptables
```


## vsftp

```
usermod -d /ftp ftpname //更改用户ftpname的主目录为/ftp

systemctl restart vsftpd.service
```

查看 /etc/vsftpd/vsftpd.conf 查到 pam_service_name=vsftpd ，可知认证pam 文件位于 /etc/pam.d/vsftpd
查看 /etc/pam.d/vsftpd 看到一行 auth required pam_shells.so ,因为创建ftp账户时候，禁止了ssh登陆 所以此处应该改为 pam_nologin.so
重启 systemctl vsftpd.service restart ，可以正常登陆。

* [CentOS 6.8 安装配置 vsftpd 文件服务器](https://blog.csdn.net/Andy86869/article/details/78659588)
* [vsftp 服务连接报530 login incorrect](https://blog.csdn.net/changshaoshao/article/details/100009123)
* [linux ftp服务器设置，只允许用户访问指定的文件夹，禁止访问其他文件夹](https://blog.csdn.net/ever_peng/article/details/80161929?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-3&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-3)
* [慕课网Java电商项目部署](http://learning.happymmall.com/)