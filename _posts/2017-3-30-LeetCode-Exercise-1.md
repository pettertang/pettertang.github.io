---
layout: post_layout
title: LeetCode刷题-1
time: 2017年3月30日 星期四
location: 北京
pulished: true
excerpt_separator: "**2.Convert Sorted Array to Binary Search Tree"
---
**Leetcode刷题（一）**　　　**2017年3月30日** 　　　**天气：多云**   
**1.判断二叉平衡树。(110)**     
For this problem, a height-balanced binary tree is defined as a binary tree in which the depth of the two subtrees of every node never differ by more than 1.
```Java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
public class Solution {
    public boolean isBalanced(TreeNode root) {
        if(root == null){
            return true;
        }
        int leftDepth = depth(root.left);
        int rightDepth = depth(root.right);
        return Math.abs(leftDepth - rightDepth)<=1 && isBalanced(root.left) && isBalanced(root.right);
    }
    
    public int depth(TreeNode root){
        if(root == null){
            return 0;
        }
        return Math.max(depth(root.left),depth(root.right))+1;
    }
}
```
----
**2.Convert Sorted Array to Binary Search Tree(108)**  
Given an array where elements are sorted in ascending order, convert it to a height balanced BST.
```Java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
public class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        if(nums.length == 0){
            return null;
        }
        TreeNode head = createTree(nums,0,nums.length-1);
        return head;
    }
    
    public TreeNode createTree(int[] nums,int low,int high){
        if(low > high){
            return null;
        }
        int mid = (low + high)/2;
        TreeNode node = new TreeNode(nums[mid]);
        node.left = createTree(nums,low,mid-1);
        node.right = createTree(nums,mid+1,high);
        return node;
    }
}
```
---
**3.Minimum Depth of Binary Tree(111)**     
Given a binary tree, find its minimum depth.
The minimum depth is the number of nodes along the shortest path from the root node down to the nearest leaf node.
```Java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
public class Solution {
    public int minDepth(TreeNode root) {
        if(root == null){
            return 0;
        }
        int leftDepth = minDepth(root.left);
        int rightDepth = minDepth(root.right);
        int depth = Math.min(leftDepth,rightDepth);
        //若只存在左子树或右子，则取子树不为空的这边的最小深度
        return 1+(depth > 0 ? depth : Math.max(leftDepth,rightDepth));
    }
}
```
---
**4.Path Sum(112) (只需要判断是否存在，注意和下面题目5的区别  )**    
Given a binary tree and a sum, determine if the tree has a root-to-leaf path such that adding up all the values along the path equals the given sum.
```Java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
public class Solution {
    public boolean hasPathSum(TreeNode root, int sum) {
        if(root == null){
            return false;
        }
        if(root.left == null && root.right == null){
            return root.val == sum;
        }
        return hasPathSum(root.left,sum - root.val) || hasPathSum(root.right,sum - root.val);
    }
}
```
---
**5.Path Sum II (113)**       
Given a binary tree and a sum, find all root-to-leaf paths where each path's sum equals the given sum.(需要输出所有满足的路径集合)        
Java代码1：
```Java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
public class Solution {
    private List<List<Integer>> result = new ArrayList<List<Integer>>();
    public List<List<Integer>> pathSum(TreeNode root, int sum) {
        if(root == null){
            return result;
        }
        Stack<Integer> path = new Stack<Integer>();
        pathSumHelper(root,sum,path);
        return result;
    }
    
    public void pathSumHelper(TreeNode root,int sum,Stack<Integer> path){
        path.push(root.val);
        if(root.left == null && root.right == null){
            if(sum == root.val){
                result.add(new ArrayList<Integer>(path));
            }
        }
        if(root.left != null) pathSumHelper(root.left,sum - root.val,path);
        if(root.right != null) pathSumHelper(root.right,sum - root.val,path);
        path.pop();
    }
}
```  
Java代码2：   
```Java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
public class Solution {
    
    public List<List<Integer>> pathSum(TreeNode root, int sum){
        List<List<Integer>> result  = new LinkedList<List<Integer>>();
        List<Integer> currentResult  = new LinkedList<Integer>();
        pathSum(root,sum,currentResult,result);
        return result;
    }
    
    public void pathSum(TreeNode root, int sum, List<Integer> currentResult,List<List<Integer>> result) {
        if (root == null){
            return;
        }
        currentResult.add(new Integer(root.val));
        if (root.left == null && root.right == null && sum == root.val) {
            result.add(new LinkedList(currentResult));
            currentResult.remove(currentResult.size() - 1);//don't forget to remove the last integer
            return;
            
        } else {
            pathSum(root.left, sum - root.val, currentResult, result);
            pathSum(root.right, sum - root.val, currentResult, result);
            
        }
        currentResult.remove(currentResult.size() - 1);
    }
}
```
Python代码：
```Python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def pathSum(self, root, sum):
        """
        :type root: TreeNode
        :type sum: int
        :rtype: List[List[int]]
        """
         if not root: return []
    if root.left == None and root.right == None:
        if sum == root.val: 
            return \[\[root.val\]\]
        else: 
            return []
    a = self.pathSum(root.left, sum - root.val) + \
        self.pathSum(root.right, sum - root.val)
    return [[root.val] + i for i in a]

```
---
**6.Pascal's Triangle   (LeetCode 118)**      
Given numRows, generate the first numRows of Pascal's triangle.
```Java
public class Solution {
    
    public List<List<Integer>> generate(int numRows) {
        List<List<Integer>> resultList = new ArrayList<List<Integer>>();
        if(numRows < 1){
            return resultList;
        }
        for(int i = 0;i < numRows;++i){
            List<Integer> rowList = new ArrayList<Integer>();
            for(int j = 0;j < i+1;++j){
                if(j == 0 || j == i){
                    rowList.add(1);
                }else{
                    rowList.add(resultList.get(i-1).get(j-1) + resultList.get(i-1).get(j));
                }
            }
            resultList.add(rowList);
        }
        return resultList;
    }
}
```
---
**7.Pascal's Triangle II(LeetCode 119)**     
Given an index k, return the kth row of the Pascal's triangle.     
For example, given k = 3,Return [1,3,3,1].
```Java
public class Solution {
    public List<Integer> getRow(int rowIndex) {
        List<Integer> resultList = new ArrayList<Integer>();
        if(rowIndex < 0){
            return resultList;
        }
        for(int i = 0;i <= rowIndex;++i){
            resultList.add(0,1);
            for(int j = 1;j < resultList.size() - 1;++j){
                resultList.set(j,resultList.get(j) + resultList.get(j+1));
            }
        }
        return resultList;
    }
}
```
---
**8.Best Time to Buy and Sell Stock (121)**     
Say you have an array for which the ith element is the price of a given stock on day i.If you were only permitted to complete at most one transaction (ie, buy one and sell one share of the stock), design an algorithm to find the maximum profit.           
**Example 1:**    
Input: [7, 1, 5, 3, 6, 4]        
Output: 5       
max. difference = 6-1 = 5 (not 7-1 = 6, as selling price needs to be larger than buying price)       
**Example 2:**      
Input: [7, 6, 4, 3, 1]       
Output: 0        
In this case, no transaction is done, i.e. max profit = 0. 
```Java
public class Solution {
    public int maxProfit(int[] prices) {
        if(prices.length == 0){
            return 0;
        }
        int maxProfit = 0;
        int currentMin = prices[0];
        for(int i = 0;i < prices.length;++i){
            if(prices[i] > currentMin){
                maxProfit = Math.max(maxProfit,prices[i] - currentMin);
            }else{
                currentMin = prices[i];
            }
        }
        return maxProfit;
    }
}
```