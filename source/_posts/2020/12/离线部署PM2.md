---
title: 离线部署PM2
comments: true
date: 2020-12-03 15:12:46
updated: 2020-12-03 15:12:46
tags:
    - Linux
    - PM2
    - Node
---

<blockquote class="blockquote-center">离线安装PM2管理项目</blockquote>

<!--more-->

# 一丶安装Node

1. 先到官网下载二进制包地址：[https://nodejs.org/en/download/](https://nodejs.org/en/download/)

![Node下载地址](image-20201203173545961.png)
2. 上传到服务器，解压(我上传到/usr/local/下，这个可以自行选择)


```bash
tar -Jxvf node-v14.15.1-linux-x64.tar.xz
# 重命名
mv node-v14.15.1-linux-x64 nodejs
```

3. 建立软连接

```bash
ln -s /usr/local/nodejs/bin/npm /usr/local/bin/
ln -s /usr/local/nodejs/bin/node /usr/local/bin/
```

4. 测试是否安装成功

```bash
node -v
```

# 二、安装PM2

1. 先到github下载：[https://github.com/Unitech/pm2/tags](https://github.com/Unitech/pm2/tags)

![PM2下载地址](image-20201203174942356.png)

2. 上传到服务器，解压(我上传到/usr/local/下，这个可以自行选择)

```bash
tar -zxvf pm2-4.5.0.tar.gz
# 改名
mv pm2-4.5.0 pm2
```

3. 添加软连接

```
ln -s /usr/local/pm2/bin/pm2 /usr/local/bin/
```

4. 测试安装

```bash
pm2 ls
```

# 三、整合安装

上面的方法繁琐，还有可能出现问题。所以我们可以制作一个包含pm2的nodejs包。步骤如下：

- 在一台可以联网的电脑上使用第一步安装nodejs
- 执行`npm install pm2 -g`
- 将整个nodejs文件夹打包`tar -zcvf nodejs.tar.gz nodejs`

提供我打包好的安装包（node版本为14.15.1）

> https://wws.lanzous.com/ixEZ1j0jz1e
> 密码:2hxq

当我们拿到安装包后执行下面的操作

1. 解压

```bash
tar -zxvf node-v14.15.1.tar.gz
mv node-v14.15.1 nodejs
```

2. 建立软连接

```bash
ln -s /usr/local/nodejs/bin/npm /usr/local/bin/
ln -s /usr/local/nodejs/bin/node /usr/local/bin/
ln -s /usr/local/nodejs/lib/node_modules/pm2/bin/pm2 /usr/local/bin/
```

3. 测试安装

```bash
node -v
pm2 ls
```

分别输出下面两个内容代表安装成功

![node成功输出](image-20201203180642393.png)

![pm2成功输出](image-20201203180702553.png)

# 四、常用命令

| 命令                                       | 说明                   |
| :----------------------------------------- | :--------------------- |
| pm2 startup                                | 设置pm2开机自启动      |
| pm2 unstartup                              | 移除pm2开机自启动      |
| pm2 save                                   | 保存当前进程开机自启动 |
| pm2 start <进程启动命令> [--name <进程名>] | 启动应用程序           |
| pm2 list                                   | 显示所有进程状态       |
| pm2 monit                                  | 监控进程               |
| pm2 logs [进程id或名字]                    | 显示进程日志           |
| pm2 stop [all]                             | 停止[所有]进程         |
| pm2 restart [all]                          | 重启[所有]进程         |
| pm2 delete [<进程名或者id>,all]            | 删除指定[所有]进程     |
| pm2 info [进程id或名字]                    | 查看应用程序信息       |

> 配置文件

```json
{
    "apps": [{
        "name": "BaoWu",
        "script": "/usr/bin/java",
        "watch": true,
        "ignore-watch":[
            "wwwroot"
        ],
        "args": [
            "-jar",
            "/home/visible/Project/BaoWu/baowu-zwt.jar"
        ],
        "exec_interpreter": "",
        "out_file": "/dev/null",
        "error_file": "/dev/null",
        "exec_mode": "fork"
    },{
        "name": "CloudWatchUniversal",
        "script": "/usr/bin/java",
        "watch": true,
        "ignore-watch":[
            "wwwroot"
        ],
        "args": [
            "-jar",
            "/home/visible/Project/universalCloudWatch/cloudwatch-universal-1.0-SNAPSHOT.jar"
        ],
        "exec_interpreter": "",
        "exec_mode": "fork"
    }]
}
```

