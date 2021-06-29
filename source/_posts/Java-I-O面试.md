---
title: 'Java:I/O面试'
top: false
cover: true
author: 张文军
date: 2020-07-15 20:08:07
tags: 
  - 字节流
  - 字符流
  - 缓冲流
  - 转换流
  - 序列化流
  - IO面试
category: I/O
summary: IO面试
---
<center>更多内容请关注：</center>

![Java快速开发学习](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1586869254.png)

<center><a href="https://wjhub.gitee.io">锁清秋</a></center>

----

## Java中有几种类型的流

- 按照流的方向:输入流(inputStream) 和输出流(outputStream) 。
- 按照实现功能分:节点流(可以从或向一个特定的地方(节点)读写数据。如FileReader)和处理流(是对一个已存在的流的连接和封装,通过所封装的流的功能调用实现数据读写。如BufferedReader.处理流的构造方法总是要带一个其他的流对象做参数。一个流对象经过其他流的多次包装，称为流的链接。)
- 按照处理数据的单位:字节流和字符流。字节流继承于InputStream 和OutputStream, 字符流继承于InputStreamReader和OutputStreamWriter.

> 方向：输入流和输出流
> 功能：节点流和处理流
> 处理数据单位：字节流和字符流

![流分类](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1594817577.png)

## 字节流如何转为字符流

字节输入流转字符输入流通过InputStreamReader实现，该类的构造函数可以传入InputStream对象。
字节输出流转字符输出流通过OutputStreamWriter实现，该类的构造函数可以传入OutputStream对象。

## 字节流和字符流的区别

- 字节流读取的时候，读到一个字节就返回一个字节;字符流使用了字节流读到一个或多个字节(中文对应的字节数是两个,在UTF-8码表中是3个字节)时。先去查指定的编码表,将查到的字符返回。字节流可以处理所有类型数据,如:图片，MP3, AVI 视频文件,而字符流只能处理字符数据。只要是处理纯文本数据，就要优先考虑使用字符流，除此之外都用字节流。
- 字节流主要是操作byte类型数据,以byte数组为准，主要操作类就OutputStream、InputStream，字符流处理的单元为2个字节的Unicode字符,分别操作字符、字符数组或字符串,而字节流处理单元为1个字节,操作字节和字节数组。所以字符流是由Java虚拟机将字节转化为2个字节的Unicode字符为单位的字符而成的,所以它对多国语言支持性比较好!如果是音频文件、图片、歌曲，就用字节流好点,如果是关系到中文(文本)的,用字符流好点。在程序中一个字符等于两个字节，java 提供了Reader、Writer 两个专门操作字符流的类。

> 主要区别：处理单位不同，字节流处理的是以字节（byte）为单位的数据，而字符流处理的是以字符（char）为单位的数据。

## 什么是java序列化，如何实现java列化?

序列化就是一种用来处理对象流的机制，所谓对象流也就是将对象的内容进行流化。可以对流化后的对象进行读写操作，也可将流化后的对象传输于网络之间。列化是为了解决在对对象流进行读写操作时所引发的问题。

### 序列化的实现

将需要被序列化的类实现Serializable 接口（该接口没有需要实现的方法,implements Serializable只是为了标注该对象是可被序列化的）然后使用一个输出流(如: FileOutputStream)来构造一个ObjectOutputStream(对象流，ObjectXXXXStream也属于处理流，是对其他流的封装和处理，即在构造的时候也需要传入一个流对象)对象,接着,使用ObjectOutputStream对象的writeObject(Object obj)方法就可以将参数为obj的对象写出(即保存其状态),要恢复的话则用输入流。（对不想持久化数据可添加`transient`关键字）
