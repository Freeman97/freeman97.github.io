---
title: 剑指offer题解（一） - JZ1 ~ JZ10
date: 2020-08-11 20:39:49
categories: 数据结构与算法
tags:
- Algorithm
- Java
---

**二刷一下剑指offer的题顺便记录一下题解，日后再做分类整理（咕咕咕）**
**题目顺序和标号按照牛客网( https://www.nowcoder.com/ta/coding-interviews )，本文持续增量更新（**
<!--more-->
# JZ1 二维数组中的查找(查找, 数组)
## 题目描述
在一个二维数组中（每个一维数组的长度相同），每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。
## 解1
对每一个一维数组进行二分查找，设二维数组的尺寸是m * n，那么复杂度为`O(mlogn)`。
```java
public class Solution {
    public boolean Find(int target, int [][] array) {
        int level = 0, maxLevel = array.length, n = array[0].length - 1;
        if(n <= 0 || maxLevel <= 0) return false;
        int j = 0, k = n, l = 0;
        for(int i = 0; i < maxLevel; i++) {
            while(j < k) {
                l = (j + k) / 2;
                if(array[i][l] == target) {
                    return true;
                } else if(array[i][l] > target) {
                    k = l - 1;
                } else {
                    j = l + 1;
                }
            }
            if(array[i][j] == target) 
                return true;
            else if(array[i][j] < target) 
                k = n;
            else
                j = 0;
        }
        return false;
    }
}
```
## 解2
对于这一数组，从最左下角出发，向上数字减小，向右数字增大。该方法的时间复杂度为`O(m+n)`。
因此，若当前数字大于待查找整数，则向上查找。若当前数字小于待查找整数，则向右查找。可能存在的问题是“为什么在当前数字大于待查找整数时，不需要考虑向左查找”。对于这一问题，可以对“上一步操作”进行分情况讨论。
1. 上一步的操作是向右查找，那么这种情况必然不用考虑向左查找（走回头路）
2. 上一步的操作是向上查找，此时可以再次分情况讨论。如果之前此时的所有操作都不包括“向右查找”，那么此时当前位置位于数组的最左侧，无法继续向左查找。
3. 如果上一步的操作是向上查找，且之前的所有操作至少包含一次向右查找。记最近一次进行向右查找时的整数为`a[i][j]`，可以得知待查找整数必定比`a[i][j]`大。而二维数组向上时数字递减，则可知待查找整数比`a[i][0] ~ a[i][j]`都大，进而可知待查找整数`a[i][0] ~ a[i][j]`左侧的所有整数都大，因此不需要进行向左查找。
```java
public class Solution {
    public boolean Find(int target, int [][] array) {
        int level = 0, m = array.length, n = array[0].length;
        if(n <= 0 || m <= 0) return false;
        int i = m - 1, j = 0;
        while(i >= 0 && j < n) {
            if(array[i][j] > target) {
                i--;
            } else if(array[i][j] < target) {
                j++;
            } else {
                return true;
            }
        }
        return false;
    }
}
```

# JZ2 替换空格(字符串)
## 题目描述
请实现一个函数，将一个字符串中的每个空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。

## 解
没啥好说的字符串题，注意从后往前进行替换就行。
```java
public class Solution {
    public String replaceSpace(StringBuffer str) {
    	// 不使用额外存储空间
        // 从后往前移动更有效率
        // 首先计算空格数
        int spaceNum = 0;
        int oldLength = str.length();
        for(int i = 0; i < oldLength; i++) {
            if(str.charAt(i) == ' ')
                spaceNum++;
        }
        // 根据空格数扩大字符串
        str.setLength(str.length() + spaceNum * 2);
        // 从尾部开始替换较容易计算移动
        for(int i = oldLength - 1; i >= 0; i--) {
            if(str.charAt(i) != ' ') {
                str.setCharAt(i + spaceNum * 2, str.charAt(i));
            } else {
                str.replace(i + spaceNum * 2 - 2, i + spaceNum * 2 + 1, "%20");
                spaceNum--;
            }
        }
        return str.toString();
    }
}
```

# JZ3 从尾到头打印链表(链表)
## 题目描述
输入一个链表，按链表从尾到头的顺序返回一个ArrayList。

## 解
原地反转链表后输出，基本功。
```java
/**
*    public class ListNode {
*        int val;
*        ListNode next = null;
*
*        ListNode(int val) {
*            this.val = val;
*        }
*    }
*
*/
import java.util.ArrayList;
public class Solution {
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        // 反转链表之后就能顺序输出了
        ListNode newHead = null, p = listNode, q;
        ArrayList<Integer> res = new ArrayList<Integer>();
        while(p != null) {
            q = p;
            p = p.next;
            q.next = newHead;
            newHead = q;
        }
        p = newHead;
        while(p != null) {
            res.add(p.val);
            p = p.next;
        }
        return res;
    }
}
```

# JZ4 重建二叉树(树，数组，DFS)
## 题目描述
输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。

## 解
利用递归。将递归的每一步归纳为“找到根并确定左右子树的前序序列和中序序列”的过程，假设左右子树都已经构建好了，那么把左右子树连接到根节点上即可。
``` java
/**
 * Definition for binary tree
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
public class Solution {
    public TreeNode reConstructBinaryTree(int [] pre,int [] in) {
        if(pre.length <= 0 || in.length <= 0) return null;
        TreeNode res = reconstruct(pre, in, 0, pre.length - 1, 0, in.length - 1);
        return res;
    }
    // 递归
    public TreeNode reconstruct(int [] pre, int [] in, int preMin, int preMax, int inMin, int inMax) {
        if(preMin > preMax || inMin > inMax) return null;
        // 构造根
        TreeNode root = new TreeNode(pre[preMin]);
        int leftCount = 0, rightCount = 0, i = inMin;
        while(in[i++] != pre[preMin]) {
            leftCount++;
        }
        rightCount = inMax - leftCount - 1;
        root.left = reconstruct(pre, in, preMin + 1, preMin + leftCount, inMin, inMin + leftCount - 1);
        root.right = reconstruct(pre, in, preMin + leftCount + 1, preMax, inMin + leftCount + 1, inMax);
        return root;
    }
}
```

# JZ5 用两个栈实现队列(栈)
## 题目描述
用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。

## 解
```java
import java.util.Stack;

public class Solution {
    Stack<Integer> stack1 = new Stack<Integer>();
    Stack<Integer> stack2 = new Stack<Integer>();
    
    public void push(int node) {
        stack1.push(node);
    }
    
    public int pop() {
        if(stack1.isEmpty() && stack2.isEmpty())
            throw new RuntimeException("Stack is Empty");
        if(stack2.isEmpty()) {
            while(!stack1.isEmpty()) {
                stack2.push(stack1.pop());
            }
        }
        return stack2.pop();
    }
}
```

# JZ6 旋转数组的最小数字(二分，数组)
## 题目描述
把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。
输入一个非递减排序的数组的一个旋转，输出旋转数组的最小元素。
例如数组`[3,4,5,1,2]`为`[1,2,3,4,5]`的一个旋转，该数组的最小值为1。
NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。

## 解
旋转后的数组可以看作一前一后两个非递减序列，找到最小数字即找到第二个非递减序列的起点，可以考虑使用二分查找。如果不考虑相同数字，设数组`array`的查找起点为`min`，终点为`max`：
* 令`mid = (min + max) / 2`，若`array[min] <= array[mid] && array[max] <= array[mid]`，可知此时`mid`落在了前一个非递减序列，进而可知最小值点在`mid`之后。
* 若`array[mid] <= array[min] && array[mid] <= array[max]`，可知此时`mid`落在了后一个非递减序列，进而可知最小值点在`mid`之前。
* 若`array`在区间`[min, max]`上非递减，则已经定位到了第二个非递减序列，直接取`array[min]`即为整个数组的最小值
在此基础上考虑相同数字。如果`array`在`min, max, mid`三个位置的值都相等，则难以判断此时`mid`落在了哪一个非递减序列，此时只能顺序查找（牛客上的测试用例不够强，不考虑这一点也能AC）。
```java
import java.util.ArrayList;
public class Solution {
    public int minNumberInRotateArray(int [] array) {
        if(array.length <= 0) {
            return 0;
        }
        int min = 0, max = array.length - 1;
        while(min < max) {
            int mid = (min + max) / 2;
            if(array[min] == array[mid] && array[mid] == array[max]) {
                // find by order
                int res = array[min];
                while(min < max) {
                    if(array[min] < res) {
                        res = array[min];
                    }
                    min++;
                }
            }
            if(array[min] <= array[mid] && array[mid] <= array[max]) {
                return array[min];
            } else if(array[min] <= array[mid] && array[mid] >= array[max]){
                min = mid + 1;
            } else {
                max = mid;
                min++;
            }
        }
        return array[min];
    }
}
```

# JZ7 斐波那契数列
## 题目描述
大家都知道斐波那契数列，现在要求输入一个整数n，请你输出斐波那契数列的第n项（从0开始，第0项为0，第1项是1）。
n<=39

## 解
非常的简单。不过这算不算有一点DP的思想呢（
```java
public class Solution {
    public int Fibonacci(int n) {
        if(n == 0 || n == 1) return n;
        int res = 1, pre = 1;
        n -= 2;
        int temp = 0;
        while(n-- > 0) {
            temp = res + pre;
            pre = res;
            res = temp;
        }
        return res;
    }
}
```

# JZ8 跳台阶(递归，动态规划)
## 题目描述
一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法（先后次序不同算不同的结果）。

## 解
可以先观察一下n比较小的情况。设`N(k)`为`k`级台阶的跳法数：
* 跳0级: 0种跳法
* 跳1级: 1种跳法
* 跳2级: 2种跳法
* 跳3级: N(2) + N(1)
* 跳4级: N(3) + N(2)
进而知道，需要跳`n`级时，前一步要么跳了1级，要么跳了2级。那么只需要知道跳`n-1`级和跳`n-2`级分别有几种跳法，即可算出`N(n) = N(n - 1) + N(n - 2)`
需要注意的是，不要认为“跳2级有2种跳法”，进而得到`N(n) = N(n - 1) + 2 * N(n - 2)`的结论，因为“先跳`n - 2`级，再跳两次1级”的情况与“先跳`n - 1`级，再跳1级”的情况有重复。
本题的非递归做法比较容易看出，因此使用非递归做法以节省空间。
```java
public class Solution {
    public int JumpFloor(int target) {
        if(target >= 0 && target <= 2) return target;
        int res = 2, pre = 1, temp = 0;
        target -= 2;
        while(target-- > 0) {
            temp = res + pre;
            pre = res;
            res = temp;
        }
        return res;
    }
}
```

# JZ9 变态跳台阶(递归)
## 题目描述
一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

## 解
还是可以先观察一下`n`比较小的情况。
* 跳0级: 0种跳法
* 跳1级: 1种跳法
* 跳2级: 2种跳法 = N(1) + 1
* 跳3级: 4种跳法 = N(2) + N(1) + 1 = 2 * N(2)
* 跳4级: 8种跳法 = N(3) + N(2) + 1 = 2 * N(3)
需要跳`n`级时，前一步可以跳`0, 1, 2, ..., n - 1`级。可知`N(n) = 1 + N(1) + N(2) + ... + N(n - 1)`，进一步化简得`N(n) = [1 + N(1) + ... + N(n - 2)] + N(n - 1) = 2 * N(n - 1)`。转换为非递归做法也比较容易。
```java
public class Solution {
    public int JumpFloorII(int target) {
        if(target == 0 || target == 1) return target;
        target -= 2;
        int res = 2;
        while(target-- > 0) {
            res *= 2;
        }
        return res; 
    }
}
```

# JZ10 矩形覆盖
## 题目描述
我们可以用2*1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2*1的小矩形无重叠地覆盖一个2*n的大矩形，总共有多少种方法？
比如n=3时，2*3的矩形块有3种覆盖方法

## 解
`2 * n`的矩形可以视为由两种“块”填满：
* 竖着摆的2 * 1
* 两条上下摆放的横向2 * 1构成的2 * 2块
需要注意的是没有第三种，两条左右摆的纵向2 * 1就变成了两个第一种块。
于是本题的做法就和JZ8 跳台阶一样了。
```java
public class Solution {
    public int RectCover(int target) {
        if(target >= 0 && target <= 2) return target;
        target -= 2;
        int res = 2, pre = 1, temp = 0;
        while(target-- > 0) {
            temp = res + pre;
            pre = res;
            res = temp;
        }
        return res;
    }
}
```