---
title: LeetCode-Weekly-Contest-129-题解
date: 2019-03-25 00:00:47
categories: 
- 数据结构与算法
tags:
- LeetCode
- Algorithm
- KMP
- 动态规划
- C++
---
**部分问题的题解是自己在比赛时实现的，有些题解的效率并不高，有些题解可能只是因为测试用例太弱而勉强AC，如果有人看到希望能够赐教。**
**所有题目都使用C++作答**
<!--more-->
# Q1 1020. Partition Array Into Three Parts With Equal Sum
## 题意
给出一个整数数组``A``，若能找到下标``i``，``j``满足``i + 1 < j``且``A[0] + A[1] + ... + A[i] == A[i + 1] + A[i + 2] + ... + A[j - 1] == A[j] + A[j + 1] + ... + A[A.length - 1]``，则返回``true``，否则返回``false``。
## 解法
三个部分的和相等时，三个部分的和将等于此数组所有元素的算术平均数，且这一算术平均数必定为整数。据此对数组进行划分，不能成功划分即返回``false``。
## 代码
```cpp
class Solution {
public:
    bool canThreePartsEqualSum(vector<int>& A) {
        int sum1 = 0;
        int sum2 = 0;
        int sum3 = 0;
        int sum = 0;
        for(int i = 0; i < A.size(); i++)
        {
            sum += A[i];
        }
        int av = sum / 3;
        int l = sum % 3;
        int Index = 0;
        if(l != 0)
        {
            return false;
        }
        for(int i = 0; i < A.size(); i++)
        {
            sum1 += A[i];
            if(sum1 == av)
            {
                Index = i + 1;
                break;
            }
        }
        if(sum1 != av)
        {
            return false;
        }
        for(int i = Index; i < A.size(); i++)
        {
            sum2 += A[i];
            if(sum2 == av)
            {
                Index = i + 1;
                break;
            }
        }
        if(sum2 != av)
        {
            return false;
        }
        for(int i = Index; i < A.size(); i++)
        {
            sum3 += A[i];
        }
        if(sum1 == sum2 && sum2 == sum3)
        {
            return true;
        }
        else
        {
            return false;
        }
    }
};
```
# Q2 1022. Smallest Integer Divisible by K
## 题意
给出一个正整数``K``，找到能被``K``整除的最小正整数，并且此正整数的所有位数字均为1。若能找到这样的正整数，返回这一正整数的位数（长度），若找不到这样的正整数则返回-1。
## 解法
首先排除偶数。对于某一所有位均为1的正整数``n``，若``n``除以``K``的余数为``r``，则``n * 10 + 1``（下一个大于``n``的全1正整数）除以``K``的余数可表示为``(r * 10 + 1) % K``。当``r == 0``时意味着找到了该整数，而当``r``重复出现时意味着找不到满足条件的正整数。
## 代码
```cpp
class Solution {
public:
    int smallestRepunitDivByK(int K) {
        if(K % 2 == 0)
        {
            return -1;
        }
        int length = 1;
        int n = 1;
        int r = 0;
        while(n < K)
        {
            n = n * 10 + 1;
            length++;
        }
        r = n % K;
        if(!r)
        {
            return length;
        }
        std::set<int> s;
        while(r)
        {
            r = (r * 10 + 1) % K;
            length++;
            if(s.find(r) == s.end())
            {
                s.insert(r);
            }
            else
            {
                return -1;
            }
        }
        return length;
    }
};
```
# Q3 1021. Best Sightseeing Pair
## 题意
给出一个正整数数组，求下标对``i, j (i < j)``，使得``A[i] + A[j] + i - j``最大。
## 解法
考虑DP。对于数组中的每一个元素``A[i]``，不停地更新``bestPrev``（比较``A[i]``和``A[bestPrev]``，取大者），使得``A[bestPrev] + bestPrev``最大，此时即可让``A[i] + A[bestPrev] + bestPrev - i``最大。
## 代码
```cpp
class Solution {
public:
    int maxScoreSightseeingPair(vector<int>& A) {
        int bestPrev = 0;
        int result = A[0];
        for(int i = 1; i < A.size(); i++)
        {
            result = max(result, A[i] + A[bestPrev] + bestPrev - i);
            if(A[i] + i > A[bestPrev] + bestPrev)
            {
                bestPrev = i;
            }
        }
        return result;
    }
};
```
# Q4 1023. Binary String With Substrings Representing 1 To N
## 题意
给定一个只含有``'0'``和``'1'``的字符串``S``和正整数``N``，当从1到``N``的每一个正整数X的二进制表示都是``S``的子串时，返回``true``，否则返回``false``。
## 解法
将从1到``N``的所有正整数转化为二进制表示，用KMP算法确认二进制表示是否为``S``的子串。
## 代码
```cpp
class Solution {
    // calculate the 'next array'
    void GetNext(string t, int next[])
    {
        int j = 0, k = -1;
        next[0] = -1;
        
        while(j < t.size() - 1)
        {
            if(k == -1 || t[j] == t[k])
            {
                j++;
                k++;
                next[j] = k;
            }
            else
            {
                k = next[k];
            }
        }
    }
    // KMP Algorithm
    int KMP(string s, string t)
    {
        int next[1001];
        int i = 0, j = 0;
        GetNext(t, next);
        while(i < (int)s.size() && j < (int)t.size())
        {
            if(j == -1 || s[i] == t[j])
            {
                i++;
                j++;
            }
            else
            {
                j = next[j];
            }
        }
        if(j >= t.size())
        {
            return true;
        }
        else
        {
            return false;
        }
    }
public:
    bool queryString(string S, int N) {
        int next[1001];
        for(int i = 1; i <= N; i++)
        {
            string tempStr;
            int temp = i;
            while(temp)
            {
                tempStr = (char)(temp % 2 + '0') + tempStr;
                temp /= 2;
            }
            if(!KMP(S, tempStr))
            {
                return false;
            }
        }
        return true;
    }
};
```
