---
title: 并发：openjdk编译调试、java线程模型
date: 2020-01-16 19:12:37
tags: 并发
category: 并发
summary: 并发：openjdk编译调试、java线程模型
top: true
cover: true
author: 张文军
---

![](/images/favicon.png)


# 并发：openjdk编译调试、java线程模型

## 一 、 java当中的线程和操作系统的线程之间的关系
### 1、关于操作系统的线程
 - linux操作系统的线程控制原语

 ```java
 int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                          void *(*start_routine) (void *), void *arg);
                          
 ```

- pthread_create会创建一个线程，这个函数是linux系统的函数，可以用C或者C++直接调用，上面信息也告诉程序员这个函数在**pthread.h**， 这个函数有四个参数

| pthread_t *thread               | 传出参数，调用之后会传出被创建线程的id     | 定义 pthread_t pid; 继而 取地址 &pid                   |
| ------------------------------- | ------------------------------------------ | ------------------------------------------------------ |
| const pthread_attr_t *attr      | 线程属性，关于线程属性是linux的知识        | 在学习pthread_create函数的时候一般穿NULL，保持默认属性 |
| void *(*start_routine) (void *) | 线程的启动后的主体函数 相当于java当中的run | 需要你定义一个函数，然后传函数名即可                   |
| void *arg                       | 主体函数的参数                             | 如果没有可以传NULL                                     |



在linux上启动一个线程的代码：

```c++
#include <pthread.h>//头文件
#include <stdio.h>
pthread_t pid;//定义一个变量，接受创建线程后的线程id
//定义线程的主体函数
void* thread_entity(void* arg)
{   
    printf("i am new Thread!");
}
//main方法，程序入口，main和java的main一样会产生一个进程，继而产生一个main线程
int main()
{
    //调用操作系统的函数创建线程，注意四个参数
    pthread_create(&pid,NULL,thread_entity,NULL);
    //usleep是睡眠的意思，那么这里的睡眠是让谁睡眠呢？
    //为什么需要睡眠？如果不睡眠会出现什么情况
    usleep(100);
    printf("main\n");
}
```



### 2、 操作系统和 JAVA 线程模式的关系

- 在 Java 中启动一个线程

```java
  
  
  /**
   * @author 张文军
   * @Description: TODO:
   * @Company:南京农业大学工学院
   * @version:1.0
   * @date 2020/1/1619:54
   */
  public class ThreadTest1 {
      public static void main(String[] args) {
          new Thread(new Runnable() {
              @Override
              public void run() {
                  // TODO:
                  System.out.println(" thread1");
              }
          }, "thread1").start();
  
      }
  }
  
```

- 这里启动的线程和上面我们通过linux的pthread_create函数启动的线程有什么关系呢？

- 查看源码可知：

  - 