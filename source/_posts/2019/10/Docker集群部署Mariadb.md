---
title: Docker集群部署Mariadb
comments: true
date: 2019-10-10 14:03:28
updated: 2019-10-10 14:03:28
tags:
  - Docker
  - Mariadb
  - Linux
  - Docker集群部署Mariadb
---

<blockquote class="blockquote-center">Docker集群部署Mariadb</blockquote>

<!--more-->

{% note info %}
# 一丶主从模式
{% endnote %}

## 前期准备

```bash
# 拉取mariadb镜像
docker pull mariadb:latest

# 创建docker网络 默认bridge驱动
docker network create db-net

# 新建两个目录用来挂载mariadb数据
mkdir -p /home/docker-databases/master-db
mkdir /home/docker-databases/slave-db1
```

## 启动镜像
`--name`：容器别名
`-v`：本地路径:docker容器路径  
`-p`：本地IP:docker容器IP  
`-e`：环境变量  
`-d`：后台运行  
`--name`：容器别名  
`--character-set-server`：设置编码  
`--collation-server`：设置编码  
```bash

# 创建容器的时候报错WARNING: IPv4 forwarding is disabled. Networking will not work.需要开启路由转发
vim  /usr/lib/sysctl.d/00-system.conf
# 添加 路由转发功能
net.ipv4.ip_forward=1
# 重启网络
systemctl restart network

# 启动master-db
docker run \
    --name masterdb \
    --network db-net \
    -v /home/docker-databases/master-db:/var/lib/mysql \
    -p 13306:3306 \
    -e MYSQL_ROOT_PASSWORD=123456 \
    -d mariadb:latest \
    --character-set-server=utf8mb4 \
    --collation-server=utf8mb4_unicode_ci
# 启动slave-db1
docker run \
    --name slavedb1 \
    --network db-net \
    -v /home/docker-databases/slave-db1:/var/lib/mysql \
    -p 13307:3306 \
    -e MYSQL_ROOT_PASSWORD=123456 \
    -d mariadb:latest \
    --character-set-server=utf8mb4 \
    --collation-server=utf8mb4_unicode_ci
```

## 环境准备
```bash
# 进入容器
docker exec -it masterdb bash
docker exec -it slavedb1 bash

# 都更新列表并安装vim
apt-get update
apt-get install vim
```

## 配置Master主服务器`172.18.0.2`

```bash
# 创建并授权slave权限 
# slave---用户名
# 172.18.0.2---本机IP
# 123456---要设置的密码
CREATE USER 'slave'@'172.18.0.2' IDENTIFIED BY '123456';
GRANT REPLICATION SLAVE ON *.*  TO 'slave'@'172.18.0.2';
mysql> create user slave;
mysql> GRANT REPLICATION SLAVE ON *.* TO 'slave'@'172.18.0.2' IDENTIFIED BY '123456';

# 配置/etc/mysql/my.conf----在[mysqld]下面增加下面几行代码
server-id        = 1                   # server-id 必须唯一
log_bin          = master-bin          # 二进制日志
log_bin_index    = master-bin.index
binlog-ignore-db = mysql               # 忽略记录的数据库,多个库用‘,’分隔
binlog-ignore-db = information_schema
binlog-ignore-db = performance_schema
# binlog-do-db = testdb                # 只记录testdb库变化,多个库用‘,’分隔

# 重启容器

# 查看日志
mysql> SHOW MASTER STATUS;
```

## 配置slave从服务器`172.18.0.3`

```bash
# 配置/etc/mysql/my.cnf---在[mysqld]下面增加下面几行代码
server-id        = 2               # server-id 必须唯一
relay_log        = relay-bin       # 中继日志
relay_log_index  = relay-bin.index
# log-slave-updates                # 当做级联复制，或者从库做备份时A-->B-->C，B服务需要开启log-bin和log-slave-updates，二者缺一不可
# expire_logs_days =7              # 当binlog日志较多时，此参数的值意思是只保留7天内的数据。主库及从库都可以配置此参数。
# replication-do-db = testdb       # 只同步testdb库，多个库用‘,’分隔
# replication-ignore-db=mysql      # 不同步mysql库，多个库用‘,’分隔
# replication-do-table = test_tb   # 只同步test_tb表，多个表用‘,’分隔
# replication-ignore-table = test_tb #不同步test_tb表，多个表用‘,’分隔


# 重启容器

# 连接到主服务器
# 172.18.0.2---masterIP
# 3306---master端口
# slave---master创建的用户名
# 123456----slave的密码
# master-bin.000001和329----master执行SHOW MASTER STATUS显示的;
mysql> change master to master_host='172.18.0.2',
master_port=3306,
master_user='slave',
master_password='123456', 
master_log_file='master-bin.000001',
master_log_pos=329;

# 启动Slave  终止:stop slave;
start slave;

# 查看Slave状态
mysql> show slave status \G;

# 关闭写入权限
mysql> SHOW global variables like 'read%';   //查看read_only 状态
+----------------------+--------+
| Variable_name        | Value  |
+----------------------+--------+
| read_buffer_size     | 131072 |
| read_only            | OFF    |
| read_rnd_buffer_size | 262144 |
+----------------------+--------+
 rows in set (0.01 sec) 

 # 设置read_only为只读
 # 当前环境生效，重启后失效。
 mysql> SET GLOBAL read_only=1;    
 # 全局有效
 vim /etc/my.cnf 添加一行read_only=1   
 # 注意：read-only = ON ，这项功能只对非管理员组以为的用户有效！

```

## 测试主从同步

```bash
mysql> CREATE DATABASE IF NOT EXISTS testdb  DEFAULT CHARSET utf8mb4 COLLATE utf8mb4_general_ci;
mysql> use testdb;
mysql> CREATE TABLE `test1` (
  `id` int(11) NOT NULL,
  `name` varchar(100) NOT NULL,
  `create_time` int(11) NOT NULL,
  `update_time` int(11) NOT NULL,
  `delete_time` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

mysql> ALTER TABLE `test1`  ADD PRIMARY KEY (`id`);
mysql> ALTER TABLE `test1`  MODIFY `id` int(11) NOT NULL AUTO_INCREMENT;

mysql> insert into test1 values (null,'tianye',1,1,null);

# Slave服务器上查看数据
mysql> use testdb;
mysql> select * from test1 limit 10;

# 完成
```