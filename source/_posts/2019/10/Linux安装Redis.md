---
title: Linux安装Redis
comments: true
date: 2019-10-30 16:04:53
updated: 2019-10-30 16:04:53
tags:
  - Redis
  - Linux
---

<blockquote class="blockquote-center">Linux安装Redis过程记录</blockquote>

<!--more-->

# 一丶安装gcc

先检查是否存在gcc,命令:`gcc --version`.出现gcc版本证明存在,可以跳过此步

### 在线安装

```bash
yum install gcc gcc-c++
```

### 离线安装

> 请移步另一篇博客： {% post_link 离线安装gcc %}

# 二丶安装配置Redis

[Redis下载地址](http://download.redis.io/releases/)

## 安装Redis
```bash
# 创建安装目录
cd /usr/local
mkdir redis
cd redis
# 下载
wget http://download.redis.io/releases/redis-4.0.14.tar.gz

# 解压
tar -zxvf redis-4.0.14.tar.gz
cd redis-4.0.14/src
make
# 安装完成
```

## 配置Reids

```bash
# 开放防火墙6379端口
firewall-cmd --zone=public --add-port=6379/tcp --permanent
firewall-cmd --reload
firewall-cmd --zone=public --query-port=6379/tcp

# 开启远程访问redis

# 编辑配置文件(在redis-4.0.14目录下)
vi redis.conf
# 连接IP, 0.0.0.0 为所有
bind 0.0.0.0
# 限制局域网连接
protected-mode no
# 后台运行
daemonize yes
# 启动redis,带上配置文件
./redis-server ../redis-conf
```
