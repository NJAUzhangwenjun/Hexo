---
title: SSH、 yum、 Shell、Maven、Git
date: 2020-02-03 07:06:36
tags: Linux之 SSH, yum, Shell,Maven,Git
category: 工具
summary: Linux之 SSH、 yum、 Shell、Maven、Git
top: true
cover: true
author: 张文军
---

![](/images/favicon.png)

## 一、SSH

### 1、SSH工作机制

ssh为Secure Shell（安全外壳协议）的缩写。
很多ftp、pop和telnet在本质上都是不安全的。
我们使用的Xshell6就是基于SSH的客户端实现。
SSH的服务端实现为openssh- deamon。
在linux上使用ssh

```shell
ssh root@192.168.33.88
```

### 2、SSH免密码登录

生成秘钥

```shell
ssh-keygen
```

把自己的公钥拷给对方

```shell
ssh-copy-id 192.168.33.3
```

基于ssh的文件拷贝：

```shell
scp abc.txt 192.168.33.3:/root
```

基于ssh的目录拷贝：需要加-r递归拷贝

```shell
scp aaa -r 192.168.33.3:/root
```

远程执行命令：

```shell
ssh 192.168.33.3 "echo hello > /root/hello.txt" 
```

## 二、yum工具

yum类似于Maven工具，可以从中央仓库下载安装各种软件。
当某个软件有依赖其它软件时，yum也会自动下载并安装其它软件。
常用命令：

- 安装软件包：`yum install xxx -y `（-y 表示免确认）

- 清除本地索引数据：`yum clear all`

- 查找库中软件包：`yum list | grep xxx`

- 列出本地所配置的仓库信息：`yum repolist`

- 其它参数：直接敲yum回车
  如：搜索jdk工具

  ```shell
  yum list | grep jdk
  ```

  [使用 yum 安装 JDK 和 Tomcat](<https://www.cnblogs.com/yoyoketang/p/10186513.html>)

## 三、Shell编程

### 1、基本格式

Shell俗称壳（用来区别于核），是指“为使用者提供操作界面”的软件（命令解析器）。
Shell是用户与内核进行交互操作的一种接口，目前最流行的Shell称为 bash Shell。
Shell也是一门编程语言（解释型的编程语言），即shell脚本（就是在用linux的shell命令编程）。
一个系统可以存在多个shell，可以通过`cat /etc/shells`命令查看系统中安装的shell，不同的shell可能支持的命令语法是不
同的。

```shell
[root@localhost /]# cat /etc/shells
```

代码写在普通文本文件中，通常以.sh为后缀名

```shell
vi hello.sh
#!/bin/bash ##表示用哪一种shell解析器来解析执行这个脚本
echo "hello word" ##注释也可以写在这里
##这是一行注释
```

执行脚本：

```shell
sh hello.sh
```

或者给脚本添加x权限，直接执行

```shell
./hello.sh
```

### 2、变量

变量=值（例如A=5）
注意：等号两侧不能有空格
变量名一般习惯为大写
使用变量：$A
定义变量
A=1
查看变量

```shell
echo $A
```

查看当前进程中所有变量

```shell
set
```

撤销变量

```shell
unset A
```

声明静态变量,不能unset

```shell
readonly B=2
```

注意：变量中的值没有类型，全部为字符串。

## 四、Maven

### 1、Maven的安装与配置

1、确认jdk是否安装
http://maven.apache.org/docs/history.html
Maven3.3.X及以上版本至少需要jdk1.7的支持。
2、去官网http://maven.apache.org/download.cgi
3、下载 apache-maven-3.5.3-bin.zip
4、解压
目录结构：
-bin：命令
-boot：maven启动所需
-conf：配置文件（常用的setting.xml就在其中）
-lib：maven所依赖的jar包
5、环境变量
环境变量配置和JDK配置一样，如：M2_HOME/MAVEN_HOME和Path 

### 2、Maven规定了一套默认的项目格式 

-src/main/java —— 存放项目的.java文件

-src/main/resources —— 存放项目资源文件，如spring、struts2配置文件，db.properties

-src/main/webapp —— 存放jsp，css，image等文件src/test/java —— 存放所有测试.java文件，如JUnit测试类

-src/test/resources —— 测试资源文件

-pom.xml——主要要写的maven配置文件

-target —— 项目由maven自动输出位置

  即：

  --src
  -----main
  ----------java
  ----------resources
  -----test
  ----------java
  ----------resources
  --target
  --pom.xml

### 3、常用命令

- mvn -v：查看版本
- mvn compile：编译
- mvn test：测试
- mvn package：打包
- mvn install：打包并拷贝到本地仓库 

### 4、Maven相关概念介绍

#### 1、项目对象模型（POM）

#### 2、坐标

什么是坐标
在平面几何中坐标（x，y）可以标识平面中唯一的一点
Maven坐标主要组成
    groupId：定义当前Maven项目隶属项目
    artifactId：定义实际项目中的一个模块
    version：定义当前项目的当前版本
    packaging：定义该项目的打包方式
Maven为什么使用坐标
Maven世界拥有大量构建，我们需要找一个用来标识

#### 3、依赖管理

​1、依赖声明

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

2、依赖范围
依赖范围scope用来控制依赖和编译、测试、运行的classpath的关系:

- compile：默认编译依赖范围。对于编译、测试、运行三种classpath都有效。
- test：测试依赖范围。只对于测试classpath有效。
- provided：已提供依赖访问。对于编译、测试的classpath有效，但对于运行无效，因为容器已经提供，例如servlet-api
- runtime：运行时提供。例如jdbc驱动 

#### 4、仓库管理

何为Maven仓库
--用来统一存储所有Maven共享构建的位置就是仓库
Maven仓库布局
  根据Maven坐标定义每个构建在仓库中唯一存储路径
  大致为：groupId/artifactId/version/artifactId-version.packaging
仓库的分类

- 本地仓库
   ~/.m2/repository/
   每个用户只有一个本地仓库
- 远程仓库
  - 中央仓库：Maven默认的远程仓库http://repo1.maven.org/maven2/
  - 私服：是一种特殊的远程仓库，它是架设在局域网内的仓库
  - 镜像：用来替代中央仓库，速度一般比中央仓库 

#### 5、生命周期

何为生命周期

- Maven生命周期就是为了对所有的构建过程进行抽象和统一包括项目清理、初始化、编译、打包、测试、部署等几乎所有构建步骤

Maven三大生命周期

- clean：清理项目的
- default：构建项目的
- site：生成项目站点的

#### 6、仓库

1、本地仓库
设置本地仓库：conf/setting.xml

## 五、git

#### 1、git初始化

1、安装-----Google it

- 为每一台电脑配置身份信息

```shell
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

#### 2、**git基本命令**

```shell
# 把某个目录变成可以让git管理到的目录（创建版本库）
git init

#把内容输入到一个文件
echo "xxxx" > xxxx 

#把工作空间的某个文件添加到cache
git add  xxxx

#把工作空间的所有内容添加到cache
git add -A

#把cache当中的某个文件提交到本地库
git commit -m "xxxx"

#all
git commit -am "xxxx"

#cache ---->work file恢复一个文件  file1 file2 恢复两个文件  .恢复所有文件
git checkout readme.txt 

#git状态查询
git status

#查看不同的地方 默认查看工作区和cache
#git diff --cached   比较cache和Repository
#git diff HEAD 工作区和最新的Resository
#git diff commit-id 工作区和制定的repository
#git diff --cached commit-id
#git diff --commit-id commit-id
git diff 

#reset 顾名思义   （HEAD~100）
git reset HEAD^

#git的日志
git log git log --pretty=oneline

#oh my pretty pretty boy i love you 
git reflog  查看历史命令

#git rm --cached file_path
git rm 
git mv
#远程仓库的克隆岛本地库
git clone
#关联远程仓库
git remote add
#推送到远程仓库
git push
#拉取远程仓库的内容
git pull
#查看当前分支 -a查看所有分支 -av 查看所有分支的信息 -avv 查看所有分支的信息和关系
git branch
#创建一个分支 基于当前分支创建分支
git branch  xxx
#基于oldType创建分支
git branch newBranch oldType
#切换分支
git checkout 分支的名字
#删除分支
git branch -d   xxx
#查看文件内容
git cat-file -p  commitid
#查看对象类型 blob commit tree
git cat-file -t commitid

```

#### 3、github

1. $ ssh-keygen -t rsa -C "email"    //public key for push

1.  git remote add **nickName** **gitUrl**    **// conn remote**

1. git push -u **remoteBranch** **localBranch**

###### 分支

查看分支

创建分支

    基于当前分支创积分分支    
    
    基于远程分支创建分支
    
    基于新分支创建分支
    
    基于提交创建分支其
    实都是基于**commit**创建分支

合并分支

一定要切换到被合并的分支上去合并

比如说A要合并A1

那么先要切换到A1，然后在A1上面执行merge

#### 4、**远程仓库**



基于文件共享（nfs）

linux  ssh

装好git yum install git

git地址：username@ip:/dir

密码



gitlab

<https://about.gitlab.com/installation/#centos-7>

传上去

gitlab-ce-10.8.2-ce.0.el7.x86_64.rpm

安装 rpm -ivh /home/tools/gitlab-ce-10.8.2-ce.0.el7.x86_64.rpm



sudo gitlab-ctl reconfigure   时间很少8-10分钟



 gitlab-ctl start 启动gitlab



访问gitlab  [http://ip:port](http://ip:port/)

设置密码

use

github

## github骚操作

### stars或fork数量关键词去查找

- 公式

	- xxx关键词 star通配符

		- ：>或者：>=

	- 区间范围数字

		- 数字1..数字2

- 查找stars数大于等于5000的springboot项目

	- springboot stars:>=5000

- 查找forks数大于500的springcloud项目

	- springcloud forks:>500

- 组合使用

	- 查找fork在100到200之间并且stars数在80到100之间的springboot项目

		- springboot forks:100..200 stars:80..100

### awesome加强搜索

- 公式

	- awesome关键字

		- awesome系列一般是用来学习、工具、书籍类相关的项目

- 搜索优秀的redis相关的项目，包括框架、教程等

### 高亮显示某一行代码

- 公式

	- 1行

		- 地址后面紧跟#L数字

	- 多行

		- 地址后面紧跟#L数字-L数字2

### 项目内搜索

- 英文t

### 搜索某个地区内的大佬

- 公式

	- location：地区
	- language：语言

- 地区北京的Java方向的用户

	- location：beijing language：java


  