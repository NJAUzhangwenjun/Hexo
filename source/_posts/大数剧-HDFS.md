---
title: 大数剧-HDFS
date: 2020-11-28 03:13:07
tags: 大数剧-HDFS
category: 
 - 大数剧
 - HDFS
summary: 大数剧-HDFS
top: false
cover: true
author: 张文军
---
<center>更多内容请关注：https://wjhub.gitee.io</center>

![Java快速开发学习](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1586869254.png)

<center><a href="https://wjhub.gitee.io">锁清秋</a></center>

----

# 大数剧-HDFS

## 分布式文件系统

分布式文件系统在物理结构上是由计算机集群中的多个节点构成的，这些节点分为两类，一类叫“主节点”(Master Node)或者也被称为“名称结点”(NameNode)，另一类叫“从节点”（Slave Node）或者也被称为“数据节点”(DataNode)

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128032303477.png)

## HDFS简介

1). HDFS要实现以下目标：

- 兼容廉价的硬件设备
- 流数据读写
- 大数据集
- 简单的文件模型
- 强大的跨平台兼容性


2). HDFS特殊的设计，在实现上述优良特性的同时，也使得自身具有一些应用局限性，主要包括以下几个方面：

- 不适合低延迟数据访问
- 无法高效存储大量小文件
- 不支持多用户写入及任意修改文件

3). HDFS默认一个块64MB，一个文件被分成多个块，以块作为存储单位块的大小远远大于普通文件系统，可以最小化寻址开销HDFS采用抽象的块概念可以带来以下几个明显的好处：

- 支持大规模文件存储：文件以块为单位进行存储，一个大规模文件可以被分拆成若干个文件块，不同的文件块可以被分发到不同的节点上，因此，一个文件的大小不会受到单个节点的存储容量的限制，可以远远大于网络中任意节点的存储容量
- 简化系统设计：首先，大大简化了存储管理，因为文件块大小是固定的，这样就可以很容易计算出一个节点可以存储多少文件块；其次，方便了元数据的管理，元数据不需要和文件块一起存储，可以由其他系统负责管理元数据
- 适合数据备份：每个文件块都可以冗余存储到多个节点上，大大提高了系统的容错性和可用性

3). HDFS主要节点的功能

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128033551243.png)

hdfs 两大组件

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128033714099.png)

4). 元数据

- 文件是什么
- 文件被分成多少块
- 每个块和文件是怎么映射的
- 每个块被存储在哪个服务器.上面

## 名称节点（namenode）

在HDFS中，名称节点（NameNode）负责管理分布式文件系统的命名空间（Namespace），保存了两个核心的数据结构，即FsImage和EditLog

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128050843829.png)

- FsImage用于维护文件系统树以及文件树中所有的文件和文件夹的元数据

- 操作日志文件EditLog中记录了所有针对文件的创建、删除、重命名等操作

- 名称节点记录了每个文件中各个块所在的数据节点的位置信息

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128045604745.png)

### FsImage

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128051052902.png)

- FsImage文件包含文件系统中所有目录和文件inode的序列化形式。每个inode是一个文件或目录的元数据的内部表示，并包含此类信息：文件的复制等级、修改和访问时间、访问权限、块大小以及组成文件的块。对于目录，则存储修改时间、权限和配额元数据

- FsImage文件没有记录文件包含哪些块以及每个块存储在哪个数据节点。而是由名称节点把这些映射信息保留在内存中，当数据节点加入HDFS集群时，数据节点会把自己所包含的块列表告知给名称节点，此后会定期执行这种告知操作，以确保名称节点的块映射是最新的。

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128051150469.png)

### 名称节点的启动

- 在名称节点启动的时候，它会将FsImage文件中的内容加载到内存中，之后再执行EditLog文件中的各项操作，使得内存中的元数据和实际的同步，存在内存中的元数据支持客户端的读操作。

- 一旦在内存中成功建立文件系统元数据的映射，则创建一个新的FsImage文件和一个空的EditLog文件

- 名称节点起来之后，HDFS中的更新操作会重新写到EditLog文件中，因为FsImage文件一般都很大（GB级别的很常见），如果所有的更新操作都往FsImage文件中添加，这样会导致系统运行的十分缓慢，但是，如果往EditLog文件里面写就不会这样，因为EditLog 要小很多。每次执行写操作之后，且在向客户端发送成功代码之前，edits文件都需要同步更新

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128051253420.png)


### 名称节点运行期间EditLog不断变大的问题

- 在名称节点运行期间，HDFS的所有更新操作都是直接写到EditLog中，久而久之， EditLog文件将会变得很大

- 虽然这对名称节点运行时候是没有什么明显影响的，但是，当名称节点重启的时候，名称节点需要先将FsImage里面的所有内容映像到内存中，然后再一条一条地执行EditLog中的记录，当EditLog文件非常大的时候，会导致名称节点启动操作非常慢，而在这段时间内HDFS系统处于安全模式，一直无法对外提供写操作，影响了用户的使用

如何解决？答案是：SecondaryNameNode第二名称节点

### SecondaryNameNode第二名称节点

第二名称节点是HDFS架构中的一个组成部分，它是用来保存名称节点中对HDFS 元数据信息的备份，并减少名称节点重启的时间。SecondaryNameNode一般是单独运行在一台机器上

SecondaryNameNode的工作情况：

（1）SecondaryNameNode会定期和NameNode通信，请求其停止使用EditLog文件，暂时将新的写操作写到一个新的文件edit.new上来，这个操作是瞬间完成，上层写日志的函数完全感觉不到差别；
（2）SecondaryNameNode通过HTTP GET方式从NameNode上获取到FsImage和EditLog文件，并下载到本地的相应目录下；
（3）SecondaryNameNode将下载下来的FsImage载入到内存，然后一条一条地执行EditLog文件中的各项更新操作，使得内存中的FsImage保持最新；这个过程就是EditLog和FsImage文件合并；
（4）SecondaryNameNode执行完（3）操作之后，会通过post方式将新的FsImage文件发送到NameNode节点上
（5）NameNode将从SecondaryNameNode接收到的新的FsImage替换旧的FsImage文件，同时将edit.new替换EditLog文件，通过这个过程EditLog就变小了

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128052717990.png)

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128052048397.png)

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128052551313.png)

## 数据节点（DataNode）

数据节点是分布式文件系统HDFS的工作节点，负责数据的存储和读取，会根据客户端或者是名称节点的调度来进行数据的存储和检索，并且向名称节点定期发送自己所存储的块的列表；

每个数据节点中的数据会被保存在各自节点的本地Linux文件系统中；

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128051150469.png)

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128053056699.png)



## HDFS体系结构

HDFS采用了主从（Master/Slave）结构模型，一个HDFS集群包括一个名称节点（NameNode）和若干个数据节点（DataNode）。名称节点作为中心服务器，负责管理文件系统的命名空间及客户端对文件的访问。集群中的数据节点一般是一个节点运行一个数据节点进程，负责处理文件系统客户端的读/写请求，在名称节点的统一调度下进行数据块的创建、删除和复制等操作。每个数据节点的数据实际上是保存在本地Linux文件系统中的

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128053943351.png)

HDFS只设置唯一一个名称节点，这样做虽然大大简化了系统设计，但也带来了一些明显的局限性，具体如下：
（1）命名空间的限制：名称节点是保存在内存中的，因此，名称节点能够容纳的对象（文件、块）的个数会受到内存空间大小的限制。
（2）性能的瓶颈：整个分布式文件系统的吞吐量，受限于单个名称节点的吞吐量。
（3）隔离问题：由于集群中只有一个名称节点，只有一个命名空间，因此，无法对不同应用程序进行隔离。
（4）集群的可用性：一旦这个唯一的名称节点发生故障，会导致整个集群变得不可用。

## HDFS存储原理

### 冗余数据保存

作为一个分布式文件系统，为了保证系统的容错性和可用性，HDFS采用了多副本方式对数据进行冗余存储，通常一个数据块的多个副本会被分布到不同的数据节点上，如图所示，数据块1被分别存放到数据节点A和C上，数据块2被存放在数据节点A和B上。这种多副本方式具有以下几个优点：
（1）加快数据传输速度
（2）容易检查数据错误
（3）保证数据可靠性

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128054544055.png)

### 数据存取策略

（1）数据存放

- 第一个副本：放置在上传文件的数据节点；如果是集群外提交，则随机挑选一台磁盘不太满、CPU不太忙的节点
- 第二个副本：放置在与第一个副本不同的机架的节点上
- 第三个副本：与第一个副本相同机架的其他节点上
- 更多副本：随机节点

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128054902887.png)

（2）数据读取

- HDFS提供了一个API可以确定一个数据节点所属的机架ID，客户端也可以调用API获取自己所属的机架ID
- 当客户端读取数据时，从名称节点获得数据块不同副本的存放位置列表，列表中包含了副本所在的数据节点，可以调用API来确定客户端和这些数据节点所属的机架ID，当发现某个数据块副本对应的机架ID和客户端对应的机架ID相同时，就优先选择该副本读取数据，如果没有发现，就随机选择一个副本读取数据

### 数据错误与恢复

HDFS具有较高的容错性，可以兼容廉价的硬件，它把硬件出错看作一种常态，而不是异常，并设计了相应的机制检测数据错误和进行自动恢复，主要包括以下几种情形：名称节点出错、数据节点出错和数据出错。

（1）名称节点出错

名称节点保存了所有的元数据信息，其中，最核心的两大数据结构是FsImage和Editlog，如果这两个文件发生损坏，那么整个HDFS实例将失效。因此，HDFS设置了备份机制，把这些核心文件同步复制到备份服务器SecondaryNameNode上。当名称节点出错时，就可以根据备份服务器SecondaryNameNode中的FsImage和Editlog数据进行恢复。

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128055831458.png)


（2）数据节点出错

- 每个数据节点会定期向名称节点发送“心跳”信息，向名称节点报告自己的状态
- 当数据节点发生故障，或者网络发生断网时，名称节点就无法收到来自一些数据节点的心跳信息，这时，这些数据节点就会被标记为“宕机”，节点上面的所有数据都会被标记为“不可读”，名称节点不会再给它们发送任何I/O请求
- 这时，有可能出现一种情形，即由于一些数据节点的不可用，会导致一些数据块的副本数量小于冗余因子
- 名称节点会定期检查这种情况，一旦发现某个数据块的副本数量小于冗余因子，就会启动数据冗余复制，为它生成新的副本
- HDFS和其它分布式文件系统的最大区别就是可以调整冗余数据的位置

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128060019007.png)

（3）数据出错

- 网络传输和磁盘错误等因素，都会造成数据错误
- 客户端在读取到数据后，会采用md5和sha1对数据块进行校验，以确定读取到正确的数据
- 在文件被创建时，客户端就会对每一个文件块进行信息摘录，并把这些信息写入到同一个路径的隐藏文件里面
- 当客户端读取文件的时候，会先读取该信息文件，然后，利用该信息文件对每个读取的数据块进行校验，如果校验出错，客户端就会请求到另外一个数据节点读取该文件块，并且向名称节点报告这个文件块有错误，名称节点会定期检查并且重新复制这个块

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128060125173.png)

## HDFS数据读写过程

（1）读取文件

````java

import org.apache.hadoop.conf.Configuration;  
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.Path; 
public class Chapter {    
        public static void main(String[] args) { 
                try {
                        Configuration conf = new Configuration();  
                        conf.set("fs.defaultFS","hdfs://localhost:9000");
                        conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
                        FileSystem fs = FileSystem.get(conf);
                        byte[] buff = "Hello world".getBytes(); // 要写入的内容
                        String filename = "test"; //要写入的文件名
                        FSDataOutputStream os = fs.create(new Path(filename));
                        os.write(buff,0,buff.length);
                        System.out.println("Create:"+ filename);
                        os.close();
                        fs.close();
                } catch (Exception e) {  
                        e.printStackTrace();  
                }  
        }  
}

````

- FileSystem是一个通用文件系统的抽象基类，可以被分布式文件系统继承，所有可能使用Hadoop文件系统的代码，都要使用这个类
- Hadoop为FileSystem这个抽象类提供了多种具体实现
- DistributedFileSystem就是FileSystem在HDFS文件系统中的具体实现
- FileSystem的open()方法返回的是一个输入流FSDataInputStream对象，在HDFS文件系统中，具体的输入流就是DFSInputStream；FileSystem中的create()方法返回的是一个输出流FSDataOutputStream对象，在HDFS文件系统中，具体的输出流就是DFSOutputStream。

加载配置文件：` Configuration conf = new Configuration();`

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128063610236.png)

FileSystem在HDFS文件系统中的具体实现： 

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128063208559.png)


（2）读取数据的过程：

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128061527861.png)

(3 写数据的过程：

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128062218957.png)


## HDFS常用命令

首先启动hadoop

```shell
cd /usr/local/hadoop
./sbin/start-dfs.sh #启动hadoop
```

> 此处命令是以”./bin/hadoop dfs”开头的Shell命令方式，实际上有三种shell命令方式。
>
> 1. hadoop fs
> 2. hadoop dfs
> 3. hdfs dfs
>
> - hadoop fs适用于任何不同的文件系统，比如本地文件系统和HDFS文件系统
> - hadoop dfs只能适用于HDFS文件系统
> - hdfs dfs跟hadoop dfs的命令作用一样，也只能适用于HDFS文件系统


查看fs总共支持了哪些命令

```shell
./bin/hadoop fs

```
![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-HDFS/20201128065847597.png)

在终端输入如下命令，可以查看具体某个命令的作用

例如：我们查看put命令如何使用，可以输入如下命令

```bash
./bin/hadoop fs -help put
```

### 1.目录操作

需要注意的是，Hadoop系统安装好以后，第一次使用HDFS时，需要首先在HDFS中创建用户目录。本教程全部采用hadoop用户登录Linux系统，因此，需要在HDFS中为hadoop用户创建一个用户目录，命令如下：

```bash
cd /usr/local/hadoop./bin/hdfs dfs –mkdir –p /user/hadoop
```

该命令中表示在HDFS中创建一个“/user/hadoop”目录，“–mkdir”是创建目录的操作，“-p”表示如果是多级目录，则父目录和子目录一起创建，这里“/user/hadoop”就是一个多级目录，因此必须使用参数“-p”，否则会出错。
“/user/hadoop”目录就成为hadoop用户对应的用户目录，可以使用如下命令显示HDFS中与当前用户hadoop对应的用户目录下的内容：

```bash
 ./bin/hdfs dfs –ls .
```

该命令中，“-ls”表示列出HDFS某个目录下的所有内容，“.”表示HDFS中的当前用户目录，也就是“/user/hadoop”目录，因此，上面的命令和下面的命令是等价的：

```bash
./bin/hdfs dfs –ls /user/hadoop
```

如果要列出HDFS上的所有目录，可以使用如下命令：

```bash
./bin/hdfs dfs –ls
```

下面，可以使用如下命令创建一个input目录：

```bash
 ./bin/hdfs dfs –mkdir input
```

在创建个input目录时，采用了相对路径形式，实际上，这个input目录创建成功以后，它在HDFS中的完整路径是“/user/hadoop/input”。如果要在HDFS的根目录下创建一个名称为input的目录，则需要使用如下命令：

```bash
./bin/hdfs dfs –mkdir /input
```

可以使用rm命令删除一个目录，比如，可以使用如下命令删除刚才在HDFS中创建的“/input”目录（不是“/user/hadoop/input”目录）：

```bash
./bin/hdfs dfs –rm –r /input
```

上面命令中，“-r”参数表示如果删除“/input”目录及其子目录下的所有内容，如果要删除的一个目录包含了子目录，则必须使用“-r”参数，否则会执行失败。

### 2.文件操作

在实际应用中，经常需要从本地文件系统向HDFS中上传文件，或者把HDFS中的文件下载到本地文件系统中。
首先，使用vim编辑器，在本地Linux文件系统的“/home/hadoop/”目录下创建一个文件myLocalFile.txt，里面可以随意输入一些单词，比如，输入如下三行：

```shell
Hadoop
Spark
XMU DBLAB
```

然后，可以使用如下命令把本地文件系统的“/home/hadoop/myLocalFile.txt”上传到HDFS中的当前用户目录的input目录下，也就是上传到HDFS的“/user/hadoop/input/”目录下：

```bash
./bin/hdfs dfs -put /home/hadoop/myLocalFile.txt  input
```

可以使用ls命令查看一下文件是否成功上传到HDFS中，具体如下：

```bash
./bin/hdfs dfs –ls input
```

该命令执行后会显示类似如下的信息：

```shell
Found 1 items   
-rw-r--r--   1 hadoop supergroup         36 2017-01-02 23:55 input/ myLocalFile.txt
```

下面使用如下命令查看HDFS中的myLocalFile.txt这个文件的内容：

```bash
./bin/hdfs dfs –cat input/myLocalFile.txt
```

下面把HDFS中的myLocalFile.txt文件下载到本地文件系统中的“/home/hadoop/下载/”这个目录下，命令如下：

```bash
./bin/hdfs dfs -get input/myLocalFile.txt  /home/hadoop/下载
```

可以使用如下命令，到本地文件系统查看下载下来的文件myLocalFile.txt：

```bash
cd ~
cd 下载
ls
cat myLocalFile.txt
```

最后，了解一下如何把文件从HDFS中的一个目录拷贝到HDFS中的另外一个目录。比如，如果要把HDFS的“/user/hadoop/input/myLocalFile.txt”文件，拷贝到HDFS的另外一个目录“/input”中（注意，这个input目录位于HDFS根目录下），可以使用如下命令：

```bash
./bin/hdfs dfs -cp input/myLocalFile.txt  /input
```

## 利用Web界面管理HDFS

