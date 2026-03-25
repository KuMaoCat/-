# 算法登天梯

[TOC]



## Day01——二分查找

[力扣题目链接](https://leetcode.cn/problems/binary-search/)

题目：给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target  ，写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1。

示例1：

```java
输入：nums = [-1,0,3,5,9,12], target = 9
输出：4
```

示例2：

```java
输入：nums = [-1,0,3,5,9,12], target = 2
输出：-1
```

提示：

- 你可以假设 nums 中的所有元素是不重复的。
- n 将在 [1, 10000]之间。
- nums 的每个元素都将在 [-9999, 9999]之间。

> 思路：
>
> **这道题目的前提是数组为有序数组**，同时题目还强调**数组中无重复元素**，因为一旦有重复元素，使用二分查找法返回的元素下标可能不是唯一的，这些都是使用二分法的前提条件，当大家看到题目描述满足如上条件的时候，可要想一想是不是可以用二分法了。
>
> 二分查找涉及的很多的边界条件，逻辑比较简单，但就是写不好。例如到底是 `while(left < right)` 还是 `while(left <= right)`，到底是`right = middle`呢，还是要`right = middle - 1`呢？
>
> 大家写二分法经常写乱，主要是因为**对区间的定义没有想清楚，区间的定义就是不变量**。要在二分查找的过程中，保持不变量，就是在while寻找中每一次边界的处理都要坚持根据区间的定义来操作，这就是**循环不变量**规则。
>
> 写二分法，区间的定义一般为两种，左闭右闭即[left, right]，或者左闭右开即[left, right)。

作答：

```java
class Solution {
    public int search(int[] nums, int target) {
        int len = nums.length;
        if (len == 0) {
            return -1;
        } else {
            int len_half = len / 2;
            if (target < nums[len_half]) {
                int[] arr = new int[len_half];
                System.arraycopy(nums, 0, arr, 0, len_half);
                return search(arr, target);
            } else if (target > nums[len_half]) {
                int[] arr = new int[len - len_half - 1];
                System.arraycopy(nums, len_half + 1, arr, 0, len - len_half - 1);
                int result = search(arr, target);
                return result == -1 ? -1 : result + (len_half + 1);
            } else {
                return len_half;
            }
        }
    }
}
```

参考答案：

```java
class Solution {
    public int search(int[] nums, int target) {
        // 避免当 target 小于nums[0] nums[nums.length - 1]时多次循环运算
        if (target < nums[0] || target > nums[nums.length - 1]) {
            return -1;
        }
        int left = 0, right = nums.length - 1;
        while (left <= right) {
            int mid = left + ((right - left) >> 1);
            if (nums[mid] == target) {
                return mid;
            }
            else if (nums[mid] < target) {
                left = mid + 1;
            }
            else { // nums[mid] > target
                right = mid - 1;
            }
        }
        // 未找到目标值
        return -1;
    }
}
```

相关题目：

- [35.搜索插入位置](https://programmercarl.com/0035.搜索插入位置.html)
- [34.在排序数组中查找元素的第一个和最后一个位置](https://programmercarl.com/0034.在排序数组中查找元素的第一个和最后一个位置.html)
- [69.x 的平方根](https://leetcode.cn/problems/sqrtx/)
- [367.有效的完全平方数](https://leetcode.cn/problems/valid-perfect-square/)



题目：给你一个非负整数 `x` ，计算并返回 `x` 的 算术平方根 。由于返回类型是整数，结果只保留 整部分，小数部分将被 舍去 。注意：不允许使用任何内置指数函数和算符，例如 `pow(x, 0.5)` 或者`x**0.5` 。

示例1：

```java
输入：x = 4
输出：2
```

示例2：

```java
输入：x = 8
输出：2
```

提示：

- `0 <= x <= 2^31 - 1`

> 思路：
>
> - 本质：找到满足`k * k <= x`的最大整数k
> - 搜索范围：`[1, x]`
>   - `mid * mid <= x`   ->   可能溢出
>   - `mid <= x / mid`   ->   安全
> - 区间收缩：
>   - `mid <= x / mid`   ->   答案可能在右侧   ->   `left = mid +1`
>   - `mid > x / mid`   ->   答案可能在左侧   ->   `right = mid -1`
> - 循环结束返回`right`

参考答案：

```java
class Solution {
    public int mySqrt(int x) {
        if (x == 0) return 0;
        int left = 1;
        int right = x;
        if (x >= 4) right = x >> 1;
        while(left <= right){
            int mid = left + ((right - left) >> 1);
            if (mid <= x / mid){
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        return right;
    }
}
```



## Day02——双指针、滑动窗口

### 双指针

[力扣题目链接](https://leetcode.cn/problems/remove-element/)

题目：给你一个数组 nums 和一个值 val，你需要 原地 移除所有数值等于 val 的元素，并返回移除后数组的新长度。

- 不要使用额外的数组空间，你必须仅使用 O(1) 额外空间并**原地**修改输入数组。
- 元素的顺序可以改变。
- 你不需要考虑数组中超出新长度后面的元素。

示例1：

```java
输入：nums = [3,2,2,3], val = 3
输出：2, nums = [2,2,_,_]
解释：你的函数应该返回 k = 2, 并且 nums 中的前两个元素均为 2。
你在返回的 k 个元素之外留下了什么并不重要（因此它们并不计入评测）。
```

示例2：

```java
输入：nums = [0,1,2,2,3,0,4,2], val = 2
输出：5, nums = [0,1,4,0,3,_,_,_]
解释：你的函数应该返回 k = 5，并且 nums 中的前五个元素为 0,0,1,3,4。
注意这五个元素可以任意顺序返回。
你在返回的 k 个元素之外留下了什么并不重要（因此它们并不计入评测）。
```

> 思路：
>
> 双指针法（快慢指针法）： **通过一个快指针和慢指针在一个for循环下完成两个for循环的工作。**
>
> 定义快慢指针
>
> - 快指针：寻找新数组的元素 ，新数组就是不含有目标元素的数组
> - 慢指针：指向更新 新数组下标的位置
>
> ![](image\算法登天梯\27.移除元素-双指针法.gif)

参考答案：

```java
class Solution {
    public int removeElement(int[] nums, int val) {
       int slow = 0;
       for(int fast = 0; fast < nums.length; fast++) {
            if(nums[fast] != val){
                nums[slow] = nums[fast];
                slow++;
            }
       }
       return slow;
    }
}
```



### 滑动窗口



题目：给定一个含有 n 个正整数的数组和一个正整数 s ，找出该数组中满足其和 ≥ s 的长度最小的 连续 子数组，并返回其长度。如果不存在符合条件的子数组，返回 0。

示例1：

```java
输入：target = 7, nums = [2,3,1,2,4,3]
输出：2
解释：子数组 [4,3] 是该条件下的长度最小的子数组。
```

示例2：

```java
输入：target = 4, nums = [1,4,4]
输出：1
```

示例3：

```java
输入：target = 11, nums = [1,1,1,1,1,1,1,1]
输出：0
```

提示：

- `1 <= target <= 109`
- `1 <= nums.length <= 105`
- `1 <= nums[i] <= 104`

> **滑动窗口**
>
> 所谓滑动窗口，**就是不断的调节子序列的起始位置和终止位置，从而得出我们想要的结果**。
>
> 在暴力解法中，是一个for循环滑动窗口的起始位置，一个for循环为滑动窗口的终止位置，用两个for循环 完成了一个不断搜索区间的过程。
>
> 那么滑动窗口如何用一个for循环来完成这个操作呢。
>
> 首先要思考 如果用一个for循环，那么应该表示 滑动窗口的起始位置，还是终止位置。
>
> 如果只用一个for循环来表示 滑动窗口的起始位置，那么如何遍历剩下的终止位置？
>
> 此时难免再次陷入 暴力解法的怪圈。
>
> 所以 只用一个for循环，那么这个循环的索引，一定是表示 滑动窗口的终止位置。
>
> ![](image\算法登天梯\209.长度最小的子数组.gif)

参考答案：

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int left = 0;
        int sum = 0;
        int result = Integer.MAX_VALUE;
        for(int right = 0; right < nums.length; right++){
            sum += nums[right];
            while(sum >= target){
                result = Math.min(result, right - left +1);
                sum -= nums[left++];
            }
        }
        return result == Integer.MAX_VALUE ? 0 : result;
    }
}
```
