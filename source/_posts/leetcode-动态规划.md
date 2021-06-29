---
title: leetcode-动态规划
date: 2020-11-20 22:23:34
tags: leetcode-动态规划
category: leetcode
summary: leetcode-动态规划
top: false
cover: true
author: 张文军
---
<center>更多内容请关注：</center>

![Java快速开发学习](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1586869254.png)

<center><a href="https://wjhub.gitee.io">锁清秋</a></center>

----

# leetcode-动态规划 （穷举最值）

> 动态规划一般有三个步骤：
> （首先，如果问题存在“重叠子问题，那么一般情况下就可以使用 动态规划”）
> 
> 1. 找出最优子结构
> 2. 找到转台转移方程
> 3. 剪枝

## 1. 斐波那契数列

```text
斐波那契数，通常用 F(n) 表示，形成的序列称为斐波那契数列。该数列由 0 和 1 开始，
后面的每一项数字都是前面两项数字的和。也就是：

 F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1.


 给定 N，计算 F(N)。



 示例 1：

 输入：2
输出：1
解释：F(2) = F(1) + F(0) = 1 + 0 = 1.


 示例 2：

 输入：3
输出：2
解释：F(3) = F(2) + F(1) = 1 + 1 = 2.


 示例 3：

 输入：4
输出：3
解释：F(4) = F(3) + F(2) = 2 + 1 = 3.




 提示：


 0 ≤ N ≤ 30

 Related Topics 数组

```

```java 
	public int fib(int N) {
		if (N <= 1) {
			return N;
		}
		int pre = 0;
		int cur = 1;
		int ct = 1;
		while (ct++ < N) {
			int temp = cur + pre;
			pre = cur;
			cur = temp;
		}
		return cur;
	}

```


## 2. 钱币问题

```text

硬币。给定数量不限的硬币，币值为25分、10分、5分和1分，编写代码计算n分有几种表示法。
 (结果可能会很大，你需要将结果模上1000000007)

 示例1:


 输入: n = 5
 输出：2
 解释: 有两种方式可以凑成总金额:
5=5
5=1+1+1+1+1


 示例2:


 输入: n = 10
 输出：4
 解释: 有四种方式可以凑成总金额:
10=10
10=5+5
10=5+1+1+1+1+1
10=1+1+1+1+1+1+1+1+1+1


 说明：

 注意:

 你可以假设：


 0 <= n (总金额) <= 1000000

 Related Topics 动态规划
 
 ```
 
```java


```


