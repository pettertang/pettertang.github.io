---
layout: post_layout
title: TD-Summary for week 7
time: 2017年7月20日 星期四
location: 北京
pulished: true
excerpt_separator: "**2**.LeetCode(11)"
---
## 第7周总结

**1**.Mysql中的表增加删除字段  
1.1向表中增加一个字段  
alter table user add COLUMN new1 VARCHAR(20) DEFAULT NULL; //增加一个字段，默认为空  
alter table user add COLUMN new2 VARCHAR(20) NOT NULL;  //增加一个字段，默认不能为空   
user为表名，new1和new2为添加的字段名。  
在上面的语句中，我们还可以加上其他的一些条件，比如下面这个例子：  
alter table `user_movement_log`   
Add column GatewayId int not null default 0 AFTER `Regionid`    
说明：上面这个语句向“user_movement_log”表中添加了一个叫“GatewayId”的字段，为int类型，不为空，默认初始值为0，在“Regionid”这个字段后面添加。  
1.2删除一个字段  
alter table user DROP COLUMN new2; 　 //删除一个字段  
1.3修改一个字段  
alter table user MODIFY new1 VARCHAR(10); 　//修改一个字段的类型  
alter table user CHANGE new1 new4 int;　 //修改一个字段的名称，此时一定要重新指定该字段的类型  
以上只是一些最基本的操作，这些操作都可以根据实际需求来添加限定条件，当然还有很多其他的操作，这里都不展开叙述了。

---
**2**.LeetCode(11)  
Given n non-negative integers a1, a2, ..., an, where each represents a point at coordinate (i, ai). n vertical lines are drawn such that the two endpoints of line i is at (i, ai) and (i, 0). Find two lines, which together with x-axis forms a container, such that the container contains the most water.  
Note: You may not slant the container and n is at least 2.  
```Java
public class Solution {
    public int maxArea(int[] height) {
        int maxarea = 0;
        int l = 0;
        int r = height.length - 1;
        while (l < r) {
            maxarea = Math.max(maxarea, Math.min(height[l],height[r])*(r - l));
            if (height[l] < height[r])
                l++;
            else
                r--;
        }
        return maxarea;
    }
}
```

---
**3**.LeetCode(15)  
Given an array S of n integers, are there elements a, b, c in S such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero.  
Note: The solution set must not contain duplicate triplets.
```
For example, given array S = [-1, 0, 1, 2, -1, -4],
A solution set is:
[
  [-1, 0, 1],
  [-1, -1, 2]
]
```
```Java
public class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> result = new ArrayList<List<Integer>>();
        int length = nums.length;
        if (length < 3) {
            return result;
        }
        Arrays.sort(nums);
        for (int i = 0; i< length - 2;++i) {
            if (nums[i] > 0) break;
            if (i > 0 && nums[i] == nums[i - 1])
                continue;
            int left = i + 1;
            int right = length - 1;
            while(left < right) {
                int sum = nums[i] + nums[left] + nums[right];
                if (sum == 0) {
                    result.add(Arrays.asList(nums[i],nums[left],nums[right]));
                left++;
                right--;
                while(left < right && nums[left] == nums[left - 1])
                    left++;
                while(left < right && nums[right] == nums[right + 1])
                    right--;
                }
                else if (sum > 0)
                    right--;
                else
                    left++;                 
            }
        }
        return result;
    }
}
```

---
**4**.关于Sql语句查询返回查询条件的结果的总行数。  
SELECT COUNT(*) from `table` WHERE ......;   　　　查出符合条件的记录总数  
SELECT * FROM `table` WHERE ...... limit M,N;　　 查询当页要显示的数据   

---
**5**.Leetcode(16)  
Given an array S of n integers, find three integers in S such that the sum is closest to a given number, target. Return the sum of the three integers. You may assume that each input would have exactly one solution.
```
For example, given array S = {-1 2 1 -4}, and target = 1.
The sum that is closest to the target is 2. (-1 + 2 + 1 = 2).
```
```Java
public class Solution {
    public int threeSumClosest(int[] nums, int target) {
        int length = nums.length;
        int min = Integer.MAX_VALUE;
        Arrays.sort(nums);
        for (int i = 0;i < length - 2;++i) {
            int l = i + 1;
            int r = length - 1;
            while (l < r) {
                int sum = nums[i] + nums[l] + nums[r];
                if (Math.abs(sum - target) < Math.abs(min))
                    min = sum - target;
                if (sum == target)
                    return target;
                else if (sum > target)
                    r--;
                else
                    l++;
            }
        }
        return min+target;
    }
}
```

---
**6**.Leetcode(268)  
Given an array containing n distinct numbers taken from 0, 1, 2, ..., n, find the one that is missing from the array.
For example,  
Given nums = [0, 1, 3] return 2.  
Your algorithm should run in linear runtime complexity. Could you implement it using only constant extra space complexity?
```Java
public class Solution {
    public int missingNumber(int[] nums) {
        int result = nums.length;
        for (int i = 0;i < nums.length;++i) {
            result ^= i;
            result ^= nums[i];
        }
        return result;
    }
}
```





