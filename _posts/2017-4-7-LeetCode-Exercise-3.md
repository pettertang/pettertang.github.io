---
layout: post_layout
title: LeetCode刷题-3
time: 2017年4月10日 星期一
location: 北京
pulished: true
excerpt_separator: "there are at least 3 different"
---
###LeetCode刷题(三)　　2017年4月10日　　天气：阴 转多云 
####1.Factorial Trailing Zeroes(172)  
Given an integer n, return the number of trailing zeroes in n!.  
**Note:**Your solution should be in logarithmic time complexity.  
```Java
public class Solution {
    //求n的阶乘中尾部0的个数
    //n! = n*(n-1)*(n-2)...1，即求其中能整除5的个数
    public int trailingZeroes(int n) {
        int count = 0;
        while(n > 0){
            count += n/5;
            n = n/5;
        }
        return count;
    }
}
```  
####2.Rotate Array(189)  
Rotate an array of n elements to the right by k steps.  
For example, with n = 7 and k = 3, the array ``[1,2,3,4,5,6,7] ``is rotated to ``[5,6,7,1,2,3,4]``.  
**Note:**  
Try to come up as many solutions as you can, there are at least 3 different ways to solve this problem.  
```Java
方法一：
public class Solution {
    public void rotate(int[] nums, int k) {
        //保证k为[0,n-1]内的合法数值
        k %= nums.length;
        //先旋转整个数组
        for(int i = 0,j = nums.length - 1;i < j;i++,j--){
            int temp = nums[i];
            nums[i] = nums[j];
            nums[j] = temp;
        }
        //再旋转前k个数组元素
        for(int i = 0,j = k - 1;i < j;i++,j--){
            int temp = nums[i];
            nums[i] = nums[j];
            nums[j] = temp;
        }
        //最后旋转后(nums.length-k)个数组元素
        for(int i = k,j = nums.length - 1;i < j;i++,j--){
            int temp = nums[i];
            nums[i] = nums[j];
            nums[j] = temp;
        }
    }
}
```
```Java
方法二：
public class Solution {
    public void rotate(int[] nums, int k) {
        if(nums.length <= 1){
            return;
        }
        k = k % nums.length;
        int[] temp = new int[k];
        for(int i = 0;i < k;++i){
            temp[i] = nums[nums.length - k + i];
        }
        for(int i = nums.length - k - 1;i >= 0;--i){
            nums[i + k] = nums[i];
        }
        for(int i = 0;i < k;++i){
            nums[i] = temp[i];
        }
    }
}
```
####3.Number of 1 Bits(191)  
Write a function that takes an unsigned integer and returns the number of ’1' bits it has (also known as the ``Hamming weight``).  
For example, the 32-bit integer ’11' has binary representation`` 00000000000000000000000000001011``, so the function should return 3.  
```Java
方法一：
public class Solution {
    // you need to treat n as an unsigned value
    public int hammingWeight(int n) {
        int count = 0;
        //n&(n-1)会清除掉n最低位的1
        while(n != 0){
            n = n&(n-1);
            count++;
        }
        return count;
    }
}
```
```Java
方法二：
public class Solution {
    // you need to treat n as an unsigned value
    public int hammingWeight(int n) {
        int count = 0;
        while(n != 0){
            if((n&1) == 1){
                count++;
            }
            n >>>= 1;   //右移一位，最高位补0
        }
        return count;
    }
}
```
####4.House Robber(198)  
You are a professional robber planning to rob houses along a street. Each house has a certain amount of money stashed, the only constraint stopping you from robbing each of them is that adjacent houses have security system connected and **it will automatically contact the police if two adjacent houses were broken into on the same night**.  
Given a list of non-negative integers representing the amount of money of each house, determine the maximum amount of money you can rob tonight **without alerting the police**.  
```Java
方法一：
public class Solution {
    //开辟一个数组来记录下搜索过程中的值，避免递归超时
    public static int[] result;
    public int rob(int[] nums) {
        result = new int[nums.length];
        for(int i = 0;i < nums.length;++i){
            result[i] = -1;
        }
        return robHelper(nums,nums.length - 1);
    }
    
    public int robHelper(int[] nums,int idx){
        if(idx < 0){
            return 0;
        }
        if(result[idx] >= 0){
            return result[idx];
        }
        //从最后面开始向前搜索
        //动态规划的思想，去除后向性，计划搜索
        result[idx] = Math.max(robHelper(nums,idx - 2) + nums[idx],robHelper(nums,idx - 1));
        return result[idx];
    }
}
```
```Java
方法二：
public class Solution {
    
    public int rob(int[] nums) {
        int[][] result = new int[nums.length + 1][2];
        //result[i][0]代表不偷第i家店，result[i][1]代表偷第i家店
        for(int i = 1;i <= nums.length;++i){
            result[i][0] = Math.max(result[i - 1][0],result[i - 1][1]);
            result[i][1] = nums[i - 1] + result[i - 1][0];
        }
        return Math.max(result[nums.length][0],result[nums.length][1]);
    }
}
```
####5.Happy Number(202)  
Write an algorithm to determine if a number is "happy".  
A happy number is a number defined by the following process: Starting with any positive integer, replace the number by the sum of the squares of its digits, and repeat the process until the number equals 1 (where it will stay), or it loops endlessly in a cycle which does not include 1. Those numbers for which this process ends in 1 are happy numbers.  
**Example**:19 is a happy number  
　　　　1x1 + 9x9 = 82  
　　　　8x8 + 2x2 = 68  
　　　　6x6 + 8x8 = 100  
　　　　1x1 + 0x0 + 0x0 = 1  
```Java
public class Solution {
    public boolean isHappy(int n) {
        int temp1 = n;
        int temp2 = n;
        while(temp1 > 1){
            temp1 = helper(temp1);
            if(temp1 == 1){
                return true;
            }
            temp2 = helper(helper(temp2));
            if(temp2 == 1){
                return true;
            }
            if(temp1 == temp2){
                return false;
            }
        }
        return true;
    }
    
    public int helper(int n){
        int result = 0;
        while(n > 0){
            result += (n%10)*(n%10);
            n /= 10;
        }
        return result;
    }
}
```

