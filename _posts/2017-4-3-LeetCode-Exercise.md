---
layout: post_layout
title: LeetCode刷题
time: 2017年4月3日 星期一
location: 北京
pulished: true
excerpt_separator: " However, you may not engage "
---

**Leetcode刷题（二）**　　　**2017年4月3日** 　　　**天气：多云**  
**1.Best Time to Buy and Sell Stock II.(122)**     
Say you have an array for which the ith element is the price of a given stock on day i.Design an algorithm to find the maximum profit. You may complete as many transactions as you like (ie, buy one and sell one share of the stock multiple times). However, you may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).      
```Java
public class Solution {
    public int maxProfit(int[] prices) {
        int totalProfit = 0;
        for(int i = 0;i < prices.length - 1;++i){
            if(prices[i + 1] > prices[i]){
                totalProfit += prices[i + 1] - prices[i];
            }
        }
        return totalProfit;
    }
}
```
---
**2.Valid Palindrome(125)**     
Given a string, determine if it is a palindrome, considering only alphanumeric characters and ignoring cases.  
**For example:**     
"A man, a plan, a canal: Panama" is a palindrome.      
"race a car" is not a palindrome.      <
Note:Have you consider that the string might be empty? This is a good question to ask during an interview.For the purpose of this problem, we define empty string as valid palindrome.     
```Java
public class Solution {
    public boolean isPalindrome(String s) {
        boolean flag = false;
        //对字符串进行处理，只保留英文字母和数字，并将所有英文字母转换成小写
        String str = s.replaceAll("[^a-z^A-Z^0-9]","").toLowerCase();
        if(str.length() == 0 || str.length() == 1){
            flag = true;
        }
        char[] c = str.toCharArray();
        for(int i = 0;i < c.length/2;++i){
            if(c[i] == c[c.length - 1 - i]){
                flag = true;
            }else{
                flag = false;
                break;
            }
        }
        return flag;
    }
}
```
---
**3.Single Number(136)**       
Given an array of integers, every element appears twice except for one. Find that single one.      
Note:Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?      
```Java
public class Solution {
    public int singleNumber(int[] nums) {
        int result = 0;
        for(int num : nums){
        //运用异或操作，即:0 ^ N = N,N ^ N = 0.
            result = result ^ num;
        }
        return result;
    }
}
```
---
**4.Linked List Cycle(141)**      
Given a linked list, determine if it has a cycle in it.Can you solve it without using extra space?        
```Java
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public boolean hasCycle(ListNode head) {
        if(head == null || head.next == null){
            return false;
        }
        ListNode oneStep = head;
        ListNode twoStep = head;
        //用两个指针来扫描,一个指针每次向后移动一步，另一个指针每次向后移动两步
        //如果存在环，则这两个指针会相遇;否则不会。
        while(twoStep.next != null && twoStep.next.next != null){
            oneStep = oneStep.next;
            twoStep = twoStep.next.next;
            if(oneStep == twoStep){
                return true;
            }
        }
        return false;
    }
}
```
---
**5.Min Stack(155)**      
Design a stack that supports push, pop, top, and retrieving the minimum element in constant time.  
push(x) -- Push element x onto stack.     
pop() -- Removes the element on top of the stack.    
top() -- Get the top element.    
getMin() -- Retrieve the minimum element in the stack.    
**Example: **    
MinStack minStack = new MinStack();      
minStack.push(-2);     
minStack.push(0);     
minStack.push(-3);     
minStack.getMin();   --> Returns -3.     
minStack.pop();      
minStack.top();      --> Returns 0.     
minStack.getMin();   --> Returns -2.    
```Java
public class MinStack {

    /** initialize your data structure here. */
    private int min = Integer.MAX_VALUE;
    Stack<Integer> stack = new Stack<Integer>();
    public MinStack() {
        
    }
    //1.若待入栈的值小于min，则我们可以将min值记为old_min，old_min = min。并往栈中插入old_min和x两个值，同时将min值改变为x，min = x。
    //2.若待入栈的值大于min，则这次入栈操作不会改变min值大小,直接将x入栈。
    public void push(int x) {
        if(x <= min){
            stack.push(min);
            min = x;
        }
        stack.push(x);
    }
    //1.在进行出栈操作时，若此时待出栈的栈顶元素为当前记录的min值:结合前面push入栈操作可知，此时栈顶元素的一下个栈值为之前入栈的old_min，该值为我们多插入栈中的一个值，需要将它也出栈。也就是要进行两次出栈操作。
    //2.若待出栈的元素不是当前记录的min值，则直接将出栈，只进行一次出栈操作。
    public void pop() {
        if(stack.pop() == min){
            min = stack.pop();
        }
    }
    
    public int top() {
        return stack.peek();
    }
    
    public int getMin() {
        return min;
    }
}
```
---
**6.Intersection of Two Linked Lists(160)**  
Write a program to find the node at which the intersection of two singly linked lists begins.   
For example, the following two linked lists:   
A:a1 → a2 → c1 → c2 → c3                 
B:b1 → b2 → b3 → c1 → c2 → c3   
begin to intersect at node c1.  
**Notes:**   
If the two linked lists have no intersection at all, return null.  
The linked lists must retain their original structure after the function returns.  
You may assume there are no cycles anywhere in the entire linked structure.  
Your code should preferably run in O(n) time and use only O(1) memory.  
```Java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        ListNode currentA = headA;
        ListNode currentB = headB;
        if(headA == null || headB == null){
            return null;
        }
        //用两次迭代，最差时间复杂度为O(A+B)，其中A，B分别为两个链表的长度。
        //第一次遍历比较时，当其中较短的那个链表遍历到尾部时，我们将指向当前链表尾部的指针指向另一个链表的头指针，开始第二次迭代。
        //所以最坏的情况就是两个链表没有交叉点的情况，我们相当于对长度为(A+B)的链表遍历了两次。
        while(currentA != currentB){
            if(currentA == null){
                currentA = headB;
            }else{
                currentA = currentA.next;
            }
            if(currentB == null){
                currentB = headA;
            }else{
                currentB = currentB.next;
            }
        }
        return currentA;
    }
}
```
---
**7.Two Sum II - Input array is sorted(167)**  
Given an array of integers that is already **sorted in ascending order**, find two numbers such that they add up to a specific target number.The function twoSum should return indices of the two numbers such that they add up to the target, where index1 must be less than index2. Please note that your returned answers (both index1 and index2) are not zero-based.  
You may assume that each input would have exactly one solution and you may not use the same element twice.  
**Example:**  
**Input:** numbers={2, 7, 11, 15}, target=9   
**Output:** index1=1, index2=2  
```Java
public class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int[] result = new int[2];
        if(numbers == null || numbers.length < 2){
            return result;
        }
        int low = 0;
        int high = numbers.length - 1;
        //数组已经排好序，所以可以通过设置头尾两个points来一前一后的向中间扫描
        //利用两层循环暴力搜索时间复杂度较高，可能会超时
        while(low < high){
            int currentResult = numbers[low] + numbers[high];
            if(currentResult == target){
                result[0] = low + 1;
                result[1] = high + 1;
                break;
            }else if( currentResult < target){
                low++;
            }else{
                high--;
            }
        }
        return result;
    }
}
```
---  
**8.Excel Sheet Column Title(168)**  
Given a positive integer, return its corresponding column title as appear in an Excel sheet.  
**For example:**   
1 -> A  
2 -> B  
3 -> C  
...  
26 -> Z  
27 -> AA  
28 -> AB   
```Java
方法一:
public class Solution {
    //本质上是将一个10进制数转换成26进制数，对26取余，一直除到0。
    //不过注意因为A对应1，而不是0，所以相当于26进制的数都整体减1，才能对应上从0开始的10进制数。 
    //注意我们在对str迭代赋值时，str应该写在"+"号右边。即每次求余得到的余数应该是向前插入(头插入)。  
    //我们也可以向后插入最后将余数逆序一下。(见方法二)
    public String convertToTitle(int n) {
        String str = "";
        while(n != 0){
            str = (char)((n - 1)%26 + 'A') + str;
            n = (n - 1)/26;
        }
        return str;
    }
}
方法二:
public class Solution {
    public String convertToTitle(int n) {
        StringBuilder sb = new StringBuilder();
        while(n != 0){
            sb.append((char)('A' + (n - 1) % 26));
            n = (n - 1) / 26;
        }
        return sb.reverse().toString();
    }
}   
```
---
**9.Excel Sheet Column Number(171)**  
Related to question **Excel Sheet Column Title**.  
Given a column title as appear in an Excel sheet, return its corresponding column number.  
**For example:**  
A -> 1  
B -> 2  
C -> 3  
...  
Z -> 26  
AA -> 27  
AB -> 28  
```Java
public class Solution {
    //本质上就是将26进制数转换为10进制数。从后往前第n位上的值乘以26^(n-1)。这里26进制数是从1开始的，即A代表1。
    public int titleToNumber(String s) {
        int num = 0;
        int pow = 1;
        for(int i = s.length() - 1; i >= 0 ; i--){
            num += (s.charAt(i) - 'A' + 1)*pow;
            pow *= 26;
        }
        return num;
    }
}
```
---
**10.Majority Element(169)**  
Given an array of size n, find the majority element. The majority element is the element that appears more than ⌊ n/2 ⌋ times.  
You may assume that the array is non-empty and the majority element always exist in the array.  
```Java
public class Solution {
    public int majorityElement(int[] nums) {
        int majorityElement = nums[0];
        int count = 1;
        //注意到majorityElement的数量超过了数组长度的一半这个特点。  
        //所以我们用count来记录相同元素的个数，遍历完一次数组元素，最后count大于1所对应的nums[i]一定是符合题意的majorityElement。
        for(int i = 1;i < nums.length;++i){
            if(nums[i] == majorityElement){
                count++;
            }else if(count == 0){
                count++;
                majorityElement = nums[i];
            }else{
                count--;
            }
        }
        return majorityElement;
    }
}
```