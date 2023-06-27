---
title: Hyperledger fabric环境搭建
date: 2021-03-01 10:34:00 +0800
categories: [区块链]
tags: [Hyperledger Fabric]
pin: true
author: 小李学编程

toc: true
comments: true
typora-root-url: ../../libits.github.io
math: false
mermaid: true


---

### 1 安装docker

1. 添加阿里的docker镜像仓库

   ```shell
   # 添加阿里的docker镜像仓库
   $ curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
   $ sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
   # 更新软件源
   $ sudo apt-get update
   ```

2. 安装docker

   ```shell
   # 安装docker
   $ sudo apt-get install docker-ce -y
   # 查看安装的docker版本
   ```

3. 将当前用户添加到docker组

   ```shell
   # 将用户加入该 group 内。然后退出并重新登录就生效啦。
   $ sudo gpasswd -a ${USER} docker
   # 重启docker服务
   $ systemctl restart docker
   # 当前用户切换到docker群组
   $ newgrp - docker
   ```

4. 安装docker-compose

   ```shell
   #安装依赖工具
   $ sudo apt-get install python-pip -y
   #安装编排工具
   $ sudo pip install docker-compose
   #查看版本
   $ sudo docker-compose version
   ```

### 2 安装go

```http
# 安装包下载地址:
https://golang.org/dl/  - 翻墙
https://studygolang.com/dl - 国内镜像源
```

如果没有进行安装包下载, 课直接使用如下命令(目前最新版本):

```shell
# 1. 使用wget工具下载安装包
$ wget https://dl.google.com/go/go1.11.linux-amd64.tar.gz
# 2. 解压tar包到/usr/local
$ sudo tar zxvf go1.11.linux-amd64.tar.gz -C /usr/local
# 3. 创建Go目录
$ mkdir $HOME/go
# 4. 用vi打开~./bashrc，配置环境变量
$ vim ~/.bashrc
# 5. 增加下面的环境变量，保存退出
	export GOROOT=/usr/local/go
    export GOPATH=$HOME/go
    export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
# 6. 使环境变量立即生效, 一些命令二选一
$ source ~/.bashrc  
$ . ~/.bashrc
# 7. 检测go是否安装好
$ go version
```

### 3 安装Node.js

1. 官方下载地址

   ```http
   https://nodejs.org/en/download/
   ```

2. 将node.js设置为全局可用

   ```shell
   # 打开系统级别的配置文件 /etc/profile
   $ sudo vim /etc/profile
   # 添加如下配置项, 保存退出
       export NODEJS_HOME=/opt/node-v8.11.4-linux-x64
       export PATH=$PATH:$NODEJS_HOME/bin
   # 重新加载配置文件
   $ . /etc/profile
   /etc/profile -> 设置环境变量的配置文件
   	- 对当前系统下所有用户生效
   ```

 





