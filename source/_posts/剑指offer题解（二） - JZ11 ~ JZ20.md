---
title: 剑指offer题解（二） - JZ11 ~ JZ20
date: 2020-09-08 16:33:26
categories: 数据结构与算法
tags:
- Algorithm
- Java
---

**二刷一下剑指offer的题顺便记录一下题解，日后再做分类整理（咕咕咕）**
**题目顺序和标号按照牛客网( https://www.nowcoder.com/ta/coding-interviews )，本文持续增量更新（**
<!--more-->

# JZ11 二进制中1的个数(数学，数字的机器表示)
## 题目描述
输入一个整数，输出该数32位二进制表示中1的个数。其中负数用补码表示。

## 解
一般情况下整数的机器表示使用补码，便于实现减法和负数运算。于是直接用位运算数出来就好。
```java
public class Solution {
    // 可以稍微看一下java中的>>和>>>的区别
    public int NumberOf1(int n) {
        int length = 32, mask = 1, count = 0;
        while(length-- > 0) {
            if((n & mask) == 1) {
                count++;
            }
            n = n >> 1;
        }
        return count;
    }
}
```

# JZ12 数值的整数次方(数学)
## 题目描述
给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。
保证base和exponent不同时为0

## 解
基本思路是使用乘法快速幂，注意这题的`exponent`可以为负数。乘法快速幂的详解可以查看另一篇文章“算法中的数学类型题归集”。
递归版本
```java
public class Solution {
    public double Power(double base, int exponent) {
        if(base == 0)
            return 0;
        double res = absPower(base, exponent);
        if(exponent >= 0)
            return res;
        else
            return 1 / res;
    }
    public double absPower(double base, int exponent) {
        if(exponent == 0) 
            return 1;
        double temp = 1;
        // exponent绝对值为奇数的时候，对2求模可能为1或-1
        // 这么表达更简洁
        if((exponent & 1) == 1) {
            temp = base;
        }
        double ret = absPower(base, exponent / 2);
        return ret * ret * temp;
    }
}
```
非递归版本
```cpp
class Solution {
public:
    double Power(double base, int exponent) {
        // 本题是一个简单的乘法快速幂
        // 二进制分解exponent，达到log(exponent)的时间复杂度
        if(exponent == 0) return 1;
        if(base == 0) return 0;
        if(exponent < 0) exponent--;
        double res = 1;
        int length = sizeof(int) * 8;
        while(length--) {
            if((exponent & 1) && exponent > 0) {
                res *= base;
            } else if(!(exponent & 1) && exponent < 0) {
                res = res / base;
            }
            base *= base;
            exponent >>= 1;
        }
        return res;
    }
};
```

# JZ13 调整数组顺序使奇数位于偶数前面(数组)
## 题目描述
输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。

## 解
因为要保持数据的稳定性，没办法借鉴快速排序的想法了。不考虑以空间换时间的话，可以借鉴一下插入排序的思想：先从前往后找到第一个偶数，再从此处开始向后找到第一个奇数，将奇数插入到偶数之前。时间复杂度$O(n^2)$。
```java
public class Solution {
    public void reOrderArray(int [] array) {
        int n = array.length;
        for(int i = 0; i < n; ) {
            if(array[i] % 2 == 0) {
                int j = i + 1;
                while(j < n) {
                    if(array[j] % 2 != 0) {
                        break;
                    }
                    j++;
                }
                if(j < n) {
                    int temp = array[j], index = j;
                    while(j > i) {
                        array[j] = array[j - 1];
                        j--;
                    }
                    array[i] = temp;
                } else {
                    break;
                }
                i = j + 1;
            } else {
                i++;
            }
        }
    }
}
```

# JZ14 链表中倒数第k个节点(链表)
## 题目描述
输入一个链表，输出该链表中倒数第k个结点。

## 解
很朴素的双指针。
```java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/
public class Solution {
    public ListNode FindKthToTail(ListNode head,int k) {
        ListNode p = head, q = head;
        while(k > 0 && p != null) {
            p = p.next;
            k--;
        }
        if(k > 0)
            return null;
        while(p != null) {
            p = p.next;
            q = q.next;
        }
        return q;
    }
}
```

# JZ15 反转链表(链表)
## 题目描述
输入一个链表，反转链表后，输出新链表的表头。

## 解
个人感觉想成“利用原来列表的所有节点，使用头插法构造新链表”会比较简单。
```java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/
public class Solution {
    public ListNode ReverseList(ListNode head) {
        ListNode res = null, p = head;
        // 头插法
        while(p != null) {
            ListNode q = p;
            p = p.next;
            q.next = res;
            res = q;
        }
        return res;
    }
}
```

# JZ16 合并两个排序的链表(链表，分治)
## 题目描述
输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。

## 解
每次从两个链表的头节点中选一个值较小的节点，用尾插法插入结果链表中。最后可能会有其中一个链表还有节点剩下，直接让其与结果链表相接即可。
```java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/
public class Solution {
    public ListNode Merge(ListNode list1,ListNode list2) {
        ListNode res = null, p = list1, q = list2, tail = null;
        // 对于无头结点的链表题，可以通过手动构造一个头来避免处理特殊情况
        res = new ListNode(-1);
        tail = res;
        while(p != null && q != null) {
            if(p.val <= q.val) {
                tail.next = p;
                p = p.next;
            } else {
                tail.next = q;
                q = q.next;
            }
            tail = tail.next;
        }
        if(p != null)
            tail.next = p;
        if(q != null)
            tail.next = q;
        return res.next;
    }
}
```

# JZ17 树的子结构(树)
## 题目描述
输入两棵二叉树A，B，判断B是不是A的子结构。（ps：我们约定空树不是任意一个树的子结构）

## 解
首先可以使用递归的方法实现判断子结构是否匹配的算法。然后使用递归的方法对于A树的每一个节点使用一次这个算法，直到找到第一处成功匹配的子结构。（暂时不知道还能不能更快，看到一个思路是想办法序列化二叉树，空节点用一个题目中不会出现的节点值来填充，然后使用KMP，不过暂时没有实现）
```java
/**
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
import java.util.*;
public class Solution {
    public boolean HasSubtree(TreeNode root1,TreeNode root2) {
        if(root2 == null || root1 == null) return false;
        boolean res = false, lRes = false, rRes = false;
        res = ifCurrentNodeEquals(root1, root2);
        if(!res) {
            // 稍微更快一点：如果此处匹配成功，则不必继续向下查找是否有匹配的子结构
            lRes = HasSubtree(root1.left, root2);
            rRes = HasSubtree(root1.right, root2);
        }
        return res || lRes || rRes;
    }
    
    public boolean ifCurrentNodeEquals(TreeNode tree1, TreeNode tree2) {
        // 这两个if和else if的语句稍微注意一下，其实对几种情况进行了合并
        if(tree2 == null) {
            return true;
        } else if(tree1 == null) {
            return false;
        } else if(tree1.val == tree2.val){
            boolean lRes = ifCurrentNodeEquals(tree1.left, tree2.left);
            boolean rRes = ifCurrentNodeEquals(tree1.right, tree2.right);
            return lRes && rRes;
        } else {
            return false;
        }
    }
}
```

# JZ18 二叉树的镜像(树)
## 题目描述
操作给定的二叉树，将其变换为源二叉树的镜像。
```
二叉树的镜像定义：源二叉树 
    	    8
    	   /  \
    	  6   10
    	 / \  / \
    	5  7 9 11
    	镜像二叉树
    	    8
    	   /  \
    	  10   6
    	 / \  / \
    	11 9 7  5
```

## 解
可以使用递归。
```java
/**
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
public class Solution {
    public void Mirror(TreeNode root) {
        if(root == null) return;
        TreeNode temp = root.left;
        root.left = root.right;
        root.right = temp;
        Mirror(root.left);
        Mirror(root.right);
    }
}
```
非递归版本：遍历每个节点并反转每个节点的左右子树
```java
import java.util.*;
public class Solution {
    public void Mirror(TreeNode root) {
        if(root == null) return;
        TreeNode p = root;
        Stack<TreeNode> s = new Stack<TreeNode>();
        while(!s.isEmpty() || p != null) {
            if(p != null) {
                s.push(p);
                // reverse
                TreeNode temp = p.left;
                p.left = p.right;
                p.right = temp;
                p = p.left;
            } else {
                p = s.pop();
                p = p.right;
            }
        }
    }
}
```

# JZ19 顺时针打印矩阵(数组)
##　题目描述
输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字，例如，如果输入如下4 X 4矩阵： 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 则依次打印出数字1,2,3,4,8,12,16,15,14,13,9,5,6,7,11,10.

## 解
个人很不擅长做二维数组相关的题目，因为比较粗心，边界条件总是理不清楚，得加强练习。个人解的思路如下：顺时针打印时，方向总是遵循“水平-竖直-水平-竖直”的规律。水平方向打印完一条边后，竖直方向需要打印的元素数会减少一个（避免重复打印角上的元素），反之亦然。重复这一过程，直至某一边需要打印的元素数变为0。
```java
import java.util.ArrayList;
public class Solution {
    public ArrayList<Integer> printMatrix(int [][] matrix) {
        ArrayList<Integer> res = new ArrayList<>();
        int n = matrix.length;
        if(n <= 0) return res;
        // 下标j从-1开始计算更好控制
        int m = matrix[0].length, i = 0, j = -1;
        if(m <= 0) return res;
        int direction = 1;
        boolean ifHorizontal = true;
        // n, m分别代表i, j当前要在对应方向上"前进"的步数
        while(n > 0 && m > 0) {
            if(ifHorizontal) {
                int hLength = m;
                while(hLength-- > 0) {
                    j += direction;
                    res.add(matrix[i][j]);
                }
                n--;
            } else {
                int vLength = n;
                while(vLength-- > 0) {
                    i += direction;
                    res.add(matrix[i][j]);
                }
                m--;
                direction *= -1;
            }
            ifHorizontal = !ifHorizontal;
        }
        return res;
    }
}
```
一次循环打印一圈的做法代码会更简洁，不过边界条件需要计算好。

# JZ20 包含min函数的栈(栈)
## 题目描述
定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））。

## 解
压入栈中的最小值可能被弹出，因此需要记录新最小值入栈之前的所有最小值。可以使用辅助栈来保存当前栈中的最小值和之前的最小值。至于在当前最小值之后入栈的、比当前最小值大的元素，因为它必将比当前最小值先出栈，永远没有可能成为某一时刻的最小值，因此不需要记录这类元素的信息。
```java
import java.util.Stack;

public class Solution {
    // 最小值栈的维护：
    // 栈顶储存当前栈内的最小值
    // 如果当前入栈元素比栈顶元素大，则不理会（肯定会比最小值先出栈，没必要记录）
    // 如果当前入栈元素小于等于栈顶元素，则将其加入最小值栈
    Stack<Integer> dataStack = new Stack<>();
    Stack<Integer> minStack = new Stack<>();
    
    public void push(int node) {
        dataStack.push(node);
        if(minStack.isEmpty() || node <= minStack.peek()) {
            minStack.push(node);
        } 
    }
    
    public void pop() {
        int temp = dataStack.pop();
        if(temp == minStack.peek()) {
            minStack.pop();
        }
    }
    
    public int top() {
        return dataStack.peek();
    }
    
    public int min() {
        return minStack.peek();
    }
}
```