---
title: LeetCode：最长回文子串（Longest Palindromic Substring）的解法
date: 2019-01-12 14:32:23
tags:
- 算法
- LeetCode
categories:
- 算法
---

## 题目
给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为1000。

> Input: "babcd"
Output: "bab"

<!--more-->

本文将介绍三种解法，重点介绍时间复杂度最低的Manacher算法。

## O(n^3)算法

### 思路
- 从最长的子串开始，遍历所有该原字符串的子串；
- 每找出一个字符串，就判断该字符串是否为回文；
- 子串为回文时，则找到了最长的回文子串，因此结束；反之，继续遍历。

### 时间复杂度
- 遍历字符串子串：嵌套一个循环：O(n^2)；   
- 判断是否为回文：再次嵌套一个循环：O(n^3)。

### 代码

```java
public static String longestPalindrome(String s) {
    if(s.length() <= 1)
        return s;
    for(int i = s.length();i > 0; i--) {//子串长度
        for(int j = 0; j <= s.length() - i; j++) {
            String sub = s.substring(j , i + j);//子串位置
            int count = 0;//计数，用来判断是否对称
            for (int k = 0; k < sub.length() / 2; k++) {//左右对称判断
                if (sub.charAt(k) == sub.charAt(sub.length() - k - 1))
                    count++;
            }
            if (count == sub.length() / 2)
                return sub;
        }
    }
    return "";//表示字符串中无回文子串
}
```

## O(n^2)算法

### 思路

- 将子串分为单核和双核的情况，单核即指子串长度为奇数，双核则为偶数；
- 遍历每个除最后一个位置的字符index(字符位置)，单核：初始low = 初始high = index，low和high均不超过原字符串的下限和上限；判断low和high处的字符是否相等，相等则low++、high++（双核：初始high = 初始low+1 = index + 1）；
- 每次low与high处的字符相等时，都将当前最长的回文子串长度与high-low+1比较。后者大时，将最长的回文子串改为low与high之间的；
- 重复执行第二、三步，直至high-low+1 等于原字符串长度或者遍历到最后一个字符，取当前截取到的回文子串，该子串即为最长的回文子串。

### 时间复杂度

- 遍历字符：一层循环——O(n-1)；

- 找以当前字符为中心的最长回文子串：嵌套两个独立循环——O(2n*(n-1)) = O(n^2)


### 代码
```java
private static int maxLen = 0;

private static String sub = "";

public static String longestPalindrome(String s) {
    if(s.length() <= 1)
        return s;

    for(int i = 0;i < s.length()-1;i++){
        findLongestPalindrome(s,i,i);//单核回文
        findLongestPalindrome(s,i,i+1);//双核回文
    }
    return sub;
}

public static void findLongestPalindrome(String s,int low,int high) {
    while (low >= 0 && high <= s.length()-1){
        if(s.charAt(low) == s.charAt(high)){
            if(high - low + 1 > maxLen){
                maxLen = high - low + 1;
                sub = s.substring(low , high+1);
            }
            low --;//向两边扩散找当前字符为中心的最大回文子串
            high ++;
        } else
            break;
    }
}
```

## O(n)算法——Manacher算法

### 思路
Manacher算法是通过求解一个中心点，在距离这个点R长度以内都是关于这个点左右对称的，也就是说这个长度为2R的字符串是一个回文串，最后再比较大小，求出最大的长度2R及其中心点。整个过程只扫描整个字符串一遍。<br>
这里也可以看出，因为要求回文子串的中心点，这个中心点也是唯一的，所以它只能处理字符串是奇数位的情况。因此第一步就是把字符串长度变为奇数，这里使用一个非常巧妙的方式，把字符串中每个字符使用一个其它字符号包围起来，这里以`＃`号为例，可以想象一下，要把每个字符都使用#号包裹，那么需要的#号总是要比原来的字符串长度多一位，才能保证每个字符都能被插入到#与#中间，比如：

> a -> #a# 
abaf -> #a#b#a#f#

可以看出，不管原来的字符串长度是什么，奇数加偶数结果肯定为奇数，在这之后，就可以开始求最长回文子串的半径R了。

借助两个变量center、right分别记录回文子串对应的中心点和右端点

![你想输入的替代文字](LeetCode：最长回文子串（Longest-Palindromic-Substring）的解法/i-center-right.png)

可以直接看出，right就是`2*center-i`（也就是i关于center的对称点），既然是对称点，那么当端点right > i时，端点i需要进行计算回文子串R，但它的对称点有可能也进行过计算，所以可以无需从头开始匹配，因为这些点都包含在一个已经进行过匹配的父回文串中，所以这里可以直接取right-i和它的对称点回文子串半径长度较小的，用来保证绝对进行过计算的回文子串的部分;<br>
反之，就只能从1个长度开始匹配了，就是下面的这行代码:

```
r[i] = right > i ? (Math.min(r[2*center-i], right-i)) : 1
```

这里借助一个辅助的数组r[]来记录回文子串的半径R，r[i]表示的是以i为中心点的回文字符串的半径长度(初始情况下为1)，知道r[i]后，就可以继续把索引向左右两边扩充，也就是看i+r[i]与i-r[i]左右端点的位置所对应的字符是否相等，相等的话就把回文半径r[i]继续扩充，直到不相等为止。<br>
进行这一轮扩充后，观察之前的右端点right是否小于回文子串扩充后的右端点i+r[i],小于就直接更新右端点和中心点，不小于就说明当前回文子串还是在当前right端点的内部。

### 时间复杂度
只需进行一次遍历，时间复杂度为O(n)，具体的证明过程省略。

### 代码
```java
public static String longestPalindrome(String s) {
	if(s == null || s.length() < 1) {
		return s;
	}
	StringBuilder builder = new StringBuilder();
	// 防止左端点越界
	builder.append("&#");
	char[] c = s.toCharArray();
	for (char a : c) {
		builder.append(a).append("#");
	}
	String newStr = builder.toString();
	c = newStr.toCharArray();
	// 回文半径
	int[] r = new int[newStr.length()];
	// 回文子串最大右端点、中心点
	int right=0, center=0;
	// 最大回文半径、最大中心点
	int maxR=0, maxC=0;
	for (int i=1;i < c.length;i++) {
		// 以i为中心点的回文半径，可以重复利用以及匹配过对称点的半径
		r[i] = right > i ? (Math.min(r[2*center-i], right-i)) : 1;
		while (i+r[i]<c.length && c[i+r[i]]==c[i-r[i]]) {
			++r[i];
		}
		// 更新右端点和中心点
		if (right < i+r[i]) {
			right = i+r[i];
			center = i;
		}
		// 更新最大半径和最大中心点
		if (maxR < r[i]) {
			maxR = r[i];
			maxC = i;
		}
	}
    //计算在原字符串中的起始点
	int start = (maxC-maxR)/2;
	return s.substring(start, start+maxR-1);
}
```