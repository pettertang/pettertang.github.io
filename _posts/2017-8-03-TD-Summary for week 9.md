---
layout: post_layout
title: TD-Summary for week 9
time: 2017年8月3日 星期四
location: 北京
pulished: true
excerpt_separator: "The replacement must be in-place"
---

### 第9周总结

**1**.LeetCode(31)  
Next Permutation   
Implement next permutation, which rearranges numbers into the lexicographically next greater permutation of numbers.
If such arrangement is not possible, it must rearrange it as the lowest possible order (ie, sorted in ascending order).
The replacement must be in-place, do not allocate extra memory.
Here are some examples. Inputs are in the left-hand column and its corresponding outputs are in the right-hand column.  
1,2,3 → 1,3,2  
3,2,1 → 1,2,3  
1,1,5 → 1,5,1
```Java
public class Solution {
    public void nextPermutation(int[] nums) {
        int i = nums.length - 2;
        while (i >= 0 && nums[i + 1] <= nums[i]) {
            i--;
        }
        if (i >= 0) {
            int j = nums.length - 1;
            while (j >= 0 && nums[j] <= nums[i]) {
                j--;
            }
            swap(nums,i,j);
        }
        reveser(nums,i + 1);    
    }
    
    public void swap(int[] nums,int i,int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
    
    public void reveser(int[] nums,int i) { 
        int j = nums.length - 1;
        while (i < j) {
            swap(nums,i,j);
            i++;
            j--;
        }
    }
}
```

---
**2**.关于@PostConstruct注解  
该注解是作用于Servlet生命周期的注解，和它一起的还有一个@PreDestroy注解。  
被@PostConstruct修饰的方法会在服务器加载Servlet的时候运行，并且只会被服务器调用一次。**被@PostConstruct修饰的方法会在构造函数之后，init()方法之前运行**。  
被@PreConstruct修饰的方法会在服务器卸载Servlet的时候运行，并且只会被服务器调用一次。**被@PreConstruct修饰的方法会在destroy()方法之后运行，在Servlet被彻底卸载之前**。  
关于这两个注解说明如下：   
1)只有一个方法可以使用此注释进行注解；  
2)被注解方法不得有任何参数；  
3)被注解方法返回值为void；  
4)被注解方法不得抛出已检查异常；  
5)被注解方法需是非静态方法；  
6)此方法只会被执行一次。

---
**3**.关于@RestController和@Controller注解  
　　简单通俗的解释就是，两者的共同点都是用来表示spring某个类的是否可以接收HTTP请求。  
@Controller标识一个Spring类是Spring MVC controller处理器，@Controller类中的方法可以直接通过返回String跳转到jsp、ftl、html等模版页面。在方法上加@ResponseBody注解，也可以返回实体对象。  
@RestController类中的所有方法只能返回String、Object、Json等实体对象，不能跳转到模版页面。
@RestController相当于@ResponseBody + @Controller。  
示例如下：
```Java
@Controller  
@ResponseBody  
public class MyController { }  
  
@RestController  
public class MyRestController { } 
```
@RestController中的方法如果想跳转页面，则用ModelAndView进行封装。  
示例如下：
```Java
@RestController
public class UserController {

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public String toIndex(){
        ModelAndView mv = new ModelAndView("index");
      	return mv;    
    }
}
```
