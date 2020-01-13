---
title: Docker从入门到放弃
comments: true
date: 2019-09-12 01:01:30
updated: 2019-09-12 01:01:30
tags:
  - Docker
  - Linux
  - 运维
---

<blockquote class="blockquote-center">Docker</blockquote>

<!--more-->

{% note info %}
# 一、认识Docker
{% endnote %}

Docker简单理解成Linux工厂，可以虚拟出很多linux。然后你可以在里面安装好各种软件和环境，打成包(镜像)，想用的时候可以秒起系统(容器)，然后发现之前配好的环境全部都有了。而且启动快，各个启动的小LINUX（容器）之间完全隔离

{% note info %}
# 二、安装Docker到CentOS7
{% endnote %}

其他版本可以参考文末连接

## 卸载旧版本

```bash
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
## 配置yum镜像加速
```bash
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all
yum makecache
```

## 设置Docker储存库

1、安装所需软件包

```bash
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```
2、设置储存库
```bash
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

## 安装Docker-ce版本（社区版）

1、安装最新版Docker Engine - Community 和 containerd，想安装指定版本看第二步
```bash
$ sudo yum install docker-ce docker-ce-cli containerd.io
```
如果提示您接受GPG密钥，请验证指纹是否匹配 `060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35`，如果是，则接受它。

2、安装指定版本Docker，列出版本：（从高到低）
```bash
$ yum list docker-ce --showduplicates | sort -r

docker-ce.x86_64  3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64  3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64  18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64  18.06.0.ce-3.el7                    docker-ce-stable
```

返回的列表取决于启用的存储库，并且特定于您的CentOS版本（`.el7`代表CentOS7）。
根据软件包名称（`docker-ce`）加上列出的版本字符串（`第二列`），从`:`到第一个连字符`-`
例如：`docker-ce-18.09.1`

```bash
$ sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```
这时Docker已安装但尚未启动。docker会自动创建docker组但不会添加用户进去 

需求：docker 命令与 Docker 引擎通讯之间通过 UnixSocket ，但是能够有权限访问 UnixSocket 的用户只有 root 和 docker 用户组的用户才能够进行访问，所以我们需要建立一个 docker 用户组，并且将需要访问 docker 的用户添加到这一个用户组当中来。

添加用户到docker组
```bash
$ sudo usermod -a -G docker <用户名>
```
查看是否添加进去
```bash
$ sudo groups <用户名>
```

3、启动Docker
```bash
$ sudo systemctl enable docker
$ sudo systemctl start docker
```
4、配置镜像加速

使用[阿里云镜像加速地址](https://cr.console.aliyun.com/)，登录后在左侧找到 **Docker Hub 镜像站点** 然后复制自己的加速地址

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://9ybg794r.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

5、启动一个`hello-world`,验证是否安装成功
```bash
$ sudo docker run hello-world
```
此命令下载测试图像并在容器中运行。容器运行时，它会打印参考消息并退出。

安装完成

### 番外

通常我们的var目录不是很大，但是docker下载的镜像多了很占内存，所以我们换个地方，并且增加一个软连接

```bash
mv /var/lib/docker /home/
ln -s /home/docker/ /var/lib/
### restart docker now
```

安装主要是对官网安装的翻译简化，走一遍完整流程学习记录

{% note info %}
# 三、常用docker命令
{% endnote %}

### 1、修改配置重启
```bash
systemctl daemon-reload
systemctl restart docker
```

### 2、拉取镜像 
```bash
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

### 3、列出镜像 
```bash
docker images [OPTIONS] [REPOSITORY[:TAG]]
```

### 4、列出容器 
```bash
docker ps [OPTIONS]
```
|名称|描述|
|:---|:---|
|`--all , -a`         |显示所有容器（默认显示为正在运行）|

### 5、启动容器
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```
|名称|描述|
|:---|:---|
|`--name`         |容器名称|
|`--detach , -d`  |在后台运行容器并打印容器ID|
|`--publish , -p` |端口映射|
|`--volume , -v`  |目录映射|
|`--restart`      |重启策略，always：始终重启|
|`--env , -e`     |设置环境变量|
|`--rm`           |退出时自动删除容器|

### 6、进入容器
```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```
|名称|描述|
|:---|:---|
|`--interactive , -i` |展示容器输入信息STDIN|
|`--tty , -t`         |命令行交互模式|
|`--detach, -d`       |后台运行模式，在后台执行命令相关命令|
|`--env, -e`          |设置环境变量|

### 7、停止、启动、查看容器
```bash
docker top CONTAINER [ps OPTIONS]
docker start [OPTIONS] CONTAINER [CONTAINER...]
docker stats [OPTIONS] [CONTAINER...]
```

### 8、删除容器

```bash
docker rm CONTAINER
```
|名称|描述|
|:---|:---|
|`--force , -f`   |强制删除正在运行的容器（使用SIGKILL）|
|`--link , -l`    |删除指定的链接|
|`--volumes , -v` |删除与容器关联的卷|

### 9、删除镜像
```bash
docker rmi IMAGE 
```
### 10、拷贝文件
```bash
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH

把本机opt/app 文件夹拷贝到容器opt下
docker cp /opt/app <容器ID/容器名称>:/opt/

把容器opt/app 文件夹拷贝到本机opt下
docker cp  <容器ID/容器名称>:/opt/app /opt/
```

### 11、修改容器配置
```bash
docker update [OPTIONS] CONTAINER [CONTAINER...]
```
|名称|描述|
|:---|:---|
|`--restart`   |重新启动策略|

### 12、修改容器名称
```bash
docker rename CONTAINER NEW_NAME
```

### 13、修改容器配置，比如挂载卷
```bash
# 停止docker
systemctl stop docker
# 修改配置文件config.v2.json和hostconfig.json
#（比如我们修改端口映射，发现两个文件都记录了，那么全部都改，只有一个文件记录了只改一个）
vim /var/lib/docker/containers/<container-ID>/hostconfig.json
# 重启docker
systemctl restart docker
```

> 参考文章

[Docker安装文档](https://docs.docker.com/install/linux/docker-ce/centos/)
[DockerCli文档](https://docs.docker.com/engine/reference/commandline/docker/)
