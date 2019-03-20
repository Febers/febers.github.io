---
title: 《剑指offer》题目 Java 实现
date: 2019-03-18 10:42:29
tags:
- 算法
- Java
categories:
- 算法
---

### Problem1：单例模式实现

### Problem2：二维数组中的查找

在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

```java
    /**
     * 从二维数组的右上角开始选取与 key 比较的整数
     * column 的变化:arr[0].length - 1 ---> 0
     * row 的变化 0 ---> arr.length
     *
     * @param arr 
     * @param key 
     */
    public static boolean find(int[][] arr, int key) {
        int col = arr[0].length - 1;
        int row = 0;
        while (col >= 0 && row < arr.length) {
            if (arr[row][col] == key) {
                return true;
            } else if (arr[row][col] > key) {   //大于查找值，则往前推一列
                col--;
            } else {    //小于查找值，则往下推一行
                row++;
            }
        }
        return false;
    }
```



### Problem3：替换空格

请实现一个函数，将字符串的每个空格替换为"%20"。例如输入`We are happy`，则输出`We%20are%20happy`。

```java
    /**
     * 使用 StringBuilder
     * 
     * @param str
     * @return
     */
    public static String replace(String str) {
        if (str.isEmpty()) {
            return "";
        }
        StringBuilder builder = new StringBuilder();
        for (int i = 0; i < str.length(); i++) {
            if (str.charAt(i) == ' ') {
                builder.append("%20");
            } else {
                builder.append(str.charAt(i));
            }
        }
        return builder.toString();
    }
```



### Problem4：从尾到头打印链表

输入一个链表的头结点，按照从尾到头的顺序打印出每个节点的值

```java
    static class ListNode<T> {
        T value;
        ListNode next;

        public ListNode(T value) {
            this.value = value;
        }
    }

    /**
     * 使用栈实现
     *
     * @param headNode
     */
    public static void printListReverse(ListNode headNode) {
        Stack<ListNode> stack = new Stack<>();
        while (headNode != null) {
            stack.push(headNode);
            headNode = headNode.next;
        }
        while (!stack.empty()) {
            System.out.println(stack.pop().value + " ");
        }
    }

    public static void main(String[] args) {
        ListNode<Integer> node = new ListNode<>(1);
        node.next = new ListNode<>(2);
        node.next.next = new ListNode<>(3);
        printListReverse(node);
    }
```



### Problem5：重建二叉树

输入某二叉树的前序遍历和中序遍历结果，请重建出该二叉树。

假设输入的前序遍历和中序遍历的结果中都不包含重复的数字。

例如输入前序遍历序列： `{1, 2, 4, 7, 3, 5, 6, 8}`

中序遍历序列：`{4, 7, 2, 1, 5, 3, 8, 6}`

重建出所示二叉树并且输出它的头结点。

```java
                     1
                   /   \
                  2     3
                 /     / \
                4     5   6
                 \        /
                 7        8
```



#### 知识点补充

> 前序遍历：先访问根节点，再访问左子结点，最后访问右子结点；（根左右）
>
> 中序遍历：先访问左子结点，再访问根结点，最后访问右子结点；（左根右）
>
> 后序遍历：先访问左子结点，再访问右子结点，最后访问根结点；（左右根）

> 二叉搜索树：左子结点总是小于或等于根结点，而右子结点总是大于或等于根结点。
>
> 二叉树的特例是**堆**和**红黑树**。
>
> 堆分为最大堆和最小堆。在最大堆中根节点的值最大，在最小堆中根节点的值最小。
>
> 红黑树是把树中的结点定义为红、黑两种颜色，并通过规则确保从根结点到叶结点的最长路径的长度不超过最短路径的两倍。


--------------------- 
### Problem4：用两个栈实现队列

### Problem4：旋转数组的最小数字

### Problem4：斐波那契数列

### Problem4：二进制中1的个数

### Problem4：数值的整数次方

### Problem4：O（1）时间删除链表结点

### Problem4：调整数组中奇数位和偶数位的先后顺序

### Problem4：链表中倒数第K个结点

### Problem4：翻转链表

### Problem4：合并两个排序的链表

### Problem4：树的子结构判断

### Problem4：二叉树的镜像

### Problem4：顺时针打印矩阵

### Problem4：包含min函数的栈

### Problem4：栈的压入、弹出序列

### Problem4：从上往下打印二叉树

### Problem4：二叉搜索树的后序遍历

### Problem4：二叉树中和为某一值的路径

### Problem4：字符串的排列

### Problem4：数组中出现次数超过一半的数字

### Problem4：连续子数组的最大和

### Problem4：整数中1出现的次数

### Problem4：把数组排成最小的数

### Problem4：丑数

### Problem4：第一个只出现一次的字符

### Problem4：数组中的逆序对

### Problem4：两个链表的第一个公共节点

### Problem4：二叉树的深度&&平衡二叉树判断

### Problem4：数字在排序数组中出现的次数

### Problem4：数组中只出现一次的数字

### Problem4：和为S的两个数字 

### Problem4：和为S的连续正数序列

### Problem4：翻转单词的顺序

### Problem4：扑克牌的顺子

### Problem4：y圆圈中最后剩下的数字（约瑟夫环问题）

### Problem4：计算1+2+3+ ··· + n

### Problem4：不用加减乘除做加法

### Problem4：把字符串转换为整数

### Problem4：数组中重复的数字

### Problem4：构建乘积数组

### Problem4：正则表达式匹配

### Problem4：表示数值的字符串

### Problem4：字符流中第一个不重复的字符

### Problem4：链表中环的入口结点

### Problem4：删除链表中欧冠重复的节点

### Problem4：二叉树的下一个节点

### Problem4：把二叉树打印成多行

### Problem4：按之字形顺序打印二叉树

### Problem：序列化二叉树

### Problem：二叉搜索树的第K个节点

### Problem：滑动窗口的最大值

