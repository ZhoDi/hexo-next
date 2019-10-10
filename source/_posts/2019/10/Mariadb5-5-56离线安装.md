---
title: Mariadb5.5.56离线安装
comments: true
date: 2019-10-10 14:08:41
updated: 2019-10-10 14:08:41
tags:
  - Mariadb
  - Linux
---

<blockquote class="blockquote-center">解决在没有网络情况下安装Mariadb,比如项目部署时</blockquote>

<!--more-->
{% note info %}
# Mariadb5.5.56离线安装
{% endnote %}

## 离线安装

```bash
  # 解压离线rpm包

  # 原生CentOS7需要安装这个,红旗不需要请忽略,如果下面一条命令报错需要libpcap也可执行
  rpm -ivh libpcap-1.5.3-8.axs7.x86_64.rpm

  # 安装mariadb的rpm依赖包
  rpm -ivh rsync-3.0.9-17.axs7.x86_64.rpm nmap-* lsof-4.87-4.axs7.x86_64.rpm perl-* boost-*

  # 搜索 没有搜索结果直接跳过下一步卸载
  rpm -qa | grep mariadb-libs
  # 卸载 将搜索结果复制到卸载命令
  rpm -ev --nodeps [搜索结果]

  # 安装mariadb
  rpm -ivh jemalloc-3.6.0-1.el7.x86_64.rpm jemalloc-devel-3.6.0-1.el7.x86_64.rpm
  rpm -ivh galera-25.3.20-1.rhel7.el7.centos.x86_64.rpm
  rpm -ivh MariaDB-10.1.21-centos7-x86_64-common.rpm MariaDB-5.5.56-centos7-x86_64-compat.rpm MariaDB-5.5.56-centos7-x86_64-client.rpm MariaDB-5.5.56-centos7-x86_64-server.rpm
  # 安装完成
```

## 配置Mariadb

### 初始化脚本

  ```bash
  # 开启mysql
  service mysql start
  # 查看状态
  service mysql status
  # 执行mysql初始化脚本
  mysql_secure_installation

  # 输入root密码; 初始安装完没密码直接回车
  Enter current password for root (enter for none): 

  # 为root设置密码; y 设置
  Set root password? [Y/n] y

  # 两次密码一直成功
  New password: 
  Re-enter new password: 
  Password updated successfully!
  Reloading privilege tables..
  ... Success!

  # 是否移除匿名用户; y 移除
  Remove anonymous users? [Y/n] y

  # 是否开启远程登录; y 开启
  Disallow root login remotely? [Y/n] y

  # 是否删除test数据库; y 删除
  Remove test database and access to it? [Y/n] y

  # 是否重新加载权限; y 加载
  Reload privilege tables now? [Y/n] y

  # 完成
  ```

### 配置远程连接权限

  ```bash
  # 登录mysql
  mysql -u root -p
  # 开启远程连接权限
  grant all privileges on *.* to 'root'@'%' identified by 'root密码';flush privileges;
  # 完成
  ```
### 开启3306端口
  ```bash
  firewall-cmd --zone=public --add-port=3306/tcp --permanent
  firewall-cmd --reload
  firewall-cmd --zone=public --query-port=3306/tcp
  ```

### 修改默认字符集

```bash
1）使用vi server.cnf命令编辑server.cnf文件，在[mysqld]标签下添加
  如果/etc/my.cnf.d 目录下无server.cnf文件，则直接在/etc/my.cnf文件的[mysqld]标签下添加以下内容。
  init_connect='SET collation_connection = utf8_unicode_ci' 
  init_connect='SET NAMES utf8'
  character-set-server=utf8 
  collation-server=utf8_unicode_ci 
  skip-character-set-client-handshake

2）文件/etc/my.cnf.d/client.cnf

vi /etc/my.cnf.d/client.cnf

在[client]中添加

default-character-set=utf8

3）文件/etc/my.cnf.d/mysql-clients.cnf

vi /etc/my.cnf.d/mysql-clients.cnf
// 在[mysql]中添加

default-character-set=utf8
// 全部配置完成，重启mariadb

systemctl restart mariadb
 //之后进入MariaDB查看字符集

mysql> show variables like "%character%";show variables like "%collation%";
```

  <p style="text-align:center;">-----------------------------------------安装完成-----------------------------------------</p>
  
---
{% note info %}
# 安装过程记录
{% endnote %}

### 如何获取离线包

- 获取依赖包
  ```bash
    # 安装yum-utils
    yum -y install yum-utils
    # 创建放置rpm包目录
    mkdir -p /home/soft
    # 下载
    yum install [要下载的包名] --downloadonly --downloaddir=/home/soft
  ```

- 获取mariadb包
  [mariadb官网离线npm包](http://ftp.hosteurope.de/mirror/archive.mariadb.org/mariadb-5.5.56/yum/centos/7/x86_64/rpms/)

### 先安装mariadb的rpm依赖包

  ```bash
  # 在线安装 
  yum install rsync nmap lsof perl-DBI nc
  # 离线安装
  rpm -ivh rsync-3.0.9-17.axs7.x86_64.rpm nmap-* lsof-4.87-4.axs7.x86_64.rpm perl-*
  ```

### 安装mariadb
- 安装jemalloc

  ```bash
  # 安装
  rpm -ivh jemalloc-3.6.0-1.el7.x86_64.rpm
  rpm -ivh jemalloc-devel-3.6.0-1.el7.x86_64.rpm
  ```

- 安装 MariaDB-5.5.56-centos7-x86_64-common.rpm
  ```bash
  # 安装  
  rpm -ivh MariaDB-10.1.21-centos7-x86_64-common.rpm

  # 报错如下
  错误：依赖检测失败： 
        mariadb-libs < 1:5.5.56-1.el7.centos 与 MariaDB-common-5.5.56-1.el7.centos.x86_64 冲突

  # 解决,卸载冲突的包
  # 先搜索此包
  rpm -qa | grep mariadb-libs
  # 卸载此包
  rpm -ev --nodeps mariadb-libs-5.5.52-1.axs7.x86_64
  # 再次安装
  rpm -ivh MariaDB-10.1.21-centos7-x86_64-common.rpm
  ```

- 安装 galera-25.3.20-1.rhel7.el7.centos.x86_64.rpm
  ```bash
  #安装
  rpm -ivh galera-25.3.20-1.rhel7.el7.centos.x86_64.rpm

  # 报错如下
  错误：依赖检测失败：
	libboost_program_options.so.1.53.0()(64bit) 被 galera-25.3.20-1.rhel7.el7.centos.x86_64 需要

  # 解决,安装 boost-devel.x86_64
  # 离线
  rpm -ivh boost-*
  # 在线
  yum install boost-devel.x86_64
  # 再次安装
  rpm -ivh galera-25.3.20-1.rhel7.el7.centos.x86_64.rpm
  ```
- 安装 剩下的compat、client、server

  ```bash
  rpm -ivh MariaDB-5.5.56-centos7-x86_64-compat.rpm MariaDB-5.5.56-centos7-x86_64-client.rpm MariaDB-5.5.56-centos7-x86_64-server.rpm
  # 安装完成
  ```

<!-- >

> [mariadb官网离线安装10.x](https://mariadb.com/kb/en/library/mariadb-installation-version-10121-via-rpms-on-centos-7/)

> [mariadb官网离线npm包](http://ftp.hosteurope.de/mirror/archive.mariadb.org/mariadb-5.5.56/yum/centos/7/x86_64/rpms/)

> [安装参考博客](https://www.cnblogs.com/haoliyou/p/10191926.html)

> [获取离线包参考博客](https://blog.csdn.net/topswim/article/details/86578118)

> [获取离线包参考博客](https://blog.csdn.net/GX_1_11_real/article/details/80694556)

> -->