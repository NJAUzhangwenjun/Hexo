---
title: docker
date: 2020-11-17 22:08:18
tags: docker
category: docker
summary: docker入门
top: false
cover: true
author: 张文军
---
<center>更多内容请关注：</center>

![Java快速开发学习](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1586869254.png)

<center><a href="https://wjhub.gitee.io">锁清秋</a></center>

----


## 入门

### 官网

#### docker官网：http://www.docker.com

#### docker中文网站：https://www.docker-cn.com/

### 仓库

#### Docker Hub官网: https://hub.docker.com/

# Docker安装

## 前提说明

## Docker的基本组成

### 镜像（image）

### 容器（container）

### 仓库（repository）

### 小总结

## 安装步骤

### CentOS6.8安装Docker

> yum install -y epel-release

> yum install -y docker-io

> 安装后的配置文件：/etc/sysconfig/docker

> 启动Docker后台服务：service docker start

> docker version验证

### CentOS7安装Docker

#### https://docs.docker.com/install/linux/docker-ce/centos/

## 安装步骤

### 官网中文安装参考手册

>  https://docs.docker-cn.com/engine/installation/linux/docker-ce/centos/#prerequisites

### 确定你是CentOS7及以上版本

>  cat /etc/redhat-release

> yum安装gcc相关

###  CentOS7能上外网

>  

>  yum -y install gcc

>  yum -y install gcc-c++

### 卸载旧版本

>  yum -y remove docker docker-common docker-selinux docker-engine

>  2018.8英文官网版本

### 安装需要的软件包

>  yum install -y yum-utils device-mapper-persistent-data lvm2

### 设置stable镜像仓库

###  大坑

>  yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

### 推荐

>  yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

### 更新yum软件包索引

>  yum makecache fast

### 安装DOCKER CE

>  yum -y install docker-ce

### 启动docker

>  systemctl start docker

### 测试

>  docker version

>  docker run hello-world

### 卸载

>  systemctl stop docker 

>  yum -y remove docker-ce

>  rm -rf /var/lib/docker

## 永远的HelloWorld


### 阿里云镜像加速

#### 是什么

> https://dev.aliyun.com/search.html

#### 注册一个属于自己的阿里云账户(可复用淘宝账号)

#### 获得加速器地址连接

> 登陆阿里云开发者平台

> 获取加速器地址

#### 配置本机Docker运行镜像加速器

> 配置镜像加速CentOS6.8版本

>  重新启动Docker后台服务：service docker restart

>  Linux 系统下配置完加速器需要检查是否生效

> 配置镜像加速CentOS7版本

>  mkdir -p /etc/docker

>  vim  /etc/docker/daemon.json

>  systemctl daemon-reload

>  systemctl restart docker

### 网易云加速

#### 基本同上述阿里云

### 启动Docker后台容器(测试运行 hello-world)

> docker run hello-world

#### run干了什么

### 底层原理

#### Docker是怎么工作的

#### 为什么Docker比较比VM快

# Docker常用命令

## 帮助命令

> docker version

> docker info

> docker help

## 镜像命令

> docker images

#### 列出本地主机上的镜像

#### OPTIONS说明：

> -a :列出本地所有的镜像（含中间映像层）

> -q :只显示镜像ID。

> --digests :显示镜像的摘要信息

> --no-trunc :显示完整的镜像信息

### docker search 某个XXX镜像名字

#### 网站

> https://hub.docker.com

#### 命令

> docker search [OPTIONS] 镜像名字

> OPTIONS说明：

>  --no-trunc : 显示完整的镜像描述

>  -s : 列出收藏数不小于指定值的镜像。

>  --automated : 只列出 automated build类型的镜像；


> docker pull 某个XXX镜像名字

#### 下载镜像

> docker pull 镜像名字[:TAG]

> docker rmi 某个XXX镜像名字ID

#### 删除镜像

#### 删除单个

> docker rmi  -f 镜像ID

#### 删除多个

> docker rmi -f 镜像名1:TAG 镜像名2:TAG 

#### 删除全部

> docker rmi -f $(docker images -qa)

## 容器命令

### 有镜像才能创建容器，这是根本前提(下载一个CentOS镜像演示)

> docker pull centos

### 新建并启动容器

> docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

>  OPTIONS说明

> 启动交互式容器

### 列出当前所有正在运行的容器

#### docker ps [OPTIONS]

>  OPTIONS说明

### 退出容器

#### 两种退出方式

> exit

>  容器停止退出

> ctrl+P+Q

>  容器不停止退出

### 启动容器

> docker start 容器ID或者容器名

### 重启容器

> docker restart 容器ID或者容器名

### 停止容器

> docker stop 容器ID或者容器名

### 强制停止容器

> docker kill 容器ID或者容器名

### 删除已停止的容器

#### docker rm 容器ID

> 一次性删除多个容器

>  docker rm -f $(docker ps -a -q)

>  docker ps -a -q | xargs docker rm

### 重要

#### 启动守护式容器

> docker run -d 容器名

#### 从Hub上下载tomcat镜像到本地并成功运行

>  docker run -it -p 8080:8080 tomcat

>  -p 主机端口:docker容器端口

>  -P 随机分配端口

>  i:交互

>  t:终端

#### 查看容器日志

> docker logs -f -t --tail 容器ID

>  *   -t 是加入时间戳

>  *   -f 跟随最新的日志打印

>  *   --tail 数字 显示最后多少条

#### 查看容器内运行的进程

> docker top 容器ID

#### 进入正在运行的容器并以命令行交互

> docker exec -it 容器ID bashShell

#### 重新进入

> docker attach 容器ID

> 上述两个区别

>  attach 直接进入容器启动命令的终端，不会启动新的进程

>  exec 是在容器中打开新的终端，并且可以启动新的进程

## 小总结

### 常用命令
