---
title: Nginx代理Kafka
comments: true
date: 2020-09-15 18:05:49
updated: 2020-09-15 18:05:49
tags:
  - Nginx
  - Kafka
  - Nginx代理Kafka
---

<blockquote class="blockquote-center">Nginx代理Kafka</blockquote>

<!--more-->

{% note info %}
# 一、第一种方式：ngx_kafka_module
{% endnote %}


这种方式主要是能够通过http给kafka发送消息

## 安装依赖

- [librdkafka](https://github.com/edenhill/librdkafka)

### 在线安装
```bash
# Mac OSX
brew install librdkafka
# Debian和Ubuntu
apt install librdkafka-dev
# RedHat，CentOS，Fedora
yum install librdkafka-devel
```

### 离线安装

```bash
# 下载源码包并上传
https://github.com/edenhill/librdkafka/releases

# 解压进入目录
tar -zxvf librdkafka-1.5.0.tar.gz
cd librdkafka-1.5.0

# 编译安装
# 可通过--prefix=/home/librdkafka指定目录）
# 默认在
./configure
 make && make install
```

## 配置链接库

**在线安装不需要这一步**

 make && make install默认是把动态库安装到/usr/local/lib下的，所以在Linux的默认共享库路径/lib和/usr/lib下找不到。因此添加的library如果不在/lib和/usr/lib里面的话，就需要往/etc/ld.so.conf文件追加library所在的路径，然后重新调用下ldconfig命令即可。

```bash
echo "/usr/local/lib" >> /etc/ld.so.conf
ldconfig
```


## 安装Nginx并添加ngx_kafka_module模块

### 先克隆源码

```bash
# 放在/usr/local/src一会编译nginx会用到
cd /usr/local/src
git clone https://github.com/brg-liuwei/ngx_kafka_module
```

### 安装Nginx

```bash
# 下载
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
mkdir /usr/local/nginx
mkdir /home/nginx
cd /home/nginx
wget -c http://nginx.org/download/nginx-1.16.1.tar.gz
tar -zxvf nginx-1.16.1.tar.gz
cd nginx-1.16.1

# 编译安装
./configure --prefix=/usr/local/nginx --with-http_ssl_module --add-dynamic=/usr/local/src/ngx_kafka_module

make && make install

# 启动
/usr/local/nginx/sbin/nginx


# 如果报错请看下一步 复制objs/ngx_http_kafka_module.so
```

### 复制objs/ngx_http_kafka_module.so

> 上一步报错才需要执行复制操作
> 可以先查看/usr/local/nginx/modules 是否存在模块，不存在再复制
> 为现有nginx添加模块时，module不会自动创建并复制模块

配置完kafka启动nginx时发现找不到ngx_http_kafka_module.so，所以我们手动复制一下

```bash
mkdir /usr/local/nginx/modules

\cp ./objs/ngx_http_kafka_module.so /usr/local/nginx/modules
```

### Kafka的Nginx配置

```bash

worker_processes  1;

# 动态安装的模块需要引入
load_module "modules/ngx_http_kafka_module.so";

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    kafka;
    kafka_broker_list 192.168.1.140:9092;

    server {
        listen       9032;
        server_name  192.168.1.140;
        location = /kafka/server-per-final {
                kafka_topic server-per-final;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```
{% note info %}
# 二、第二种方式：配置stream模块
{% endnote %}


这种方式主要实现整个kafka消息的代理转发，我们此次项目就使用这种方式


## 安装Nginx

```bash
# 安装
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
mkdir /usr/local/nginx
mkdir /home/nginx
cd /home/nginx
wget -c http://nginx.org/download/nginx-1.16.1.tar.gz
tar -zxvf nginx-1.16.1.tar.gz
cd nginx-1.16.1

./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-stream
make && make install

# 启动
/usr/local/nginx/sbin/nginx

```

## Kafka的Nginx配置
```bash
stream {
    upstream kafka {
        server 192.168.1.140:9092 weight=1;
    }

    server {
        listen 9032;
        proxy_connect_timeout 1s;
        proxy_timeout 6s;
        proxy_pass kafka;
    }
}
```