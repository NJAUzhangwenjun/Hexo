---
title: Java中的基本数据类型
top: false
cover: true
author: 张文军
date: 2020-07-13 20:11:26
tags: Java中的基本数据类型
category: Java基础
summary: Java中的基本数据类型
---
<center>更多内容请关注：</center>

![Java快速开发学习](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1586869254.png)

<center><a href="https://wjhub.gitee.io">锁清秋</a></center>

----

## 字符型常量和字符串常量的区别?

比较：

1. 字符常量用单引号，而字符串是双引号；
2. 字符常量相当于一个整形值（ASCII值，可以参加表达式运算），而字符串常量代表一个地址值（该字符在内存中的地址）；
3. 字符常量在Java中占两个字节；而字符串常量占若干个字节大小（没有大小规定）；

## Java中四大类基本数据结构

1. 整形:

> 1. byte ,2. short ,3. int ,4. long

2. 浮点型：

>1. float ,2.dable

3. 字符型：

> 1. char

4. 布尔型：

> boolean

![八种数据类型](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1594643642.png)

### 整型（byte、short、int、long）

虽然byte、short、int、long 数据类型都是表示整数的，但是它们的取值范围可不一样。

byte 的取值范围：-128～127（-2的7次方到2的7次方-1）

short 的取值范围：-32768～32767（-2的15次方到2的15次方-1）

int 的取值范围：-2147483648～2147483647（-2的31次方到2的31次方-1）

long 的取值范围：-9223372036854774808～9223372036854774807（-2的63次方到2的63次方-1）

### 浮点型（float、double）

float 和 double 都是表示浮点型的数据类型，它们之间的区别在于精确度的不同。

float（单精度浮点型）取值范围：3.402823e+38～1.401298e-45（e+38 表示乘以10的38次方，而e-45 表示乘以10的负45次方）

double（双精度浮点型）取值范围：1.797693e+308～4.9000000e-324（同上）

double 类型比float 类型存储范围更大，精度更高。

>通常的浮点型数据在不声明的情况下都是double型的，如果要表示一个数据时float 型的，可以在数据后面加上 "F" 。浮点型的数据是不能完全精确的，有时候在计算时可能出现小数点最后几位出现浮动，这时正常的。

### 字符型（char）

char 有以下的初始化方式：

char ch = 'a'; // 可以是汉字，因为是Unicode编码

char ch = 1010; // 可以是十进制数、八进制数、十六进制数等等。

char ch = '\0'; // 可以用字符编码来初始化，如：'\0' 表示结束符，它的ascll码是0，这句话的意思和 ch = 0 是一个意思。

Java是用unicode 来表示字符，“中” 这个中文字符的unicode 就是两个字节。

String.getBytes(encoding) 方法获取的是指定编码的byte数组表示。

通常gbk / gb2312 是两个字节，utf-8 是3个字节。

如果不指定encoding 则获取系统默认encoding 。

### 布尔型（boolean）

boolean 没有什么好说的，它的取值就两个：true 、false 。

## 基本类型之间的转换

>boolean 与其他7种数据类型都不能相互转换，其他7种数据类型可以相互转换；

转换分为自动转换和强制转换：

>自动转换（隐式）：无需任何操作。
>强制转换（显式）：需使用转换操作符（type）。

将6种数据类型按下面顺序排列一下：

double > float > long > int > short > byte

如果从小转换到大，那么可以直接转换，而从大到小，或char 和其他6种数据类型转换，则必须使用强制转换。
>低精度向高精度转换可以直接转换，高精度向低精度转换需要强制类型转换；
>高精度数据类型需要内存大，低精度需要内存小，高精度向低精度转换时，需要内存需要扩宽，即发生强制类型转换，而低精度向高精度转换，内存足够，则不需要。

即：`小转大 -> 不需要强制转换，大转小 -> 需要强制类型转换（type关键字）`

```java
byte a = 1;
short b = 1;

b = a;
a = (byte) b;
```

注意：
> `如果等号右边整数超出目标类型范围时，将对此数除以目标范围再赋值给左边；浮点类型赋给整数类型的时候，会发生截尾（truncation）。也就是把小数的部分去掉，只留下整数部分。此时如果整数超出目标类型范围，一样将对目标类型的范围取余数。`

即：以目标值范围为圆周，以目标范围为大小的周期，对目标值取值；

![byte](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1594651831.png)

例如 ：

```java

//范围为[-128,127] 即256
byte a = 1;
short b = 18;
short c = 128;
short d = 200;
short e = 257;

a = (byte) b;
System.out.println("a = " + a);

a = (byte) c;
System.out.println("a = " + a);

a = (byte) d;
System.out.println("a = " + a);

a = (byte) e;
System.out.println("a = " + a);

```

结果：![byte类型赋值给short类型](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1594652268.png)

## 赋值及表达式中的类型转换（都向高精度类型转换）

### 字面值赋值

在使用字面值对整数赋值的过程中，可以将int literal赋值给byte short char int，只要不超出范围。这个过程中的类型转换时自动完成的，但是如果你试图将long literal赋给byte，即使没有超出范围，也必须进行强制类型转换。例如 byte b = 10L；是错的，要进行强制转换。

### 表达式中的自动类型提升

除了赋值以外，表达式计算过程中也可能发生一些类型转换。在表达式中，类型提升规则如下：

· 所有byte/short/char都被提升为int。

· 如果有一个操作数为long，整个表达式提升为long。float和double情况也一样。

## 其他

相同字节类型数据相互转换，需要强制转换；

如：

```java
//short和char都占2个字节

short g = 0;
char x = 0;
x = (char) g;
g = (short) x;

```

>从byte到char的过程其实是byte-->int-->char
