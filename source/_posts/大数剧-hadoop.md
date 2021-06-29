---
title: 大数剧-hadoop-安装
date: 2020-11-26 01:31:50
tags: 大数剧-hadoop-安装
category: 大数剧
summary: 大数剧-hadoop-安装
top: false
cover: true
author: 张文军
---
<center>更多内容请关注：https://wjhub.gitee.io</center>

![Java快速开发学习](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1586869254.png)

<center><a href="https://wjhub.gitee.io">锁清秋</a></center>

----

# 大数剧-hadoop-安装

> 使用ambari环境搭建 可参考网络优秀博客：https://www.cnblogs.com/langfanyun/p/10368688.html
> 下面我的是手动搭建

## hadoop应用现状

![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%861/20201126025852554.png)

## Hadoop安装方式

Hadoop的安装方式有三种，分别是单机模式，伪分布式模式，分布式模式。

- 单机模式：单机模式：Hadoop 默认模式为非分布式模式（本地模式），无需进行其他配置即可运行。非分布式即单 Java 进程，方便进行调试。
- 伪分布式模式：Hadoop 可以在单节点上以伪分布式的方式运行，Hadoop 进程以分离的 Java 进程来运行，节点既作为 NameNode 也作为 DataNode，同时，读取的是 HDFS 中的文件。
- 分布式模式：使用多个节点构成集群环境来运行Hadoop。

## 创建hadoop用户

```bash
su root # 以 root 用户登录
useradd -m hadoop -s /bin/bash # 创建新用户hadoop
passwd hadoop #修改密码
vim sudo #hadoop 用户增加管理员权限，方便部署，避免一些对新手来说比较棘手的权限问题
```

增加

```bash
hadoop ALL=(ALL) ALL
```





## 更新yum

```bash
sudo yum update
```



## 安装SSH、配置SSH无密码登陆

集群、单节点模式都需要用到 SSH 登陆（类似于远程登陆，可以登录某台 Linux 主机，并且在上面运行命令），默认已安装了 SSH client，此外还需要安装 SSH server：

```bash
sudo yum install openssh-server
```

  

安装后，可以使用如下命令登陆本机：

```bash
ssh localhost
```

但这样登陆是需要每次输入密码的， 需要配置成SSH无密码登陆比较方便。

```bash
exit                           # 退出刚才的 ssh localhost
cd ~/.ssh/                     # 若没有该目录，请先执行一次ssh localhost
ssh-keygen -t rsa              # 会有提示，都按回车就可以
cat id_rsa.pub >> authorized_keys  # 加入授权
chmod 600 ./authorized_keys    # 修改文件权限
```

此时再用 ssh localhost
 命令，无需输入密码就可以直接登录了

这只是本机访问，不同电脑之间登录如下：

```bash
    sudo vim  /etc/ssh/sshd_config
```




大概在98行 把#去掉，ESC:wq,再执行下面命令

```undefined
sudo service sshd restart
```

然后把各自虚拟机上生成的id_rsa.pub拷贝到其他电脑上
 \#192.168.92.101
 hadoop@192.168.92.101:scp ~/.ssh/id_[rsa.pub](https://link.jianshu.com?t=http://rsa.pub) hadoop@192.168.92.102:~/.ssh/[master.pub](https://link.jianshu.com?t=http://master.pub)
 hadoop@192.168.92.101:scp ~/.ssh/id_[rsa.pub](https://link.jianshu.com?t=http://rsa.pub) hadoop@192.168.92.103:~/.ssh/[master.pub](https://link.jianshu.com?t=http://master.pub)

```ruby
#192.168.92.102
hadoop@192.168.92.102:scp ~/.ssh/id_rsa.pub hadoop@192.168.92.101:~/.ssh/slave02.pub 
hadoop@192.168.92.102:scp ~/.ssh/id_rsa.pub hadoop@192.168.92.103:~/.ssh/slave02.pub 

#192.168.92.103
hadoop@192.168.92.103:scp ~/.ssh/id_rsa.pub hadoop@192.168.92.101:~/.ssh/slave03.pub 
hadoop@192.168.92.103:scp ~/.ssh/id_rsa.pub hadoop@192.168.92.102:~/.ssh/slave03.pub
```




```shell

cat slave2.pub >> authorized_keys
cat slave3.pub >> authorized_keys
```

重复以上步骤（注意自己操作的主机，文件名）
 修改主机名:

```cpp
sudo hostnamectl --static set-hostname master
sudo hostnamectl --static set-hostname slave2
sudo hostnamectl --static set-hostname slave3
```

重新连接就可以看到新的主机名了

## 测试

```undefined
ssh master
ssh slave02
ssh slave03
```

此时会有如下提示(SSH首次登陆提示)，输入 yes 。



![](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-hadoop/20201127022337532.png)



## 安装Java环境

Hadoop3.1.3需要JDK版本在1.8及以上。需要按照下面步骤来自己手动安装JDK1.8。
 已经把JDK1.8的安装包jdk-8u162-linux-x64.tar.gz放在了百度云盘，[可以点击这里到百度云盘下载JDK1.8安装包](https://pan.baidu.com/s/1gbmPBXrJDCxwqPGkfvX5Xg)（提取码：lnwl）。请把压缩格式的文件jdk-8u162-linux-x64.tar.gz下载到本地电脑

在Linux命令行界面中，执行如下Shell命令（注意：当前登录用户名是hadoop）：

```bash
cd /usr/libsudo mkdir jvm #创建/usr/lib/jvm目录用来存放JDK文件
cd ~ #进入hadoop用户的主目录cd Downloads  #注意区分大小写字母，刚才已经通过FTP软件把JDK安装包jdk-8u162-linux-x64.tar.gz上传到该目录下
sudo tar -zxvf ./jdk-8u162-linux-x64.tar.gz -C /usr/lib/jvm  #把JDK文件解压到/usr/lib/jvm目录下
rm -rf ./jdk-8u162-linux-x64.tar.gz  # 删除
```

  

上面使用了解压缩命令tar，如果对Linux命令不熟悉，可以参考[常用的Linux命令用法](http://dblab.xmu.edu.cn/blog/1624-2/)。

JDK文件解压缩以后，可以执行如下命令到/usr/lib/jvm目录查看一下：

```bash
cd /usr/lib/jvm
ls
```

  

可以看到，在/usr/lib/jvm目录下有个jdk1.8.0_162目录。
下面继续执行如下命令，设置环境变量：

```bash
vim /etc/profile
```

  

上面命令使用vim编辑器（[查看vim编辑器使用方法](http://dblab.xmu.edu.cn/blog/1607-2/)）打开了hadoop这个用户的环境变量配置文件，请在这个文件的开头位置，添加如下几行内容：

```shell
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_162
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

让配置生效

```bash
source /etc/profile
```

  

这时，可以使用如下命令查看是否安装成功：

```bash
java -version
```

  

如果能够在屏幕上返回如下信息，则说明安装成功：

```shell
hadoop@ubuntu:~$ java -version
java version "1.8.0_162"
Java(TM) SE Runtime Environment (build 1.8.0_162-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.162-b12, mixed mode)
```

至此，就成功安装了Java环境。下面就可以进入Hadoop的安装。

## 安装 Hadoop3.1.3

Hadoop安装文件，可以到[Hadoop官网](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.1.3/hadoop-3.1.3.tar.gz)下载hadoop-3.1.3.tar.gz。
也可以直接[点击这里从百度云盘下载软件](https://pan.baidu.com/s/1gbmPBXrJDCxwqPGkfvX5Xg)（提取码：lnwl），进入百度网盘后，进入“软件”目录，找到hadoop-3.1.3.tar.gz文件，下载到本地。
 选择将 Hadoop 安装至 /usr/local/ 中：

```bash
sudo tar -zxf ~/下载/hadoop-3.1.3.tar.gz -C /usr/local    # 解压到/usr/local中cd /usr/local/sudo mv ./hadoop-3.1.3/ ./hadoop            # 将文件夹名改为hadoopsudo chown -R hadoop ./hadoop       # 修改文件权限
```

  

Hadoop 解压后即可使用。输入如下命令来检查 Hadoop 是否可用，成功则会显示 Hadoop 版本信息：

```bash
cd /usr/local/hadoop./bin/hadoop version
```

  

相对路径与绝对路径

请务必注意命令中的相对路径与绝对路径，本文后续出现的 `./bin/...`，`./etc/...` 等包含 ./ 的路径，均为相对路径，以 /usr/local/hadoop 为当前目录。例如在 /usr/local/hadoop 目录中执行 `./bin/hadoop version` 等同于执行 `/usr/local/hadoop/bin/hadoop version`。可以将相对路径改成绝对路径来执行，但如果你是在主文件夹 ~ 中执行 `./bin/hadoop version`，执行的会是 `/home/hadoop/bin/hadoop version`，就不是 所想要的了。

## Hadoop单机配置(非分布式)

Hadoop 默认模式为非分布式模式（本地模式），无需进行其他配置即可运行。非分布式即单 Java 进程，方便进行调试。

现在 可以执行例子来感受下 Hadoop 的运行。Hadoop 附带了丰富的例子（运行 `./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar` 可以看到所有例子），包括 wordcount、terasort、join、grep 等。

在此 选择运行 grep 例子， 将 input 文件夹中的所有文件作为输入，筛选当中符合正则表达式 dfs[a-z.]+ 的单词并统计出现的次数，最后输出结果到 output 文件夹中。

```bash
cd /usr/local/hadoopmkdir ./inputcp ./etc/hadoop/*.xml ./input   # 将配置文件作为输入文件
./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar 
grep ./input ./output 'dfs[a-z.]+'cat ./output/*          # 查看运行结果
```

  

执行成功后如下所示，输出了作业的相关信息，输出的结果是符合正则的单词 dfsadmin 出现了1次

**注意**，Hadoop 默认不会覆盖结果文件，因此再次运行上面实例会提示出错，需要先将 `./output` 删除。

```bash
rm -r ./output
```

  

## Hadoop伪分布式配置

Hadoop 可以在单节点上以伪分布式的方式运行，Hadoop 进程以分离的 Java 进程来运行，节点既作为 NameNode 也作为 DataNode，同时，读取的是 HDFS 中的文件。

Hadoop 的配置文件位于 /usr/local/hadoop/etc/hadoop/ 中，伪分布式需要修改2个配置文件 **core-site.xml** 和 **hdfs-site.xml** 。Hadoop的配置文件是 xml 格式，每个配置以声明 property 的 name 和 value 的方式来实现。

修改配置文件 **core-site.xml** (通过 gedit 编辑会比较方便: `gedit ./etc/hadoop/core-site.xml`)，将当中的

```xml
<configuration>
</configuration>
```

  

修改为下面配置：

```xml
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/usr/local/hadoop/tmp</value>
        <description>Abase for other temporary directories.</description>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

  

同样的，修改配置文件 **hdfs-site.xml**：

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/hadoop/tmp/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/hadoop/tmp/dfs/data</value>
    </property>
    
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>dfs.http.address</name>
        <value>0.0.0.0:50070</value>
    </property>
</configuration>
```

> 说明：
>
> 1. 开启web服务
>
> ```xml
>     <property>
>         <name>dfs.webhdfs.enabled</name>
>         <value>true</value>
>     </property>
>     <property>
>         <name>dfs.http.address</name>
>         <value>0.0.0.0:50070</value>
>     </property>
> ```
>
> 2. 端口号（关闭防火墙）
>
> web访问hadoop默认HTTP端口总结
>
> 一、HDFS
>
>    1.NameNode 默认端口   50070
>
>    2.SecondNameNode 默认端口   50090
>
>    3.TaskNode 默认端口   50075
>
> 在hadoop2的HDFS中fs.defaultFS在core-site.xml 中配置，默认端口是8020，但是由于其接收Client连接的RPC端口，所以如果在hdfs-site.xml中配置了RPC端口9000，所以fs.defaultFS端口变为9000
>
> 二、MapReduce
>
>    1.JobTracker 默认端口   50030
>
>    2.TaskTracker 默认端口   50060

Hadoop配置文件说明

Hadoop 的运行方式是由配置文件决定的（运行 Hadoop 时会读取配置文件），因此如果需要从伪分布式模式切换回非分布式模式，需要删除 core-site.xml 中的配置项。

此外，伪分布式虽然只需要配置 fs.defaultFS 和 dfs.replication 就可以运行（官方教程如此），不过若没有配置 hadoop.tmp.dir 参数，则默认使用的临时目录为 /tmp/hadoo-hadoop，而这个目录在重启时有可能被系统清理掉，导致必须重新执行 format 才行。所以  进行了设置，同时也指定 dfs.namenode.name.dir 和 dfs.datanode.data.dir，否则在接下来的步骤中可能会出错。

配置完成后，执行 NameNode 的格式化:

```bash
cd /usr/local/hadoop./bin/hdfs namenode -format
```

  

成功的话，会看到 “successfully formatted” 的提示，具体返回信息类似如下：

```shell
2020-01-08 15:31:31,560 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************

STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = hadoop/127.0.1.1
STARTUP_MSG:   args = [-format]
STARTUP_MSG:  version = 3.1.3
*************************************************************/

......
2020-01-08 15:31:35,677 INFO common.Storage: Storage directory /usr/local/hadoop/tmp/dfs/name **has been successfully formatted**.
2020-01-08 15:31:35,700 INFO namenode.FSImageFormatProtobuf: Saving image file /usr/local/hadoop/tmp/dfs/name/current/fsimage.ckpt_0000000000000000000 using no compression
2020-01-08 15:31:35,770 INFO namenode.FSImageFormatProtobuf: Image file /usr/local/hadoop/tmp/dfs/name/current/fsimage.ckpt_0000000000000000000 of size 393 bytes saved in 0 seconds .
2020-01-08 15:31:35,810 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
2020-01-08 15:31:35,816 INFO namenode.FSImage: FSImageSaver clean checkpoint: txid = 0 when meet shutdown.
2020-01-08 15:31:35,816 INFO namenode.NameNode: SHUTDOWN_MSG:  
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at hadoop/127.0.1.1
*************************************************************/
```

如果在这一步时提示 **Error: JAVA_HOME is not set and could not be found.** 的错误，则说明之前设置 JAVA_HOME 环境变量那边就没设置好，请按教程先设置好 JAVA_HOME 变量，否则后面的过程都是进行不下去的。如果已经按照前面教程在.bashrc文件中设置了JAVA_HOME，还是出现 **Error: JAVA_HOME is not set and could not be found.** 的错误，那么，请到hadoop的安装目录修改配置文件“/usr/local/hadoop/etc/hadoop/hadoop-env.sh”，在里面找到“export JAVA_HOME=${JAVA_HOME}”这行，然后，把它修改成JAVA安装路径的具体地址，比如，“export JAVA_HOME=/usr/lib/jvm/default-java”，然后，再次启动Hadoop。

接着开启 NameNode 和 DataNode 守护进程。

```bash
cd /usr/local/hadoop./sbin/start-dfs.sh  #start-dfs.sh是个完整的可执行文件，中间没有空
```

若出现如下SSH提示，输入yes即可。

启动时可能会出现如下 WARN 提示：WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform… using builtin-java classes where applicable WARN 提示可以忽略，并不会影响正常使用。

这个并不是 ssh 的问题，可通过设置 Hadoop 环境变量来解决。首先按键盘的 **ctrl + c** 中断启动，然后在 ~/.bashrc 中，增加如下两行内容（设置过程与 JAVA_HOME 变量一样，其中 HADOOP_HOME 为 Hadoop 的安装目录）：

```shell
export HADOOP_HOME=/usr/local/hadoopexport HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
```

保存后，务必执行 `source ~/.bashrc` 使变量设置生效，然后再次执行 `./sbin/start-dfs.sh` 启动 Hadoop。

启动完成后，可以通过命令 `jps` 来判断是否成功启动，若成功启动则会列出如下进程: “NameNode”、”DataNode” 和 “SecondaryNameNode”（如果 SecondaryNameNode 没有启动，请运行 sbin/stop-dfs.sh 关闭进程，然后再次尝试启动尝试）。如果没有 NameNode 或 DataNode ，那就是配置不成功，请仔细检查之前步骤，或通过查看启动日志排查原因。

Hadoop无法正常启动的解决方法

一般可以查看启动日志来排查原因，注意几点：

- 启动时会提示形如 “DBLab-XMU: starting namenode, logging to /usr/local/hadoop/logs/hadoop-hadoop-namenode-DBLab-XMU.out”，其中 DBLab-XMU 对应你的机器名，但其实启动日志信息是记录在 /usr/local/hadoop/logs/hadoop-hadoop-namenode-DBLab-XMU.log 中，所以应该查看这个后缀为 **.log** 的文件；
- 每一次的启动日志都是追加在日志文件之后，所以得拉到最后面看，对比下记录的时间就知道了。
- 一般出错的提示在最后面，通常是写着 Fatal、Error、Warning 或者 Java Exception 的地方。
- 可以在网上搜索一下出错信息，看能否找到一些相关的解决方法。

此外，**若是 DataNode 没有启动**，可尝试如下的方法（注意这会删除 HDFS 中原有的所有数据，如果原有的数据很重要请不要这样做）：

```bash
# 针对 DataNode 没法启动的解决方法
cd /usr/local/hadoop
./sbin/stop-dfs.sh   # 关闭
rm -r ./tmp     # 删除 tmp 文件，注意这会删除 HDFS 中原有的所有数据
./bin/hdfs namenode -format   # 重新格式化 NameNode
./sbin/start-dfs.sh  # 重启
```

  

成功启动后，可以访问 Web 界面 [http://localhost:50070](http://localhost:50070/)（localhost为hdfs主机IP） 查看 NameNode 和 Datanode 信息，还可以在线查看 HDFS 中的文件。

## 运行Hadoop伪分布式实例

上面的单机模式，grep 例子读取的是本地数据，伪分布式读取的则是 HDFS 上的数据。要使用 HDFS，首先需要在 HDFS 中创建用户目录：

```bash
./bin/hdfs dfs -mkdir -p /user/hadoop
```

  

> 注意
>
> 这里的命令是以”./bin/hadoop dfs”开头的Shell命令方式，实际上有三种shell命令方式。
> \1. hadoop fs
> \2. hadoop dfs
> \3. hdfs dfs
>
> hadoop fs适用于任何不同的文件系统，比如本地文件系统和HDFS文件系统
> hadoop dfs只能适用于HDFS文件系统
> hdfs dfs跟hadoop dfs的命令作用一样，也只能适用于HDFS文件系统

接着将 ./etc/hadoop 中的 xml 文件作为输入文件复制到分布式文件系统中，即将 /usr/local/hadoop/etc/hadoop 复制到分布式文件系统中的 /user/hadoop/input 中。我们使用的是 hadoop 用户，并且已创建相应的用户目录 /user/hadoop ，因此在命令中就可以使用相对路径如 input，其对应的绝对路径就是 /user/hadoop/input:

```bash
./bin/hdfs dfs -mkdir input./bin/hdfs dfs -put ./etc/hadoop/*.xml input
```

复制完成后，可以通过如下命令查看文件列表：

```bash
./bin/hdfs dfs -ls input
```

伪分布式运行 MapReduce 作业的方式跟单机模式相同，区别在于伪分布式读取的是HDFS中的文件（可以将单机步骤中创建的本地 input 文件夹，输出结果 output 文件夹都删掉来验证这一点）。

```bash
./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar grep input output 'dfs[a-z.]+'
```

查看运行结果的命令（查看的是位于 HDFS 中的输出结果）：

```bash
./bin/hdfs dfs -cat output/*
```

  我们也可以将运行结果取回到本地：

```bash
rm -r ./output    # 先删除本地的 output 文件夹（如果存在）
./bin/hdfs dfs -get output ./output     # 将 HDFS 上的 output 文件夹拷贝到本机
cat ./output/*
```

Hadoop 运行程序时，输出目录不能存在，否则会提示错误 “org.apache.hadoop.mapred.FileAlreadyExistsException: Output directory hdfs://localhost:9000/user/hadoop/output already exists” ，因此若要再次执行，需要执行如下命令删除 output 文件夹:

```bash
./bin/hdfs dfs -rm -r output    # 删除 output 文件夹
```


运行程序时，输出目录不能存在

运行 Hadoop 程序时，为了防止覆盖结果，程序指定的输出目录（如 output）不能存在，否则会提示错误，因此运行前需要先删除输出目录。在实际开发应用程序时，可考虑在程序中加上如下代码，能在每次运行时自动删除输出目录，避免繁琐的命令行操作：

```java
Configuration conf = new Configuration();
Job job = new Job(conf);
 
/* 删除输出目录 */
Path outputPath = new Path(args[1]);
outputPath.getFileSystem(conf).delete(outputPath, true);
```

若要关闭 Hadoop，则运行

```bash
./sbin/stop-dfs.sh
```

注意

下次启动 hadoop 时，无需进行 NameNode 的初始化，只需要运行 `./sbin/start-dfs.sh` 就可以！