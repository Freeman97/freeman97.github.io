---
title: 剑指offer题解（三） - JZ21 ~ JZ30
date: 2020-09-13 09:54:57
categories: 数据结构与算法
tags:
- Algorithm
- Java
---

**二刷一下剑指offer的题顺便记录一下题解，日后再做分类整理（咕咕咕）**
**题目顺序和标号按照牛客网( https://www.nowcoder.com/ta/coding-interviews )，本文持续增量更新（**
<!--more-->

# JZ21 栈的压入、弹出序列(栈)
## 题目描述
输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）

## 解
模拟栈的压入和弹出过程即可，按照入栈顺序逐个压入元素。如果当前栈顶元素与出栈顺序的当前元素相等，则将栈中元素出栈。若最终能得到空栈，则出栈顺序正确，否则可以判断该顺序不是一个正确的出栈顺序。
```java
import java.util.*;

public class Solution {
    public boolean IsPopOrder(int [] pushA,int [] popA) {
        // 朴素模拟？
        Stack<Integer> s = new Stack<>();
        int i = 0, j = 0, n = pushA.length;
        boolean flag = true;
        while(i < n && j < n) {
            if(s.isEmpty()) {
                s.push(pushA[i]);
                i++;
            } else if(s.peek() == popA[j]) {
                s.pop();
                j++;
            } else {
                s.push(pushA[i]);
                i++;
            }
        }
        // 栈内剩下的元素如果不能匹配则说明这不是一个弹出序列
        while(!s.isEmpty()) {
            if(s.pop() != popA[j]) {
                flag = false;
                break;
            }
            j++;
        }
        return flag;
    }
}
```

# JZ22 从上往下打印二叉树(树，队列)
## 题目描述
从上往下打印出二叉树的每个节点，同层节点从左至右打印。
## 解
树的层次遍历法
```java
import java.util.ArrayList;
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
    public ArrayList<Integer> PrintFromTopToBottom(TreeNode root) {
        // 层次遍历
        ArrayList<Integer> res = new ArrayList<>();
        ArrayList<TreeNode> q = new ArrayList<>();
        // ArrayList的remove(0)的时间复杂度是？
        int head = -1;
        if(root == null)
            return res;
        q.add(root);
        TreeNode p = null;
        while(head < q.size() - 1) {
            p = q.get(++head);
            res.add(p.val);
            if(p.left != null)
                q.add(p.left);
            if(p.right != null)
                q.add(p.right);
        }
        return res;
    }
}
```

# JZ23 二叉搜索树的后序遍历序列(树，栈)
## 题目描述
输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则返回true,否则返回false。假设输入的数组的任意两个数字都互不相同。

## 解
对于一个二叉搜索树而言，当前节点的左子树所有节点值都比当前节点值小，当前节点的右子树所有节点值都比当前节点大。而根据“左-右-根”的遍历顺序，后序遍历序列的前一部分为左子树，中间一部分为右子树，最后一个元素为根。前一部分的所有值都比最后一个元素小，后一部分的所有值都比最后一个元素大。而序列的左子树部分和右子树部分也满足上述定义，可以看出这是一个递归的定义。可以使用递归方法和非递归方法完成此题。
递归做法
```java
public class Solution {
    public boolean VerifySquenceOfBST(int [] sequence) {
        if(sequence.length <= 0)
            return false;
        return verify(sequence, 0, sequence.length - 1);
    }
    
    public boolean verify(int [] seq, int min, int max) {
        if(min >= max)
            return true;
        int mid = 0;
        int pivot = seq[max], i = min;
        while(seq[i] < pivot) {
            i++;
        }
        mid = i;
        while(i < max) {
            if(seq[i] < pivot) {
                return false;
            }
            i++;
        }
        return verify(seq, min, mid - 1) && verify(seq, mid, max - 1);
    }
}
```
非递归做法
```java
import java.util.*;
public class Solution {
    public boolean VerifySquenceOfBST(int [] sequence) {
        if(sequence.length <= 0)
            return false;
        Stack<Integer> s = new Stack<>();
        s.push(0);
        s.push(sequence.length - 1);
        while(!s.isEmpty()) {
            int max = s.pop(), min = s.pop();
            int pivot = sequence[max], i = min, mid = min;
            while(sequence[i] < pivot && i < max)
                i++;
            mid = i;
            while(i < max) {
                if(sequence[i] < pivot) {
                    return false;
                }
                i++;
            }
            if(min < mid - 1) {
                s.push(min);
                s.push(mid - 1);
            }
            if(mid < max - 1) {
                s.push(mid);
                s.push(max - 1);
            }
        }
        return true;
    }
}
```
需要注意的是，想要利用中序序列和后序序列之间的关系来完成本题是不正确的（虽然可以通过牛客的用例）。以中序序列为入栈顺序进行入栈的话，若某一序列是该树的后序序列，那么这一序列属于一种出栈序列。但是若某一序列是一种以中序序列为入栈顺序的出栈序列，那么这一序列不一定是树的后序序列。

# JZ24 二叉树中和为某一值的路径(树，DFS)
## 题目描述
输入一颗二叉树的根节点和一个整数，按字典序打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。

## 解
递归地遍历整棵树，并用全局变量记录根节点到当前节点的路径。遇到叶子节点时判断路径上所有节点值之和是否为目标值，如果是，将当前的路径信息深拷贝后插入到结果集合中。如果使用非递归算法，需要对树进行后序遍历。
```java
import java.util.ArrayList;
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
    ArrayList<ArrayList<Integer>> res = new ArrayList<ArrayList<Integer>>();
    ArrayList<Integer> path = new ArrayList<>();
    
    public ArrayList<ArrayList<Integer>> FindPath(TreeNode root,int target) {
        if(root == null) return res;
        path.add(root.val);
        if(root.left == null && root.right == null && target - root.val == 0) {
            res.add(new ArrayList<Integer>(path));
        }
        FindPath(root.left, target - root.val);
        FindPath(root.right, target - root.val);
        // 回收当前节点
        path.remove(path.size() - 1);
        return res;
    }
}
```

# JZ25 复杂链表的复制(链表)
## 题目描述
输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针random指向一个随机节点），请对此链表进行深拷贝，并返回拷贝后的头结点。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空）

## 解
3种解法，个人先想出的第一种，仔细想了想想出第二种，第三种是参考了牛客上的解法。

### 解1 暴力
先不处理特殊指针，使用`next`指针复制链表的所有节点。然后在原链表中为每一个节点使用`next`指针定位每一个随机指针的指向，新链表和原链表的查找指针保持同步，找到后将随机指针的指向同步反映到新链表中：如果原链表中第`i`个节点的随机指针指向第`j`个节点，新链表的第`i`个节点的随机指针也指向新链表的第`j`个节点。时间复杂度为$O(n^2)$
```java
/*
public class RandomListNode {
    int label;
    RandomListNode next = null;
    RandomListNode random = null;

    RandomListNode(int label) {
        this.label = label;
    }
}
*/
public class Solution {
    public RandomListNode Clone(RandomListNode pHead)
    {
        // 先不处理特殊指针
        RandomListNode res = new RandomListNode(-1), p = pHead, q = res, r = null, s = null;
        while(p != null) {
            q.next = new RandomListNode(p.label);
            q = q.next;
            p = p.next;
        }
        // 暴力一点的话，时间复杂度就便乘了n^2: 在原链表中暴力寻找随机指针的指向，同步反应到新链表中
        res = res.next;
        r = pHead;
        s = res;
        while(r != null) {
            p = pHead;
            q = res;
            while(p != r.random) {
                p = p.next;
                q = q.next;
            }
            s.random = q;
            s = s.next;
            r = r.next;
        }
        return res;
    }
}
```

## 解2 用HashMap储存原链表节点和新链表节点的映射关系
解1为了在新链表中还原随机指针，要在新链表中为每一个节点查找随机指针的指向，而每次查找都要遍历一遍整个链表。解2为了优化还原随机指针的过程，在解1使用`next`指针复制链表节点时，使用`HashMap`保存了原链表节点和新链表节点的映射关系。这样在新链表上还原第`i`个节点的随机指针时，只需要找到原链表上第`i`个节点的`random`所指向的节点A，再通过`HashMap`就可以找到与节点A所对应的新链表节点A'，让新链表上第`i`个节点指向A'即还原了新链表第`i`个节点的随机指针。时间复杂度为$O(n)$，需要$O(n)$的辅助存储空间。
```java
/*
public class RandomListNode {
    int label;
    RandomListNode next = null;
    RandomListNode random = null;

    RandomListNode(int label) {
        this.label = label;
    }
}
*/
import java.util.*;
public class Solution {
    public RandomListNode Clone(RandomListNode pHead)
    {
        // 先不处理随机指针, 但是建立旧链表和新链表的映射关系。后续利用映射关系建表，需要使用辅助空间，复杂度为O(n)
        // 时间复杂度为O(n)
        RandomListNode res = new RandomListNode(-1), p = pHead, q = res, r = null, s = null;
        HashMap<RandomListNode, RandomListNode> m = new HashMap<>();
        while(p != null) {
            q.next = new RandomListNode(p.label);
            q = q.next;
            m.put(p, q);
            p = p.next;
        }
        // 使用HashMap重建随机指针
        res = res.next;
        q = res;
        p = pHead;
        while(q != null) {
            if(p.random != null) {
                q.random = m.get(p.random);
            }
            q = q.next;
            p = p.next;
        }
        return res;
    }
}
```

### 解3 先原地复制复杂链表的节点，再进行拆分
首先将原链表的每个节点复制，将复制后的节点插入到原节点的后面。根据原链表节点`p`的随机指针还原复制节点`q`的随机指针：`q.random = p.random.next`（复制节点是原节点的后继节点）。最后将两个链表进行拆分即可。时间复杂度为$O(n)$，并且辅助存储空间复杂度为$O(1)$。
```java
/*
public class RandomListNode {
    int label;
    RandomListNode next = null;
    RandomListNode random = null;

    RandomListNode(int label) {
        this.label = label;
    }
}
*/
import java.util.*;
public class Solution {
    public RandomListNode Clone(RandomListNode pHead)
    {
        // 先原地复制复杂链表的每个节点，将复制后的节点插入到原节点后面
        // 然后根据原链表在复制节点上还原随机指针的指向关系
        // 最后拆分
        // 额外的存储空间复杂度为O(1)，时间复杂度为O(n)
        if(pHead == null) return null;
        RandomListNode res = new RandomListNode(-1), p = pHead, q = null;
        while(p != null) {
            q = new RandomListNode(p.label);
            q.next = p.next;
            p.next = q;
            p = q.next;
        }
        p = pHead;
        while(p != null) {
            q = p.next;
            if(p.random != null)
                q.random = p.random.next;
            p = q.next;
        }
        p = pHead;
        q = res;
        // 注意：要将复制后的节点从原链表中解开（原链表不能有指针指向复制节点），否则会被判空
        while(p != null) {
            q.next = p.next;
            p.next = (p.next).next;
            q = q.next;
            p = p.next;
        }
        return res.next;
    }
}
```