---
title: Java 排序算法
date: 2019-03-15 18:36:47
mathjax: true
categories:
- Java
- 算法
---

## 引言
排序算法一直是程序员开发和面试的重点，本文将重点讲解几种常用的排序算法。<!--more-->

## 稳定性
对于一个数组`a {6,2,4,6,1}`，其中有两个值为 6，为 a[0] 和 a[3]，排序之后的结果有两种

|  1   |  2   |  4   |    6    |    6    |
| :--: | :--: | :--: | :-----: | :-----: |
| a[4] | a[1] | a[2] | <b>a[0] | <b>a[3] |
| a[4] | a[1] | a[2] | <b>a[3] | <B>a[0] |

如果排序结束之后，a[0] 可以保证一定在 a[3] 前面，即原有的顺序不变，则该算法属于稳定的排序算法
> 冒泡排序、基数排序、插入排序、归并排序、桶排序、二叉树排序等

否则，属于不稳定的排序算法
> 选择排序，希尔排序，堆排序，快速排序等

## 冒泡排序
```Java
/**
 * 冒泡排序，O(n^2)
 * 每一次内层循环中，两两比较，将较大的数放到后面
 * 
 * @param a 待排序数据
 */
public static void bubbleSort(int[] a) {
    int length = a.length;
    int i = 0;
    int temp;

    for ( ; i < length - 1; i++) {
        int j = 0;

        for ( ; j < length - 1 - i; j++) {
            if (a[j] > a[j+1]) {
                temp = a[j];
                a[j] = a[j+1];
                a[j+1] = temp;
            }
        }
    }
}
```

## 插入排序
插入排序可以分为直接插入排序，以及其变种——折半插入排序、希尔排序

### 直接插入排序
```Java
/**
 * 直接插入排序，O(n^2)
 * i从第一个元素开始，默认i前面的序列已经排好序
 * 取出i的下一个元素，从后往前比较，找到适合的位置就插入
 *
 * @param a 待排序序列
 */
public static void insertSort(int[] a) {
    int length = a.length;

    for (int i = 0; i < length; i++) {
        int temp = a[i];

        for (int j = i; j > 0; j--) {
            if (a[j] < a[j-1]) {
                a[j] = a[j-1];
                a[j-1] = temp;
            }
        }
    }
}
```

### 折半插入排序
```Java
/**
 * 折半插入排序，O(n^2)
 * 对直接插入排序算法进行了改进
 * 直接插入算法采用遍历一个有序序列的方式插入待排序元素，完全没必要
 * 折半插入排序则使用了折半查找/二分查找
 *
 * Arrays类的binarySearch()方法便是折半查找的实现
 *
 * @param a 待排序序列
 */
public static void binaryInsertSort(int[] a) {
    int length = a.length;

    for (int i = 0; i < length; i++) {
        int temp = a[i];
        int low = 0;
        int high = i-1;
        while (low <= high) {
            int middle = (low+high) / 2;
            if (a[middle] < temp) {
                low = middle+1;
            } else {
                high = middle-1;
            }
        }
        for(int j = i-1; j >= low; j--) {
             //元素后移，为插入temp做准备
            a[j+1] = a[j];
        }
        a[low] = temp;
    }
}
```
### 希尔排序
```Java
/**
 * 希尔排序，也称 递减增量排序，O(n*(logn)^2)
 * 对于n个元素的序列，假设增量为increment
 * 从第一个元素开始，每隔increment取一个元素组成一个子序列
 * 对每个子序列进行直接插入排序，increment /= 2
 * 重复上述过程，直至increment为1
 *
 * @param a 待排序序列
 */
public static void shellSort(int[] a) {
    int length = a.length;

    // increment为增量，每次减为原来的一半，直至为1
    for (int increment = length / 2; increment > 0; increment /= 2) {

        // 共increment个组，对每一组都执行直接插入排序
        for (int i = 0; i < increment; i++) {

            for (int j = i + increment; j < length; j += increment) {

                // 如果a[j] < a[j-increment]，则寻找a[j]位置，并将后面数据的位置都后移。
                if (a[j] < a[j - increment]) {

                    int temp = a[j];
                    int k = j - increment;
                    while (k >= 0 && a[k] > temp) {
                        a[k + increment] = a[k];
                        k -= increment;
                    }
                    a[k + increment] = temp;
                }
            }
        }
    }
}
```
## 桶排序（基数排序）
```Java
/**
 * 基数排序，也称桶排序，O(d(k+n))
 * 基本思想：将整数按位数切割成不同的数字，然后按每个位数分别比较。
 * 具体做法：将所有待比较数值统一为同样的数位长度，数位较短的数前面补零。然后，从最低位开始，依次进行一次排序。这样从最低位排序一直到最高位排序完成以后, 数列就变成一个有序序列。
 *
 * @param a 待排序数组
 * @param d 位数，如果最大数为9527，则d为10000，如果为7，则d为10
 */
public static void radixSort(int[] a,int d) {
    int n = 1;  //代表位数对应的数：1,10,100...
    int k = 0;  //保存每一位排序后的结果用于下一位的排序输入
    int length = a.length;

    int[][] bucket = new int[10][length];  //排序桶用于保存每次排序后的结果，这一位上排序结果相同的数字放在同一个桶里
    int[] order = new int[length];  //用于保存每个桶里有多少个数字

    while(n < d) {
        for(int num : a) { //将数组array里的每个数字放在相应的桶里
            int digit = (num/n)%10;
            bucket[digit][order[digit]] = num;
            order[digit]++;
        }
        int i = 0;
        for( ; i<length; i++) { //将前一个循环生成的桶里的数据覆盖到原数组中用于保存这一位的排序结果
            if(order[i] != 0) {  //这个桶里有数据，从上到下遍历这个桶并将数据保存到原数组中
                int j = 0;
                for( ; j < order[i]; j++) {
                    a[k] = bucket[i][j];
                    k++;
                }
            }
            order[i]=0;  //将桶里计数器置0，用于下一次位排序
        }
        n *= 10;
        k = 0;  //将k置0，用于下一轮保存位排序结果
    }
}
```

## 归并排序
```Java
/**
 * 归并排序，O(nlog(n))
 * 典型的分治算法，把一个序列的排序分成两个子序列的排序，对子序列重复以上操作直至序列长度为1
 * 然合并以上序列
 *
 * @param array
 * @param left
 * @param right
 */
public static void mergeSort(int[] array, int left, int right) {
   if (left < right) {
       int center = (left + right) / 2;
       mergeSort(array, left, center);
       mergeSort(array, center+1, right);
       merge(array, left, center, right);
   }
}

static void merge(int[] array, int low, int mid, int high) {
    int[] tempArray = new int[high - low + 1];
    int temp = 0;
    int i = low;
    int j = mid + 1;

    while (i <= mid && j <= high) {
        if (array[i] <= array[j]) {
            tempArray[temp++] = array[i++];
        } else {
            tempArray[temp++] = array[j++];
        }
    }

    while (i <= mid) {  //此时右边已到底
        tempArray[temp++] = array[i++];
    }
    while (j <= high) { //此时左边已到底
        tempArray[temp++] = array[j++];
    }
    //将新数组中的数 覆盖原数组low之后的数据
    for (int k = 0; k < tempArray.length; k++) {
        array[k+low] = tempArray[k];
    }
}
```

## 选择排序
```Java
/**
  * 选择排序，O(n^2)
  * 遍历整个序列，将最小的数放在最前面
  * 遍历剩下的序列，将最小的数放在最前面，重复上述过程
  *
  * @param a 待排序序列
  */
 public static void selectSort(int[] a){
     int length = a.length;
     int i = 0;

     for( ; i < length; i++){  //外层循环
         int temp = a[i];
         int position = i;
         int j = i+1;

         for( ; j < length; j++) {  //往后遍历，找到最小的值以及其位置
             if(a[j] < temp) {
                 temp = a[j];
                 position = j;
             }
         }
         a[position]=a[i];  //进行交换
         a[i] = temp;
     }
 
```

## 快速排序
```Java
/**
 * 快速排序，O(nlog(n))
 * 对冒泡排序的改进，选取一个记录作为基准，经过一趟排序后，将整段序列分成两部分
 * 前半部分小于基准值，后半部分大于基准值，然后递归前、后两部分，继续排序
 *
 * @param a 待排序序列
 * @param start 序列开始值
 * @param end 序列结束值
 */
public static void quickSort(int[] a, int start, int end) {
    int baseNum = a[start];
    int temp;
    int left = start;
    int right = end;
    do {
        while (a[left] < baseNum && left < end) {
            left++;
        }
        while (a[right] > baseNum && right > start) {
            right--;
        }

        if (left <= right) {  //左边出现大于基准值或者右边出现小于基准值，且left<=right
            temp = a[left];
            a[left] = a[right];
            a[right] = temp;
            left++; right--;
        }
    } while (left <= right);

    if (start < right) {
        quickSort(a, start, right);
    }
    if (end > left) {
        quickSort(a, left, end);
    }
}
```

## 二叉树排序
通过二叉树的中序遍历实现。如果二叉排序树是平衡的，则时间复杂度为 $O(\log_2 n)$
近似于折半查找。

如果二叉排序树完全不平衡，则时间复杂度为`O(n)`
```Java
public class BinarySortTree {
    static class Node {
        private Comparable data;
        private Node left;
        private Node right;

        public Node(Comparable data) {
            this.data = data;
        }

        public void addNode(Node newNode) {
            if (newNode.data.compareTo(this.data) < 0) {
                if (left == null) {
                    left = newNode;
                } else {
                    left.addNode(newNode);
                }
            } else {
                if (right == null) {
                    right = newNode;
                } else {
                    right.addNode(newNode);
                }
            }
        }

        public void printNode() {   //中序遍历
            if (left != null) {
                left.printNode();
            }
            System.out.println(this.data);
            if (right != null) {
                right.printNode();
            }
        }
    }

    private Node root;

    public void add(Comparable data) {  //向二叉树中插入元素
        Node node = new Node(data);
        if (root == null) {
            root = node;
        } else {
            root.addNode(node);
        }
    }

    public void print() {
        root.printNode();
    }
}
```
调用如下
```Java
public static void main(String[] args) {
    int[] a = {12,0,34,5,2,8,456};
    BinarySortTree tree = new BinarySortTree();
    for (int i : a) {            
        tree.add(i);
    }
    tree.print();
}
```

## 查找算法

### 二分查找
只能对有序序列进行查找
```Java
public static int binarySearch(int[] array, int low, int high, int target) {
    if (low > high) return -1;
    int mid = low + (high - low) / 2;
    
    if (array[mid] > target)
        return binarySearch(array, low, mid - 1, target);

    if (array[mid] < target)
        return binarySearch(array, mid + 1, high, target);

    return mid;
}

public static int bSearchWithoutRecursion(int a[], int key) {
    int low = 0;
    int high = a.length - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (a[mid] > key)
            high = mid - 1;
        else if (a[mid] < key)
            low = mid + 1;
        else
            return mid;
    }
    return -1;
}
```

### 顺序查找
实现较简单，略过

### 二叉树查找
$可以通过构建一个二叉搜索树实现$