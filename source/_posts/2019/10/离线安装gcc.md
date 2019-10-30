---
title: 离线安装gcc
comments: true
date: 2019-10-30 11:08:00
updated: 2019-10-30 11:08:00
tags:
  - gcc
  - pcre
  - zlib
  - openssl
---

<blockquote class="blockquote-center">在没有网络的情况下离线安装gcc</blockquote>

<!--more-->


{% note info %}
# 一丶安装gcc
{% endnote %}

我们在Linux上源码安装一些软件的时候,需要有gcc环境,如果联网可以直接yum安装,但是部署的时可能是没网络的

 [网易离线包地址](http://mirrors.163.com/centos/6/os/x86_64/Packages/)  
 [阿里离线包地址](http://mirrors.aliyun.com/centos/7/os/x86_64/Packages/)

1. 找到下面的这些包  
mpfr-3.1.1-4.el7.x86_64.rpm  
libmpc-1.0.1-3.el7.x86_64.rpm  
kernel-headers-3.10.0-123.el7.x86_64.rpm  
glibc-headers-2.17-55.el7.x86_64.rpm  
glibc-devel-2.17-55.el7.x86_64.rpm  
cpp-4.8.2-16.el7.x86_64.rpm  
gcc-4.8.2-16.el7.x86_64.rpm  

2. 使用软件打包成tar包上传到服务器上

3. 解压  
`tar -xvf 包名.tar`

4. 统一安装  
`rpm -Uvh *.rpm --nodeps --force`

到此gcc就安装完成了


{% note warning %}
# 二丶拓展安装一些模块
{% endnote %}

我们安装gcc一般是为了安装一些需要源码编译的模块或者软件,比如安装nginx,还要安装许多模块pcre、zlib、openssl这里我们就来拓展一下这些模块的安装


## 安装pcre

> 通过wget一律下载到/usr/local/src目录下,方便寻找,比如安装nginx添加依赖模块就需要用到源码路径

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



