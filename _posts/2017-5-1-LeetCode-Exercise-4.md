---
layout: post_layout
title: LeetCode刷题-4
time: 2017年5月1日 星期一
location: 北京
pulished: true
excerpt_separator: "**2.Reverse Vowels"
---
**LeetCode刷题(四)**　　**2017年5月1日**　　**天气：晴**  
**1.Reverse String(344)**  
Write a function that takes a string as input and returns the string reversed.  
Example:  
Given s = "hello", return "olleh".
```Java
public class Solution {
    public String reverseString(String s) {
        if(s.length() == 0 || s.length() == 1) return s;
        char[] arr = s.toCharArray();
        int l = 0;
        int r = arr.length - 1;
        while(l < r){
            char temp = arr[l];
            arr[l] = arr[r];
            arr[r] = temp;
            l++;
            r--;
        }
        return new String(arr);
    }
}
```

---
**2.Reverse Vowels of a String(345)**  
Write a function that takes a string as input and reverse only the vowels of a string.  
Example 1:  
Given s = "hello", return "holle".  
Example 2:  
Given s = "leetcode", return "leotcede".  
Note:  
The vowels does not include the letter "y".
```Java
public class Solution {
    public String reverseVowels(String s) {
        if(s.length() == 0 || s.length() == 1) return s;
        char[] arr = s.toCharArray();
        int l = 0;
        int r = arr.length - 1;
        while(l < r){
            //l < arr.length - 1判断数组右边界越界的情况
            while(arr[l] != 'a' && arr[l] != 'e' && arr[l] != 'i' && arr[l] != 'o' && arr[l] != 'u' &&
               arr[l] != 'A' && arr[l] != 'E' && arr[l] != 'I' && arr[l] != 'O' && arr[l] != 'U' && l < arr.length - 1){
                l++;
            }
            //r > 0判断数组左边界越界情况
            while(arr[r] != 'a' && arr[r] != 'e' && arr[r] != 'i' && arr[r] != 'o' && arr[r] != 'u' &&
               arr[r] != 'A' && arr[r] != 'E' && arr[r] != 'I' && arr[r] != 'O' && arr[r] != 'U' && r > 0){
                r--;
            }
            //这里要判断一下左右指针的大小，是否满足最外层的循环条件
            if(l < r){
                char temp = arr[l];
                arr[l] = arr[r];
                arr[r] = temp;
                l++;
                r--;
            }
        }
        return new String(arr);
    }
}
```

---
**3.Ransom Note(383)**  
Given an arbitrary ransom note string and another string containing letters from all the magazines, write a function that will return true if the ransom note can be constructed from the magazines ; otherwise, it will return false.  
Each letter in the magazine string can only be used once in your ransom note.  
Note:  
You may assume that both strings contain only lowercase letters.
```
canConstruct("a", "b") -> false
canConstruct("aa", "ab") -> false
canConstruct("aa", "aab") -> true
```
```Java
public class Solution {
    //构成ransomNote的字符，在magazine中都存在且个数不小于在ransomNote中的个数，则返回ture，说明ransomNote可以由magazine中的字符构成。
    public boolean canConstruct(String ransomNote, String magazine) {
        int[] charNums = new int[26];
        for(int i = 0;i < magazine.length();++i){
            charNums[magazine.charAt(i) - 'a']++;
        }
        for(int i = 0;i < ransomNote.length();++i){
            if(--charNums[ransomNote.charAt(i) - 'a'] < 0){
                return false;
            }
        }
        return true;
    }
}
```

---
**4. Number of Segments in a String(434)**  
Count the number of segments in a string, where a segment is defined to be a contiguous sequence of non-space characters.  
Please note that the string does not contain any non-printable characters.  
Example:
```
Input: "Hello, my name is John"
Output: 5
```
```Java
public class Solution {
    public int countSegments(String s) {
        String str = s.trim();
        //注意这里对trim()后的结果的判断
        //trim()后为空的话，再进行下面的split()操作，则字符数组中有一个元素，且该元素为空，应该单独判断这种情况
        if(str.length() == 0) return 0;
        //"\s"匹配任何空白字符，包括空格、制表符、换页符等等，“\s+”则表示匹配任意多个上面的字符
        else return str.split("\\s+").length;
    }
}
```

---
**5.Repeated Substring Pattern(459)**  
Given a non-empty string check if it can be constructed by taking a substring of it and appending multiple copies of the substring together. You may assume the given string consists of lowercase English letters only and its length will not exceed 10000.  
Example 1:
```
Input: "abab"
Output: True
Explanation: It's the substring "ab" twice.
```
Example 2:
```
Input: "aba"
Output: False
```
Example 3:
```
Input: "abcabcabcabc"
Output: True
Explanation: It's the substring "abc" four times. (And the substring "abcabc" twice.)
```
```Java
public class Solution {
    public boolean repeatedSubstringPattern(String s) {
        if(s.length() == 0) return false;
        for(int i = 1;i <= s.length()/2;++i){
            if(s.length()%i == 0){
                String temp = s.substring(0,i);
                StringBuilder sb = new StringBuilder();
                for(int j = 0;j < s.length()/temp.length();++j){
                    sb.append(temp);
                }
                if(sb.toString().equals(s)){
                    return true;
                }
            }
        }
        return false;
    }
}
```

---
**6.Reverse String II(541)**  
Given a string and an integer k, you need to reverse the first k characters for every 2k characters counting from the start of the string. If there are less than k characters left, reverse all of them. If there are less than 2k but greater than or equal to k characters, then reverse the first k characters and left the other as original.  
Example:  
```
Input: s = "abcdefg", k = 2
Output: "bacdfeg"
```
Restrictions:  
1.The string consists of lower English letters only.  
2.Length of the given string and k will in the range [1, 10000]
```Java
public class Solution {
    public String reverseStr(String s, int k) {
        //注意边界情况的判断即可
        if(k >= s.length()){
            s = reverseWord(s,0,s.length()-1);
        }else{
            int l = 0;
            for(int i = 0;i <= s.length()/(2*k);++i){
                if((l+k-1) < s.length()){
                    s = reverseWord(s,l,l + k - 1);
                    l += 2*k;
                }else if(l < s.length() && (l+k-1) > s.length() - 1){
                    s = reverseWord(s,l,s.length()-1);
                }
            }
        }
        return s;
    }
    
    public String reverseWord(String s,int l,int r){
        char[] cs = s.toCharArray();
        while(l < r){
            char temp = cs[l];
            cs[l] = cs[r];
            cs[r] = temp;
            l++;
            r--;
        }
        return new String(cs);
    }
}
```

---
**7.Reverse Words in a String III(557)**  
Given a string, you need to reverse the order of characters in each word within a sentence while still preserving whitespace and initial word order.  
Example 1:
```
Input: "Let's take LeetCode contest"
Output: "s'teL ekat edoCteeL tsetnoc"
```
Note: In the string, each word is separated by single space and there will not be any extra space in the string.
```Java
public class Solution {
    public String reverseWords(String s) {
        char[] s1 = s.toCharArray();
        int i = 0;
        for(int j = 0; j < s1.length; j++){
            if(s1[j] == ' '){
                reverse(s1, i, j - 1);
                i = j + 1;
            }
        }
        reverse(s1, i, s1.length - 1);
        return new String(s1);
    }

    public void reverse(char[] s, int l, int r){
        while(l < r){
            char temp = s[l];
            s[l] = s[r];
            s[r] = temp;
            l++;
            r--;
        }
    }
}
```

---
**8. Student Attendance Record I(551)**  
You are given a string representing an attendance record for a student. The record only contains the following three characters:  
1.'A' : Absent.  
2.'L' : Late.  
3.'P' : Present.  
A student could be rewarded if his attendance record doesn't contain **more than one 'A' (absent)** or **more than two continuous 'L' (late)**.  
You need to return whether the student could be rewarded according to his attendance record.  
Example 1:
```
Input: "PPALLP"
Output: True
```
Example 2:
```
Input: "PPALLL"
Output: False
```
```Java
public class Solution {
    public boolean checkRecord(String s) {
        Stack<Character> ss = new Stack<Character>();
        char[] cs = s.toCharArray();
        for(int i = 0;i < cs.length;++i){
            ss.push(cs[i]);
        }
        int numA = 0;
        while(!ss.isEmpty()){
            if(ss.peek() == 'P'){
                ss.pop();
            }else if(ss.peek() == 'A'){
                ss.pop();
                numA++;
            }else if(ss.peek() == 'L'){
                ss.pop();
                if(!ss.isEmpty() && ss.peek() == 'L'){
                    ss.pop();
                    if(!ss.isEmpty() && ss.peek() == 'L'){
                        return false;
                    }
                }
            }
        }
        if(numA >= 2){
            return false;
        }else{
            return true;
        }
    }
}
```

---
**9.Longest Common Prefix(14)**  
Write a function to find the longest common prefix string amongst an array of strings.  
```Java
public class Solution {
    public String longestCommonPrefix(String[] strs) {
        if(strs.length == 0) return "";
        String prifex = strs[0];
        for(int i = 0;i < strs.length;++i){
            while(strs[i].indexOf(prifex) != 0){
                prifex = prifex.substring(0,prifex.length()-1);
                if(prifex.length() == 0) return "";
            }
        }
        return prifex;
    }
}
```

---
**10.Valid Parentheses(20)**  
Given a string containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid.  
The brackets must close in the correct order, "()" and "()[]{}" are all valid but "(]" and "([)]" are not.
```Java
public class Solution {
    public boolean isValid(String s) {
        Stack<Character> stack = new Stack<Character>();
        char[] cs = s.toCharArray();
        for(char c : cs){
            if(c == '['){
                stack.push(']');
            }else if(c == '('){
                stack.push(')');
            }else if(c == '{'){
                stack.push('}');
            }else if(stack.isEmpty() || stack.pop() != c){
                return false;
            }
        }
        return stack.isEmpty();
    }
}
```
