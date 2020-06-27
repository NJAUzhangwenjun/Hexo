---
title: '剑指 offer'
date: 2019-11-23 03:17:01
category: 
    - '算法'
author: 张文军
tags: 
    - '算法'
    -  '剑指 offer '
top: true
cover: true
summary: 记录了个人对剑指offer的理解和解题思路
---



![](/images/favicon.png)
# 剑指 offer

## 1、 二维数组中的查找

### 题目描述：

> 在一个二维数组中（每个一维数组的长度相同），每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。



### 分析：

> 1. 数组中每一行第一个数记作 m ，由给出的二维数组可知，m 是该行最小数据，也是该列从
>    0 行开始到该行最大数据
> 2.  所以，比较 target 和该行第一个数据，如果 target 比该行第一个数据大，而比该行最后一个数据小，
>    就说明 target 在这一行里寻找



```java

public class Solution {
    public boolean Find(int target, int [][] array) {
        
        if (array.length == 0 || array[0].length==0) {
            return false;
        }
        for (int[] ints : array) {
            if (target == ints[0] || target ==ints[ints.length-1]) {
                return true;
            } else if (target > ints[0] && target <ints[ints.length-1]) {
                for (int anInt : ints) {
                    if (target == anInt) {
                        return true;
                    }
                }
            }       
        }
        return false;
    }
}
```





## 2. 替换空格

### 题目描述：

> 请实现一个函数，将一个字符串中的每个空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。




### 分析：

> 1. 方法一：直接用 Java 函数替换
> 2. 方法二： 新建字符串 ，遍历原先字符串放入新的中，如果遇到空格就替换

```java
public class Solution {
    public String replaceSpace(StringBuffer str) {
    	return str.toString().replace(" ", "%20");
    }
}
```



## 3、 从未到头打印链表

### 题目描述：

> 输入一个链表，按链表从尾到头的顺序返回一个ArrayList。

### 分析：

> 1. ListNode 是链表，只能从头遍历，即 “ 先进后出 ”
> 2. - ArrayList 中有个方法是 add(index,value)，可以指定 index 位置插入 value 值
>    - 在列表中指定的位置上插入指定的元素（可选操作）。将当前处于该位置（如果有的话）和任何后续元素的元素移到*右边*（添加一个到它们的索引）
> 3. 所以我们在遍历 listNode 的同时将每个遇到的值插入到 list 的 0 位置（每添加一个元素，之前
>    添加的元素都将会被向后（右）移动）
> 4. 最后输出 listNode 即可得到逆序链表

```java
import java.util.ArrayList;
public class Solution {
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        ArrayList<Integer> integers = new ArrayList<>();

        ListNode curr =listNode;
            while (curr!= null) {
                integers.add(0,curr.val);
                curr = curr.next;
            }
        return integers;
    }
}
```





## 4 、 重建二叉树

### 题目描述

> 输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。

###  分析：

> -- 根据前序序列第一个结点确定根结点
> -- 根据根结点在中序序列中的位置分割出左右两个子序列
> -- 对左子树和右子树分别递归使用同样的方法继续分解
>
> 1. 首先通过前序遍历找到头节点 ， 然后再通过中序遍历找到左子树和右子树
>
>    - 如：前序遍历节点为 1 ，在中序遍历中找到1 的位置，1 的左边为左子树节点（4,7,2），
>
>   右边为右子树节点（5,3,8,6）。
>
> 2. 再进行前序遍历的找头结点（此时前序列表已经变为 {2,4,7,3,5,6,8} 即从上一次的下一个位置开始），
>
>    - 如找到头节点为 2 ，在（4,7,2）中 2 的左边为 2 的左子树节点， 2 的右边为右子树节点
>     （此时，2 的左子树节点为 （4,7），而右子树为空）
>
> 3. 再进行前序遍历的找头结点（此时前序列表已经变为 {4,7,3,5,6,8} 即从上一次的下一个位置开始），
>    - 如找到头节点为 4 ，在（4,7）中 4 的左边为 4 的左子树节点， 4 的右边为右子树节点，（此时，4的左子树节点为 空，而右子树为 7 ）
>    - 即，此时 ，概树的左子树遍历完毕。
>
> 4. 重复上述步骤遍历右子树，可完成所有树节点的遍历
>
>    

```java
/**
 * Definition for binary tree
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
import java.util.Arrays;
public class Solution {
    public TreeNode reConstructBinaryTree(int [] pre,int [] in) {
                if (pre.length == 0 || in.length == 0) {
            return null;
        }
        TreeNode root = new TreeNode(pre[0]);
        // 在中序中找到前序的根
        for (int i = 0; i < in.length; i++) {
            if (in[i] == pre[0]) {
                // 左子树，注意 copyOfRange 函数，左闭右开
                root.left = reConstructBinaryTree(Arrays.copyOfRange(pre, 1, i + 1), Arrays.copyOfRange(in, 0, i));
                // 右子树，注意 copyOfRange 函数，左闭右开
                root.right = reConstructBinaryTree(Arrays.copyOfRange(pre, i + 1, pre.length), Arrays.copyOfRange(in, i + 1, in.length));
                break;
            }
        }
        return root;
    }
}
```





## 5、用两个栈实现队列

###  题目描述

> 用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。

###  分析：

> 栈：先进后出
> 队列：先进先出
>
> 1. 可先将数据在插入时放入 stack1 中
> 2. 当弹出时，当 stack2 不为空，弹出 stack2 栈顶元素，如果 stack2 为空，将 stack1 中的全部数逐个出栈入栈 stack2，再弹出 stack2 栈顶元素

```java
import java.util.Stack;
public class Solution {
   Stack<Integer> stack1 = new Stack<Integer>();
    Stack<Integer> stack2 = new Stack<Integer>();

    public void push(int node) {
        stack1.push(node);

    }
    public int pop() {
        if (stack2.size() <= 0) {
            while (stack1.size() > 0) {
                stack2.push(stack1.pop());
            }
        }
        return stack2.pop();
    }
}
```



## 6、斐波那契数列

### 题目描述：

> **斐波那契数列**指的是这样一个数列 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233，377，610，987，1597，2584，4181，6765，10946，17711，28657，46368........
>
> 自然中的斐波那契数列
>
> 这个数列从第3项开始，每一项都等于前两项之和。

### 分析：

> F[n] = F[n-1] + F[n-2]   (n>=3,F[1]=1,F[2]=1)

```java
public class Solution {
    public int Fibonacci(int n) {
        if(n ==0){
            return 0;
        }
        if (n == 1 || n == 2) {
            return 1;
        } else {
            return Fibonacci(n - 1) + Fibonacci(n - 2);
        }     
    }
}
```

### 优化：

#### 优化一 :

> 利用数组将 每一次的前两位数据存起来，这样就不会出现递归将内存撑爆的情况

```java
import java.util.ArrayList;
public class Solution {
    public int Fibonacci(int n) {
        ArrayList<Integer> list = new ArrayList<>();
        list.add(0);
        list.add(1);
        for (int i = 2; i <= n; i++) {
            System.out.println(list.get(i - 1)+" + "+list.get(i - 2));
            list.add(list.get(i - 1) + list.get(i - 2));
        }
        return list.get(n);     
    }
}
```

#### 优化二 ：

> 用3个变量来表示

```java
public class Solution {
    public int Fibonacci(int n) {
        if (n < 3) {
            return 1;
        }
        int f1 = 1;
        int f2 = 1;
        int cur = 0;
        for (int i = 3; i <= n; i++) {
            cur = f1 + f2;
            f2 = f1;
            f1 = cur;
        }
        return f1;
}
```



## 7 、跳台阶

### 问题描述：

> 一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法（先后次序不同算不同的结果）。

### 分析：

> 本质上还是斐波那契数列，所以迭代也可以求
>
> 分析方法：
>
> - 如果台阶数为1 ，只能一次一级 1种
> - 如果台阶数为 2 ，可以 2次 一级，也可以1 次两级，所以有两种
> - 如果台阶数为 3 :
>   - 最后一次跳两级到第3阶：剩下台阶数为1，起跳到 1 阶有 1 种
>   - 最后一次跳一级到第3阶：剩下台阶数为 2，起跳到 2阶有 2 种
>   - 总的解法有 1+2 = 3 种
>
> 通过分类讨论，问题规模就减少了 ：
>
> - 如果台阶数为 n 阶：
>
>   - 如果先跳两级：剩下台阶数为 n-2 ，起跳到 n-2 阶设为有 pre2种
>
>   - 如果先跳一级：剩下台阶数为 n-1 ，起跳到 n-1 阶设为有 pre1种
>
>   - 总的解法有：pre2 + pre1 种解法
>
>     
>
> - 故发现，第 n 阶的解法总是 需要知道 n -1 阶和 n - 2阶的解法
>
> - 即也就是： f(n) = f(n-1) + f(n-2)   { n >2 }
>
>   

### 解法一：递归法

```java
public class Solution {
    public int JumpFloor(int target) {
        if(target<3){
            return target;
        }
        else{
            return JumpFloor(target-1)+JumpFloor(target-2);
        }
    }
}
```

### 解法二：利用数组法

```java
import java.util.ArrayList;
public class Solution {
    public int JumpFloor(int target) {
        if(target<3){
            return target;
        }
        else{
            ArrayList<Integer> list = new ArrayList<>();
            list.add(1);
            list.add(2);
            for (int i = 3; i <= target; i++) {
                list.add(list.get(i - 2) + list.get(i - 3));
            }
            return list.get(target-1); 
        }
    }
}
```

### 解法三：利用变量

```java
public class Solution {
    public int JumpFloor(int target) {
        if(target<3){
            return target;
        }
        else{
            int a = 1;
            int b = 2;
            int curr = 0;
            for (int i = 3; i <= target; i++) {
                curr = a + b;
                a = b;
                b = curr;
            }
            return curr; 
        }
    }
}
```

##  8 、变态台阶

###  问题描述：

>一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

### 分析：

> - 台阶数为 1 时 ，只能 一次 1阶 一种
> - 台阶数为 2 时 ：
>   - 可以一次两步到2阶 1种
>   - 可以两次一步到2阶 1种
>   - 总共 1+1 = 2 种
> - 台阶数为3时：
>   - 跳一次 3  阶到 第三阶时，没有剩下的， 只有 1 种
>   - 跳 一次 2 阶到第三阶时，剩下1阶，只能是一次一步， 1 种
>   - 跳 一次 1 阶到第三阶时，省下 2 阶，有前面台阶数为2时，可知，有2种
>   - 总共 1+ 1 +2 = 4  种
> - 台阶数为4时：
>   - 跳一次 4 阶到第四阶时，没有剩下的， 只有 1 种
>   - 跳 一次 3 阶到第四阶时，剩下1阶，只能是一次一步， 1 种
>   - 跳 一次 2 阶到第四阶时，剩下 2 阶，由前面台阶数为2时，可知，有 2 种
>   - 跳 一次 1 阶到第四阶时，剩下 3阶，由前面台阶数为 3 时，可知，有 4 种
>   - 总共有 1+ 1 + 2 + 4 = 8 种
> - 台阶数为 n 时：
>   - 跳 x 步到 n 有几种的和，跟前 n - 1 种状态有关
>   - 即：f(n) =f(1) + f(1) + ··· + f(n-1) 
>   - 即：f(n) = f(n-1) * 2 
>   - 即： f(n) = 2的n-1次方

### 最简单的一种做法：

```java
public class Solution {
   public int JumpFloorII(int target) {
        return (int) Math.pow(2, target -1);
    }
}
```

##  9、矩形覆盖

### 问题描述：

> 我们可以用2 x1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2 x1的小矩形无重叠地覆盖一个2xn的大矩形，总共有多少种方法？

### 分析：

> - n = 1 时：只能有一种
> - n = 2 时：两个一起横着或竖着 两种
> - n = 3 时：
>   - 第一个竖着时：剩下两个 由上面可知，有两种
>   - 第一个横着时，只能同时再横着一个 还剩下一个 有一种
>   - 总共有 1 + 2 = 3 种
> - n时：
> - 第一个竖着时：剩下 n-1 个 由上面可知，设有pre1 种
> - 第一个横着时，只能同时再横着一个 还剩下 n -2 个 ,设有pre2 种
> - 总的是：pre1+pre2 总
> - 即：第 n 个的种数总是和前 n - 1和 n -2 次的次数有关
> - 即：f(n) = f(n-1) + f(n-2)  {  n > 2}

```java
public class Solution {
    public int RectCover(int target) {
        if (target < 3) {
            return target;
        } else {
            return RectCover(target - 1) + RectCover(target - 2);
        } 
    }
}
```

> 优化方法可完全参照上面 6. 斐波那契数列的优化方法进行优化



##  10、调整数组顺序使奇数位于偶数前面

###  问题描述：

> 输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。

### 分析：

不开辟新数组：

    1.用两个下标i,j进行遍历;
    2.当i走到偶数时停下，并让j从i的后一个元素开始遍历；(若i走到队尾则循环结束)
    3.若j所指的是偶数则继续前进，j遇到奇数则停下(如果j都没遇到奇数则在队尾停下，结束。)。
    4.此时j所指的是奇数，i所指的是偶数(i到j-1都是偶数)。
    5.则可以用临时变量temp保存j对应的值，然后从j-1开始到i，挨个后移一位。
    6.将temp保存的值插入到i的位置。


```java
    /**
     * >>i++往前走碰到偶数停下来，j = i+1
     * >>若 a[j]为偶数，j++前进，直到碰到奇数
     * >>a[j]对应的奇数插到a[i]位置，j经过的j-i个偶数依次后移
     * >>如果j==len-1时还没碰到奇数，证明i和j之间都为偶数了，完成整个移动
     *
     * @param
     */
    public void reOrderArray(int[] array) {
        int len = array.length;
        if (len <= 1) {
            return;
        }
        int i = 0;
        while (i < len) {
            //如果i所指的元素是奇数，则继续前进
            if (array[i] % 2 == 1) {
                i++;
            } else {
                //当i遇到偶数停下时，j从i的后一位开始走
                int j = i + 1;
                //当j所指的元素也是偶数时，则j向后移动
                while (array[j] % 2 == 0) {
                    //当j移到队尾，则说明i到队尾全是偶数，已满足题目的奇偶分离要求
                    if (j == len - 1) {
                        return;
                    }
                    j++;
                }
                //此时j为奇数，i为偶数，用temp保存array[j]的值
                int temp = array[j];
                //把i到j-1的元素往后移一位
                while (j > i) {
                    array[j] = array[j - 1];
                    j--;
                }
                //把保存在temp中的原第j个元素的值赋给i，此时i就变成奇数了，并进入下个循环
                array[i] = temp;
            }

        }


    }
```

## 11、包含min函数的栈

### 题目描述

> 定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））

### 分析

> 用两个栈，一个（stack）栈用来记录原始数据，另一个（stackmin）用来记录当前放入原始数据时当前栈（stack）中的最小值

### 代码实现

```java

import java.util.Stack;

public class SolutionMin {

    private Stack<Integer> stack = new Stack<Integer>();
    private Stack<Integer> stackmin = new Stack<Integer>();
    private Integer min;

    /**
     * 定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））
     *
     * @param node
     */
    public void push(int node) {
        stack.push(node);
        if (stackmin.empty()) {
            stackmin.push(node);
            min = node;
        } else {
            if (min > node) {
                min = node;
            }
            stackmin.push(min);
        }
    }

    public void pop() {
        stack.pop();
        stackmin.pop();
    }

    public int top() {
        return stack.peek();
    }

    public int min() {

        return stackmin.peek();
    }
}

```

## 12、栈的压入、弹出序列

### 题目描述

> 输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。
> 假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）

### 分析

> 按照入栈的顺序重新入栈一次，但是在入栈的时候，要结合出栈的顺序来判断是否应该出栈。比如将1加入栈中时，发现出栈的第一个元素为4，不等于1，因此就直接入栈，直到将4加入栈中的时候，因为为出栈的第一个元素，因此要执行一个pop操作，接着循环查看出栈序列的下一个元素是否和当前栈顶元素相同，如果是，也就出栈。最后判断栈是否为空，如果为空，则表示弹栈顺序正确。

### 代码实现

```java
import java.util.ArrayList;
import java.util.Stack;
public class Solution{
    public boolean IsPopOrder(int[] pushA, int[] popA) {
        Stack<Integer> stack = new Stack<Integer>();
        int j = 0;
        for (int i = 0; i < pushA.length; i++) {
            stack.push(pushA[i]);
            while (!stack.empty()&& stack.peek() == popA[j] && j < popA.length ) {
                stack.pop();
                j++;
            }
        }
        return stack.empty();
    }
}
```

## 13、反转链表

### 题目描述

 > 输入一个链表，反转链表后，输出新链表的表头。

### 分析

  > 1、保存当前节点的前一个节点 pre
  > 2、用next记录当前节点的后一个节点
  > 3、将当前节点的下一个节点指向当前节点的前一个节点pre
  > 4、循环移动指针进入下一个节点（将当前节点赋给pre ，next 赋给当前节点）
  >
  >![](/images/链表反转1.png)

### 代码实现

```java
  public class SolutionReverseList {
    public ListNode ReverseList(ListNode head) {

        ListNode pre = null;

        while (head != null) {
            ListNode next = head.next;
            head.next = pre;

            pre = head;

            head = next;

        }
        return pre;
    }
  }
```

## 14、合并两个排序的链表

### 题目描述

 > 输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。

### 分析

  >方法一：
  > 将两个链表结点挨个进行比较，插入到一个新表中。

  >方法二：
  > 把list2往list1的中插。
  >
  >1. 比较list2与list1的值：
  >2. 当list2值小等于list1值时往list1的前面插，并让list2指向下一个元素
  >3. 否则不进行插入，list1指向下一个结点。
  >4. 重复上述操作，直到有一个链表为空
  >5. 判断是哪个链表空了，如果是list2则说明list2已全部插入直接返回头结点即可。如果是list1，则将剩下的list2  
  >6. 结点直接连到list1尾部，返回头结点即可。


### 代码实现

方法一：

```java
    public ListNode Merge(ListNode list1, ListNode list2) {

        ListNode hear = new ListNode(0);
        ListNode cur = hear;
        //比较处理两个链表
        while (true) {
            if (list1 != null && list2 != null && list1.val < list2.val) {
                cur.next = list1;
                list1 = list1.next;
                cur = cur.next;
            } else if (list1 != null && list2 != null && list1.val >= list2.val) {
                cur.next = list2;
                list2 = list2.next;
                cur = cur.next;
            } else {
                break;
            }

        }
        //处理剩余部分
        if (list1 != null) {
            cur.next = list1;
        } else {
            cur.next = list2;
        }
        return hear.next;
    }
```


## 15、复杂链表的复制

### 题目描述

 > 输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针指向任意一个节点），返回结果为复制后复杂链表的head。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空）

### 分析

  >1. 首先，用一个 map 保存原来 RandomListNode 和创建新的RandomListNode 的关系（此时，map中已经有了所有节点）。
  >2. 然后 遍历 map ，根据原理列表的对应关系，将map中创建的新的RandomListNode 的next指向map中的它对用的它的下一个节点，将map中创建的新的RandomListNode 的 random指向对应的节点
  >3. 返回新链表

### 代码实现

```java
public class SolutionClone {

    public RandomListNode Clone(RandomListNode pHead) {
        if (pHead == null) {
            return null;
        }
        Map<RandomListNode, RandomListNode> map = new HashMap<>();
        RandomListNode cur = pHead;
        while (cur != null) {
            map.put(cur, new RandomListNode(cur.label));
            cur = cur.next;
        }

        cur = pHead;
        while (cur != null) {
            map.get(cur).next = map.get(cur.next);
            map.get(cur).random = map.get(cur.random);
            cur = cur.next;
        }
        return map.get(pHead);
    }
}
```

## 16、两个链表的第一个公共结点

### 题目描述

 > 输入两个链表，找出它们的第一个公共结点。

### 分析

  > 将其中一个链表的所有节点保存在一个map中
  > 遍历另一个链表节点，如果该节点在map中存在，则该节点就是两个链表的第一个公共结点
  > 如果遍历完了整个链表，依然没有，则表示没有，返回null即可

### 代码实现

```java
import java.util.HashMap;
import java.util.Map;
public class Solution {
    public ListNode FindFirstCommonNode(ListNode pHead1, ListNode pHead2) {
        if (pHead1 == null || pHead2 == null) {
            return null;
        }
        Map<ListNode, ListNode> map = new HashMap<>();
        ListNode curr = pHead1;
        while (curr != null) {
            map.put(curr, curr);
            curr = curr.next;
        }

        curr = pHead2;
        while (curr != null) {
            if (map.containsKey(curr)) {
                return map.get(curr);
            } else {
                curr = curr.next;
            }
        }
        return null;
    }
}
```

## 17、孩子们的游戏(圆圈中最后剩下的数)

### 题目描述

 > 每年六一儿童节,牛客都会准备一些小礼物去看望孤儿院的小朋友,今年亦是如此。HF作为牛客的资深元老,自然也准备了一些小游戏。其中,有个游戏是这样的:首先,让小朋友们围成一个大圈。然后,他随机指定一个数m,让编号为0的小朋友开始报数。每次喊到m-1的那个小朋友要出列唱首歌,然后可以在礼品箱中任意的挑选礼物,并且不再回到圈中,从他的下一个小朋友开始,继续0...m-1报数....这样下去....直到剩下最后一个小朋友,可以不用表演,并且拿到牛客名贵的“名侦探柯南”典藏版(名额有限哦!!^_^)。请你试着想下,哪个小朋友会得到这份礼品呢？(注：小朋友的编号是从0到n-1)

 >如果没有小朋友，请返回-1

### 分析

  >使用一个list来模拟，并使用一个索引curChildIndex类指向删除的位置，当curChildIndex的值为list的size时，就让curChildIndex到头位置。

### 代码实现

```java
    public int LastRemaining_Solution(int n, int m) {
        if(n<1||m<1){
            return -1;
        }
        List<Integer> children = new ArrayList<>();
        //构建list
        for(int i = 0;i<n;i++){
            children.add(i);
        }
        int curChildIndex = -1;
        while(children.size()>1){
            for(int i = 0;i<m;i++){
                curChildIndex++;
                if(curChildIndex == children.size()){
                    curChildIndex = 0;
                }
            }
            children.remove(curChildIndex);
            //curChildIndex--的原因，因为新的children中curChildIndex指向了下一个元素，
            // 为了保证移动m个准确性，所以curChildIndex向前移动一位。
            curChildIndex--;
        }
        return children.get(0);
    }
```

## 18、链表中环的入口结点

### 题目描述

 > 给一个链表，若其中包含环，请找出该链表的环的入口结点，否则，输出null。

### 分析

  >用一个set循环保存节点信息，并在保存的同时查看该节点是否包含在set中，如果set中已经存在该节点，则该节点就是循环链表的入口节点

### 代码实现

```java
import java.util.HashSet;
import java.util.Set;
public class Solution {

    public ListNode EntryNodeOfLoop(ListNode pHead){
        Set<ListNode> set = new HashSet<>();

        ListNode head = pHead;
        while (head != null && !set.contains(head)) {
            set.add(head);
            head = head.next;
        }
        return head;
    }
}
```

## 19、二进制中1的个数

### 题目描述

 > 输入一个整数，输出该数二进制表示中1的个数。其中负数用补码表示。

### 分析


  >（搬运评论区大佬的解释）：
  >
  >>1. 如果一个整数不为0，那么这个整数至少有一位是1。如果我们把这个整数减1，那么原来处在整数最右边的1就会变为0，原来在1后面的所有的0都会变成1(如果最右边的1后面还有0的话)。其余所有位将不会受到影响。
  >>2.  举个例子：一个二进制数1100，从右边数起第三位是处于最右边的一个1。减去1后，第三位变成0， 它后面的两位0变成了1，而前面的1保持不变，因此得到的结果是1011.我们发现减1的结果是把最右边的一个1开始的所有位都取反了。这个时候如果我们再把原来的整数和减去1之后的结果做与运算，从原来整数最右边一个1那一位开始所有位都会变成0。如1100&1011=1000.也就是说，把一个整数减去1，再和原整数做与运算，会把该整数最右边一个1变成0.那么一个整数的二进制有多少个1，就可以进行多少次这样的操作。

### 代码实现

```java
public class Solution {
    public int NumberOf1(int n) {
        int count = 0;
        while (n != 0) {
            count++;
            n = n & (n - 1);
        } 
        return count;
    }
}
```

## 20、不用加减乘除做加法

### 题目描述

 > 写一个函数，求两个整数之和，要求在函数体内不得使用+、-、*、/四则运算符号。

### 分析

  >1. 使用异或和与运算符：题目要求不能使用四则运算符号，但是要求和，肯定要使用别的运算符，这里要想到两个运算符：异或和与。
  >2. 两个数异或：相当于每一位相加，并不考虑进位。
  >3. 两个数相与，并左移一位：相当于求得进位。
  >4. 然后将进位与异或的结果相加，直至进位为0，即得求和结果。

### 代码实现

```java
public class Solution {
    public int Add(int num1,int num2) {
        while(num2!=0){
            int sum = num1 ^ num2; // 每一位相加，不考虑进位
            num2 = (num1 & num2) << 1; // 计算进位
            num1 = sum;
        }
        return num1;
    }
}
```

## 21、整数中1出现的次数（从1到n整数中1出现的次数）

### 题目描述

 > 求出1~13的整数中1出现的次数,并算出100~1300的整数中1出现的次数？为此他特别数了一下1~13中包含1的数字有1、10、11、12、13因此共出现6次,但是对于后面问题他就没辙了。ACMer希望你们帮帮他,并把问题更加普遍化,可以很快的求出任意非负整数区间中1出现的次数（从1 到 n 中1出现的次数）。

### 分析

  >解法一：
  >>环累加从1到n中每个整数出现1的次数。判断每个整数是否出现1，利用对整数每次对10求余，判断整数的个位数字是不是1。如果这个数字大于10，除以10之后再判断个位数字是不是1。
  >>
  >
  >解法二：
  >
  >>

### 代码实现

解法一 :
```java
    public int NumberOf1Between1AndN_Solution(int n) {
        int count = 0;
        int i;
        while (n!=0) {
            i = n;
            while (i != 0) {
                if (i % 10 == 1) {
                    count++;
                }
                i /= 10;
            }
            n--;
        }
        return count;
    }
```


## 22、丑数

### 题目描述

 > 把只包含质因子2、3和5的数称作丑数（Ugly Number）。例如6、8都是丑数，但14不是，因为它包含质因子7。 习惯上我们把1当做是第一个丑数。求按从小到大的顺序的第N个丑数

### 分析

  > 
  >
  >这道题目自己是有思路的，丑数能够分解成2^x*3^y*5^z,
  >所以只需要把得到的丑数不断地乘以2、3、5之后并放入他们应该放置的位置即可，
  >而此题的难点就在于如何有序的放在合适的位置。
  >1乘以 （2、3、5）=2、3、5；2乘以（2、3、5）=4、6、10；3乘以（2、3、5）=6,9,15；5乘以（2、3、5）=10、15、25；
  >从这里我们可以看到如果不加策略地添加丑数是会有重复并且无序，
  >而在2*x，3*y，5*z中，如果x=y=z那么最小丑数一定是乘以2的，但关键是有可能存在x》y》z的情况，所以我们要维持三个指针来记录当前乘以2、乘以3、乘以5的最小值，然后当其被选为新的最小值后，要把相应的指针+1；因为这个指针会逐渐遍历整个数组，因此最终数组中的每一个值都会被乘以2、乘以3、乘以5，也就是实现了我们最开始的想法，只不过不是同时成乘以*2、*3、*5，而是在需要的时候乘以*2、*3、5.  

### 代码实现

```java


public class Solution {
    public int GetUglyNumber_Solution(int index) {
        if(index <= 0)return 0;
        int p2=0,p3=0,p5=0;//初始化三个指向三个潜在成为最小丑数的位置
        int[] result = new int[index];
        result[0] = 1;//
        for(int i=1; i < index; i++){
            result[i] = Math.min(result[p2]*2, Math.min(result[p3]*3, result[p5]*5));
            if(result[i] == result[p2]*2)p2++;//为了防止重复需要三个if都能够走到
            if(result[i] == result[p3]*3)p3++;//为了防止重复需要三个if都能够走到
            if(result[i] == result[p5]*5)p5++;//为了防止重复需要三个if都能够走到
 
 
        }
        return result[index-1];
    }
}
```

## 23、反转链表

### 题目描述

 > 输入一个链表，反转链表后，输出新链表的表头。

### 分析



  >用pre记录当前节点的前一个节点
  >
  >用next记录当前节点的后一个节点
  >
  >1. 当前节点a不为空，进入循环，先记录a的下一个节点位置next = b;再让a的指针指向pre
  >2. 移动pre和head的位置，正因为刚才记录了下一个节点的位置，所以该链表没有断，我们让head走向b的位置。
  >3. 当前节点为b不为空，先记录下一个节点的位置，让b指向pre的位置即a的位置，同时移动pre和head
  >4. 当前节点c不为空，记录下一个节点的位置，让c指向b，同时移动pre和head，此时head为空，跳出，返回pre。

### 代码实现

```java
    public ListNode ReverseList(ListNode head) {

        ListNode pre = null; // 当前节点的前一个节点
        ListNode next = null; // 当前节点的下一个节点
        while( head != null){
            next = head.next; // 记录当前节点的下一个节点位置；
            head.next = pre; // 让当前节点指向前一个节点位置，完成反转
            
            pre = head; // pre 往右走
            head = next;// 当前节点往右继续走
        }
        return pre;
    }
```



## 24、数值的整数次方

### 题目描述

 > 给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。
 >
 > 保证base和exponent不同时为0

### 分析

  >

### 代码实现

```java

```