---
title: Nginx添加模块
comments: true
date: 2020-09-16 12:32:04
updated: 2020-09-16 12:32:04
tags:
  - Nginx添加模块
  - Nginx-Module
---

<blockquote class="blockquote-center">Nginx添加模块</blockquote>

<!--more-->
{% note info %}
# 一、查看之前安装的参数
{% endnote %}

```bash
$ /usr/local/nginx/sbin/nginx -V

nginx version: nginx/1.16.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --with-http_ssl_module

# 可以看到最后一行显示之前安装的依赖`--with-http_ssl_module` 记住这一行重新编译nginx的时候会用到

```
{% note info %}
# 二、重新安装
{% endnote %}
```bash
# 进入到nginx之前安装的目录/home/nginx-1.16.1
# 之前的找不到了可以重新下载一个同样版本的
# 可以看到我们此次新增了/usr/local/src/ngx_kafka_module模块
# –add-module=或者--add-dynamic-module
# 一个添加动态模块一个是正常添加模块
 cd /home/nginx-1.16.1
./configure --prefix=/usr/local/nginx --with-http_ssl_module --add-dynamic-module=/usr/local/src/ngx_kafka_module

# 千万不要：make install
make
```

{% note info %}
# 三、替换Nginx
{% endnote %}

```bash
# 停止、备份、覆盖
/usr/local/nginx/sbin/nginx -s stop
\cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.bak
\cp ./objs/nginx /usr/local/nginx/sbin/


# 查看信息
/usr/local/nginx/sbin/nginx -V

nginx version: nginx/1.16.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --with-http_ssl_module --add-dynamic-module=/usr/local/src/ngx_kafka_module

# 启动
/usr/local/nginx/sbin/nginx
```