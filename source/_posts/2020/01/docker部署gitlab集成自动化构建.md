---
title: docker部署gitlab集成自动化构建
comments: true
date: 2020-01-13 10:54:09
updated: 2020-01-13 10:54:09
tags:
---

<blockquote class="blockquote-center">使用gitlab搭建内网代码仓库</blockquote>

<!--more-->

{% note info %}
# 一、认识Gitlab
{% endnote %}

GitLab 是一个用于仓库管理系统的开源项目，使用Git作为代码管理工具，并在此基础上搭建起来的Web服务,和github是一样的，不过我们可以将它部署在内网，放自己私有项目

{% note info %}
# 二、Docker安装Gitlab
{% endnote %}

## 下载gitlab镜像

```bash
# 最新版
docker pull gitlab/gitlab-ce:latest
# 指定版本，因为我要恢复之前数据，就不下最新版了（恢复数据要保证版本一致）
docker pull gitlab/gitlab-ce:11.4.5-ce.0
```

## 启动容器

```bash
docker run -d \ 
           -p 8090:80 \ 
           -p 8443:443 \ 
           --name gitlab \ 
           --restart always \ 
           -v /opt/gitlab/data/:/var/opt/gitlab \ 
           -v /opt/gitlab/log/:/var/log/gitlab \ 
           -v /opt/gitlab/etc/:/etc/gitlab \ 
           gitlab/gitlab-ce:11.4.5-ce.0
```

## 配置gitlab

1、先把gitlab停掉（很重要，不然改gitlab.yml不生效）
```bash
docker exec -it gitlab /bin/bash
gitlab-ctl stop
exit
```

2、修改访问gitlab地址
```bash
vim /opt/gitlab/etc/gitlab.rb

# external_url改成部署机器的域名或者IP地址
external_url 'http://主机IP'
```

3、修改/opt/gitlab/data/gitlab-rails/etc/gitlab.yml

```bash
vim /opt/gitlab/data/gitlab-rails/etc/gitlab.yml
```

找到`GitLab settings`如下图`1-1`
这里就是GitLab`克隆地址`
修改host和port为外部主机的IP和映射的8090端口，不然项目默认80端口，我们主机80端口是访问不到的

{% asset_img GitLabWebServerSettings.png 1-1 %}

4、重启gitlab

```bash
docker exec -it gitlab /bin/bash

gitlab-ctl reconfigure

gitlab-ctl restart
```

打开浏览器，输入本机的ip地址就能访问了

{% note info %}
# 三、备份、恢复Gitlab
{% endnote %}

## 备份

1、查看备份路径

```bash
# 默认路径，主机外部的映射路径是/opt/gitlab/etc/gitlab.rb
vim /etc/gitlab/gitlab.rb

# 路径
gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"
```

2、创建备份

```bash
docker exec -it gitlab /bin/bash

/opt/gitlab/bin/gitlab-rake gitlab:backup:create
```
3、恢复备份

```bash
# 复制文件到 backups
mv 1576721744_2019_12_18_11.4.5_gitlab_backup.tar /opt/gitlab/data/backups
# 修改权限
chmod 777 1530773117_2018_07_05_gitlab_backup.tar
# 进入容器
docker exec -it gitlab /bin/bash
# 停掉数据连接服务
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq
# 恢复然后两次yes
gitlab-rake gitlab:backup:restore BACKUP=1530773117_2018_07_05_gitlab_backup.tar
# 重启
gitlab-ctl restart
```
PS:根据版本不同恢复时可能有点小区别
可能不加`_gitlab_backup.tar`后缀，具体可以看报错信息

**注意：通过备份文件恢复gitlab必须保证gitlab版本一致**

{% note info %}
# 三、gitlab-runner实现自动化构建
{% endnote %}

## docker安装gitlab-runner

```bash
# 拉取
docker pull gitlab/gitlab-runner
# 启动
mkdir -p /opt/gitlab-runner/config
docker run -d \ 
           --name gitlab-runner \ 
           --restart always \ 
           -v /opt/gitlab-runner/config:/etc/gitlab-runner \ 
           -v /var/run/docker.sock:/var/run/docker.sock \ 
           gitlab/gitlab-runner:latest
```
## 注册gitlab-runner

1、注册

```bash
gitlab-runner register
# 按照顺序输入
# gitlab地址
# gitlab token
# runner说明
# 设置tag，构建文件可以通过tag指定触发runner
# true，是否接受未指定tag的任务
# false，是否锁定这个runner
# 选择runner执行器这里我们选择docker
# 使用构建的镜像这里我们要去构建文档项目使用node:10.15.0
```
gitlab地址和token可以在`设置`-->`runner`找到，如下图1-2

{% asset_img gitlab-runnersetting.png 1-2 %}

2、配置
配置/opt/gitlab-runner/config/config.toml文件
[runner配置文档](https://docs.gitlab.com/runner/configuration/advanced-configuration.html#the-runnersdocker-section)

```bash
vim /opt/gitlab-runner/config/config.toml
# 目前只修改一个挂载目录，用于把构建的文件拿出来，主机要有app文件夹
# volumes 加一个 "/home/app:/mnt/"
```

## 配置一个gitlab构建

1、项目下新建一个`.gitlab-ci.yml`文件

```bash
# 写入以下内容
pages:
  script:
  - echo "开始构建文档" 
  - touch /mnt/test.txt
  only:
    - master
  tags:
   - runner1
```

2、上传一个文件，等待构建完成

3、查看主机映射的文件夹有没有test.txt文件，有证明构建是没有问题的，可以根据自己的项目编写构建文件