---
title: leetcode-数组
date: 2020-11-09 22:56:55
tags: leetcode-数组
category: leetcode
summary: leetcode-数组
top: false
cover: true
author: 张文军
---
<center>更多内容请关注：</center>

![Java快速开发学习](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1586869254.png)

<center><a href="https://wjhub.gitee.io">锁清秋</a></center>

----


# leetcode-数组


> 概要：需要了解一下知识：
> - 排序:选择排序;插入排序;归并排序;快速排序
> - 查找:二分查找法
> - 数据结构:栈;队列;堆



## 一、二分法查找

```java

	private int sort(int key, int[] target, int left, int right) {
		if (left > right)
			return -1;
		int mid = (left + right) / 2;
		if (key < target[mid])
			return sort(key, target, left, mid - 1);
		if (key > target[mid])
			return sort(key, target, mid + 1, right);
		else
			return mid;
	}

```

## 2. 移动零


```text

给定一个数组 nums，编写一个函数将所有 0 移动到数组的末尾，同时保持非零元素的相对顺序。 

 示例: 

输入: [0,1,0,3,12]
输出: [1,3,12,0,0] 

 说明: 


 必须在原数组上操作，不能拷贝额外的数组。 
 尽量减少操作次数。 

 Related Topics 数组 双指针
 
```

```java
		public void moveZeroes(int[] nums) {
			/**
			 * 存放非零元素的前驱指针，
			 * 	即：
			 * 		在[0,pre)前闭后开区间存放的都是非零元素
			 * 		在[pre,...]前闭后开区间存放的都是零元素
			 */
			int pre = 0;
			/**
			 * 移动指针
			 */
			int cur = 0;
			while (cur < nums.length) {
				if (nums[cur] != 0) {
					if (pre != cur)//只有指针不相等的时候才进行交换，要不然每次相同位置也都进行交换
						//元素交换
						LeetCodeUtils.swap(nums, pre, cur);
					pre++;
				}
				cur++;
			}
		}

```

## 3. 移除元素

```text
给你一个数组 nums 和一个值 val，你需要 原地 移除所有数值等于 val 的元素，并返回移除后数组的新长度。 

 不要使用额外的数组空间，你必须仅使用 O(1) 额外空间并 原地 修改输入数组。 

 元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。 



 示例 1: 

 给定 nums = [3,2,2,3], val = 3,

函数应该返回新的长度 2, 并且 nums 中的前两个元素均为 2。

你不需要考虑数组中超出新长度后面的元素。


 示例 2: 

 给定 nums = [0,1,2,2,3,0,4,2], val = 2,

函数应该返回新的长度 5, 并且 nums 中的前五个元素为 0, 1, 3, 0, 4。

注意这五个元素可为任意顺序。

你不需要考虑数组中超出新长度后面的元素。




 说明: 

 为什么返回数值是整数，但输出的答案是数组呢? 

 请注意，输入数组是以「引用」方式传递的，这意味着在函数里修改输入数组对于调用者是可见的。 

 你可以想象内部操作如下: 

 // nums 是以“引用”方式传递的。也就是说，不对实参作任何拷贝
int len = removeElement(nums, val);

// 在函数里修改输入数组对于调用者是可见的。
// 根据你的函数返回的长度, 它会打印出数组中 该长度范围内 的所有元素。
for (int i = 0; i < len; i++) {
    print(nums[i]);
}

 Related Topics 数组 双指针

```

```java

    public int removeElement(int[] nums, int val) {
		int pre = 0;
		for (int i = 0; i < nums.length; i++) {
			if (nums[i]!=val)
				nums[pre++] = nums[i];
		}
		return pre;
    }

```




## 4. 删除排序数组中的重复项

```text

给定一个排序数组，你需要在 原地 删除重复出现的元素，使得每个元素只出现一次，返回移除后数组的新长度。 

 不要使用额外的数组空间，你必须在 原地 修改输入数组 并在使用 O(1) 额外空间的条件下完成。 



 示例 1: 

 给定数组 nums = [1,1,2], 

函数应该返回新的长度 2, 并且原数组 nums 的前两个元素被修改为 1, 2。 

你不需要考虑数组中超出新长度后面的元素。 

 示例 2: 

 给定 nums = [0,0,1,1,1,2,2,3,3,4],

函数应该返回新的长度 5, 并且原数组 nums 的前五个元素被修改为 0, 1, 2, 3, 4。

你不需要考虑数组中超出新长度后面的元素。




 说明: 

 为什么返回数值是整数，但输出的答案是数组呢? 

 请注意，输入数组是以「引用」方式传递的，这意味着在函数里修改输入数组对于调用者是可见的。 

 你可以想象内部操作如下: 

 // nums 是以“引用”方式传递的。也就是说，不对实参做任何拷贝
int len = removeDuplicates(nums);

// 在函数里修改输入数组对于调用者是可见的。
// 根据你的函数返回的长度, 它会打印出数组中该长度范围内的所有元素。
for (int i = 0; i < len; i++) {
    print(nums[i]);
}

 Related Topics 数组 双指针

```

```java

		public int removeDuplicates(int[] nums) {
			/**
			 * 前 [0,pre) 个全为不重复元素
			 */
			int pre = 0;
			for (int i = 0; i < nums.length; i++) {
				if (nums[pre] < nums[i])
					nums[++pre] = nums[i];
			}
			return pre + 1;
		}

```



## 5. 删除排序数组中的重复项 II

```text
给定一个增序排列数组 nums ，你需要在 原地 删除重复出现的元素，使得每个元素最多出现两次，返回移除后数组的新长度。 

 不要使用额外的数组空间，你必须在 原地 修改输入数组 并在使用 O(1) 额外空间的条件下完成。 



 说明： 

 为什么返回数值是整数，但输出的答案是数组呢？ 

 请注意，输入数组是以“引用”方式传递的，这意味着在函数里修改输入数组对于调用者是可见的。 

 你可以想象内部操作如下： 


// nums 是以“引用”方式传递的。也就是说，不对实参做任何拷贝
int len = removeDuplicates(nums);

// 在函数里修改输入数组对于调用者是可见的。
// 根据你的函数返回的长度, 它会打印出数组中该长度范围内的所有元素。
for (int i = 0; i < len; i++) {
    print(nums[i]);
} 



 示例 1： 


输入：nums = [1,1,1,2,2,3]
输出：5, nums = [1,1,2,2,3]
解释：函数应返回新长度 length = 5, 并且原数组的前五个元素被修改为 1, 1, 2, 2, 3 。 你不需要考虑数组中超出新长度后面的元素。


 示例 2： 


输入：nums = [0,0,1,1,1,1,2,3,3]
输出：7, nums = [0,0,1,1,2,3,3]
解释：函数应返回新长度 length = 7, 并且原数组的前五个元素被修改为 0, 0, 1, 1, 2, 3, 3 。 你不需要考虑数组中超出新长度后面
的元素。




 提示： 


 0 <= nums.length <= 3 * 104 
 -104 <= nums[i] <= 104 
 nums 按递增顺序排列 

 Related Topics 数组 双指针

```

```java

		public int removeDuplicates(int[] nums) {
			/**
			 * [0,pre) 范围是每个最多出现两次的元素
			 */
			int pre = 0;
			//标记是否为第一次相同
			Boolean isFirst = true;
			for (int i = 1; i < nums.length; i++) {
				/**
				 * 第一次相同，将isFirst设置为false
				 * 并放入 [0,pre)
				 */
				if (nums[pre] == nums[i] && isFirst) {
					nums[++pre] = nums[i];
					isFirst = false;
				} else if (nums[pre] < nums[i]) {//不同，就也放入 [0,pre)，并将标记 isFirst设置为true
					nums[++pre] = nums[i];
					isFirst = true;
				}
			}
			return pre + 1;
		}

```



## 6. 颜色分类

```text
给定一个包含红色、白色和蓝色，一共 n 个元素的数组，原地对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。 

 此题中，我们使用整数 0、 1 和 2 分别表示红色、白色和蓝色。 

 注意: 
不能使用代码库中的排序函数来解决这道题。 

 示例: 

 输入: [2,0,2,1,1,0]
 输出: [0,0,1,1,2,2] 

 进阶： 


 一个直观的解决方案是使用计数排序的两趟扫描算法。 
 首先，迭代计算出0、1 和 2 元素的个数，然后按照0、1、2的排序，重写当前数组。 
 你能想出一个仅使用常数空间的一趟扫描算法吗？ 

 Related Topics 排序 数组 双指针



```

```java

		public void sortColors(int[] nums) {
			/**
			 * [0,left)存放0
			 */
			int left = 0;
			/**
			 * (right,nums.length]存放2
			 */
			int right = nums.length - 1;
			for (int i = 0; i <= right; ) {
				//首先判断当前元素是否为2，如果是
				//就将当前元素交换到末尾
				if (nums[i] == 2) {
					swap(nums, i, right--);
				}

				/**
				 * 接下来的操作就类似于 “移动零”操作;只不过是将0移动到前面
				 * 	将0移动到前面，将1移动到后面
				 * 	 即：当前元素是 1 的时候向前移动
				 * 	 	当前元素是 0 的时候，将当前元素
				 */
				if (nums[i] == 1) {
					i++;
				} else if (nums[i] == 0) {
					swap(nums, left++, i++);
				}
			}
		}

```



## 7. 合并两个有序数组

```text
给你两个有序整数数组 nums1 和 nums2，请你将 nums2 合并到 nums1 中，使 nums1 成为一个有序数组。 



 说明： 


 初始化 nums1 和 nums2 的元素数量分别为 m 和 n 。 
 你可以假设 nums1 有足够的空间（空间大小大于或等于 m + n）来保存 nums2 中的元素。 




 示例： 


输入：
nums1 = [1,2,3,0,0,0], m = 3
nums2 = [2,5,6],       n = 3

输出：[1,2,2,3,5,6] 



 提示： 


 -10^9 <= nums1[i], nums2[i] <= 10^9 
 nums1.length == m + n 
 nums2.length == n 

 Related Topics 数组 双指针


```

```java

		public void merge(int[] nums1, int m, int[] nums2, int n) {
			//题目说nums1 有足够的空间（nums1.length == m + n）来保存 nums2 中的元素，相当于自带了一个空数组
			int l = m - 1;//第一个元素元素指针
			int r = n - 1;//第二个数组元素指针
			int i = m + n - 1;
			while (r >= 0 && l >= 0) {
				if (nums1[l] >= nums2[r]) {
					nums1[i] = nums1[l--];
				} else {
					nums1[i] = nums2[r--];
				}
				i--;
			}
			while (r >= 0) {
				nums1[i--] = nums2[r--];
			}
		}

```

## 8. 数组中的第K个最大元素

```text
在未排序的数组中找到第 k 个最大的元素。请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。 

 示例 1: 

 输入: [3,2,1,5,6,4] 和 k = 2
输出: 5


 示例 2: 

 输入: [3,2,3,1,2,4,5,5,6] 和 k = 4
输出: 4 

 说明: 

 你可以假设 k 总是有效的，且 1 ≤ k ≤ 数组的长度。 
 Related Topics 堆 分治算法


```

```java

		public int findKthLargest(int[] nums, int k) {
      //维护一个 k 大小的优先队列便可
			PriorityQueue<Integer> heap = new PriorityQueue<Integer>();
			for (int num : nums) {
				if (heap.size() < k)
					heap.add(num);
				else if (heap.peek() < num) {
					heap.poll();
					heap.add(num);
				}
			}
			return heap.peek();
		}

```


## 9. 两数之和 II - 输入有序数组

```text
给定一个已按照升序排列 的有序数组，找到两个数使得它们相加之和等于目标数。 

 函数应该返回这两个下标值 index1 和 index2，其中 index1 必须小于 index2。 

 说明: 


 返回的下标值（index1 和 index2）不是从零开始的。 
 你可以假设每个输入只对应唯一的答案，而且你不可以重复使用相同的元素。 


 示例: 

 输入: numbers = [2, 7, 11, 15], target = 9
输出: [1,2]
解释: 2 与 7 之和等于目标数 9 。因此 index1 = 1, index2 = 2 。 
 Related Topics 数组 双指针 二分查找

```

```java

		public int[] twoSum(int[] numbers, int target) {
			HashMap<Integer, Integer> map = new HashMap<>();
			for (int i = 0; i < numbers.length; i++) {
				if (map.containsKey(target - numbers[i]))
					return new int[]{map.get(target - numbers[i]) + 1, i + 1};
				else
					map.put(numbers[i], i);
			}
			return new int[]{};
		}

```


## 10. 验证回文串

```text
给定一个字符串，验证它是否是回文串，只考虑字母和数字字符，可以忽略字母的大小写。 

 说明：本题中，我们将空字符串定义为有效的回文串。 

 示例 1: 

 输入: "A man, a plan, a canal: Panama"
输出: true


 示例 2: 

 输入: "race a car"
输出: false

 Related Topics 双指针 字符串

```

```java



```


## 11. 反转字符串中的元音字母

```text
编写一个函数，以字符串作为输入，反转该字符串中的元音字母。



 示例 1：

 输入："hello"
输出："holle"


 示例 2：

 输入："leetcode"
输出："leotcede"



 提示：


 元音字母不包含字母 "y" 。

 Related Topics 双指针 字符串

```

```java

		public String reverseVowels(String s) {
			if (s.length() <= 0)
				return "";
			HashSet<Character> set = new HashSet<>(Arrays.asList('a', 'i', 'o', 'u', 'e', 'A', 'I', 'O', 'U', 'E'));
			char[] c = s.toCharArray();
			int l = 0, r = s.length() - 1;
			while (l < r) {
				while (l < r && !set.contains(c[l])) {
					l++;
				}
				while (l < r && !set.contains(c[r])) {
					r--;
				}
				if (l < r) {
					char temp = c[l];
					c[l++] = c[r];
					c[r--] = temp;
				}
			}
			return new String(c);
		}

```

## 

