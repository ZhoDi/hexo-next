---
title: Nginx部署Vue项目
comments: true
date: 2019-10-30 10:26:46
updated: 2019-10-30 10:26:46
tags:
  - Nginx
  - Vue
  - Vue部署
---

<blockquote class="blockquote-center">安装Nginx过程,以及对Vue部署的配置</blockquote>

<!--more-->


{% note info %}
# 一丶安装gcc及其依赖包
{% endnote %}

## 在线安装

```bash
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```

## 离线安装

> 请移步另一篇博客： {% post_link 离线安装gcc %}


{% note info %}
# 二丶安装nginx
{% endnote %}

1. 在线安装

```bash
cd /usr/local
mkdir nginx
cd nginx
wget -c http://nginx.org/download/nginx-1.16.1.tar.gz
tar -zxvf nginx-1.16.1.tar.gz
cd nginx-1.16.1
# 也可添加ssl模块：--with-http_ssl_module
./configure --prefix=/usr/local/nginx 
make && make install
```

2. 离线安装

> **只有./configure --prefix=/usr/local/nginx 这一步不同其他按照上方**
```bash
# --with-pcre=/usr/local/src/pcre-8.35 --with-zlib=/usr/local/src/zlib-1.2.11 这俩默认就依赖就不加了
./configure --prefix=/usr/local/nginx  --with-openssl=/usr/local/src/openssl-1.0.2n
make && make install
```

```bash
```
> **注意:**`--with-***=`后跟下载的源码地址，会自动安装编译，但也可以指定安装路径


3. 启动nginx

```bash
cd ../sbin
# 启动
/usr/local/nginx/sbin/nginx
# 常用命令
/usr/local/nginx/sbin/nginx -t
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
/usr/local/nginx/sbin/nginx -s stop
/usr/local/nginx/sbin/nginx -s reload
```

访问服务器ip查看 能看到welcome to nginx即部署成功


{% note info %}
# 二丶部署vue项目
{% endnote %}

1. 修改conf/nginx.conf配置文件

```bash
# 主要新增配置项
server {
    # 端口
    listen       80;
    # 地址名称
    server_name  localhost;
    # 设置编码格式
    charset utf-8;       
    
    # 前端地址改为html/dist
    location / {
        root   html/dist;
        index  index.html;
        # 重定向页面到使用下面配置的router路由
        try_files $uri $uri/ @router;
    }

    # 将api开头的请求转发到后端的api地址 (.NETCore可选择在启动项里自己配置跨域请求,尽量不使用Nginx代理)
    location /api/ {
      # 后端的真实接口,
      proxy_pass http://10.1.30.50:60003/api/;
    }

    # 由于路由的资源不一定是真实的路径，无法找到具体文件
    # 所以需要将请求重写到 index.html 中，然后交给真正的 Vue 路由处理请求资源
    location @router {
      rewrite ^.*$ /index.html last;
    }
}
```