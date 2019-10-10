---
title: ffmpeg+nginx实现rtsp转rtmp
comments: true
date: 2019-10-08 14:12:22
updated: 2019-10-08 14:12:22
tags:
---

<blockquote class="blockquote-center">ffmpeg+nginx实现rtsp转rtmp</blockquote>

<!--more-->

{% note info %}
# 安装
{% endnote %}

## 安装Gcc编译环境

```bash
yum install gcc gcc-c++
```
> 包一律下载到/usr/local/src目录下

## 安装pcre

pcre 的作用是让Nginx支持Rewrite功能。

```bash
# 下载
 wget http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz
# 解压 如果无法解压请下载到本地再上传到服务器
 tar -zxvf pcre-8.38.tar.gz
# 进入目录
cd pcre-8.38
# 安装
./configure
make && make install
```
## 安装zlib

zlib 的作用是让Nginx支持gzip功能。

```bash
# 下载
 wget http://www.zlib.net/zlib-1.2.11.tar.gz
# 解压
 tar -zxvf zlib-1.2.11.tar.gz
# 进入目录
cd zlib-1.2.11
# 安装
 ./configure
 make && make install
```
## 安装openssl
openssl 的作用是让Nginx支持ssl功能。

```bash
# 下载 
 wget https://www.openssl.org/source/openssl-1.0.2n.tar.gz
# 解压
 tar -zxvf openssl-1.0.2n.tar.gz
# 进入目录
cd openssl-1.0.2n
# 安装
 ./config --prefix=/usr/local/openssl
 make && make install
```

## 下载nginx-rtmp-module

```bash
git clone https://github.com/arut/nginx-rtmp-module.git
```

## 安装Nginx

```bash
# 下载 
wget http://nginx.org/download/nginx-1.8.1.tar.gz
# 解压
tar -zxvf nginx-1.8.1.tar.gz
# 进入目录
cd nginx-1.8.1
# 安装
 ./configure --prefix=/usr/local/nginx --add-module=/usr/local/src/nginx-rtmp-module --with-pcre=/usr/local/src/pcre-8.35 --with-zlib=/usr/local/src/zlib-1.2.11 --with-openssl=/usr/local/src/openssl-1.0.2n
 make && make install
```

> **注意:**`--with-pcre=`和`--add-module=`后跟下载的源码路径

## 安装yasm

```bash
# 下载
wget http://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz
# 解压
tar -zxvf yasm-1.3.0.tar.gz
# 进入目录
cd yasm-1.3.0
# 安装
./configure
make && make install
```
## 安装ffmpeg

```bash
# 下载
wget http://www.ffmpeg.org/releases/ffmpeg-4.2.1.tar.gz
# 解压
tar -zxvf ffmpeg-4.2.1.tar.gz
# 进入目录
cd ffmpeg-4.2.1
# 安装
./configure
make && make install
```
{% note info %}
# 配置
{% endnote %}

## 修改nginx配置文件

```bash
vi /usr/local/nginx/conf/nginx.conf


rtmp {    
    server {    
        listen 1935;  #监听的端口  
        chunk_size 4000;
        application live{
            live on;
        }
        application hls {  #rtmp推流请求路径  
            live on;    
            hls on;    
            hls_path /usr/local/nginx/html/hls;    
            hls_fragment 5s;
        }
    }
}
```
## 开启推送
```bash
# 首先开放防火墙端口
firewall-cmd --zone=public --add-port=1935/tcp –permanent

# Rtsp转Rtmp 并推送到Nginx
# 这里的rtsp要用自己的地址 rtsp://用户名:密码@IP地址
ffmpeg -rtsp_transport tcp -i "rtsp://root:pass@10.1.30.11/axis-media/media.amp" -vcodec copy  -acodec copy -f flv "rtmp://127.0.0.1:1935/live/"
```
## 测试

将`rtmp://127.0.0.1:1935/live/`放入软件测试如下图1-1,将127.0.0.1换成你的IP地址

{% asset_img vlc.png 1-1 %}
