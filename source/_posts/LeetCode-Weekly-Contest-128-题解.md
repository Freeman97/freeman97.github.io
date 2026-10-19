---
title: LeetCode-Weekly-Contest-128-题解
date: 2019-03-18 14:46:58
categories:
- 数据结构与算法 
tags:
- LeetCode
- Algorithm
- 动态规划
- C++
---
**部分问题的题解是自己在比赛时实现的，有些题解的效率并不高，有些题解可能只是因为测试用例太弱而勉强AC，如果有人看到希望能够赐教。**
**所有题目都使用C++作答**
**Q4由于本人太鶸还没搞懂，搞懂了一定更新一定不鸽！**
<!--more-->
# Q1 1012. Complement of Base 10 Integer
## 题意
给出一个非负整数，求这个数在二进制表示方式下按位取反得到的十进制数。
## 解法
直接将此数字按位取反。由于输入是非负数，取反后正确答案的最高位必定为0，且最高位之后的所有位都是1。以此为依据可以将正确答案从低到高地还原出来。
## 代码
```cpp
class Solution {
public:
    int bitwiseComplement(int N) {
    if(N == 0)
    {
        return 1;
    }
	N = ~N;
	unsigned long complement = 0;
	bool flag = false;
	for(unsigned long i = pow(2, 30); i >= 1; i /= 2)
	{
		if((N & i) == 0)
		{
			flag = true;
		}
		if(flag)
		{
			int b = 0;
			if((N & i) != 0)
			{
				b = 1;
			}
			complement = complement * 2 + b;
		}
	}
    return complement;
    }
};
```
# Q2 1013. Pairs of Songs With Total Durations Divisible by 60
## 题意
给出一列正整数，其中存在正整数对`time[i], time[j]`，满足`(time[i] + time[j]) % 60 == 0`且`i > j`，求这一列正整数中所有的正整数对的数量。
## 解法
直接暴力会超时。考虑到正整数的可能个数（60000）远大于正整数的可能取值数（500），在遍历时记录下所有与当前`time[i]`相同的`time[j]`数量n，遇到下一个符合条件的`time[j]`时，计数器自加n。
## 代码
```cpp
class Solution {
public:
    int numPairsDivisibleBy60(vector<int>& time) {
        int num = 0;
        int ifChecked[60000] = {0};
        for(int i = 0; i < time.size(); i++)
        {
            if(ifChecked[i] == 1)
            {
                continue;
            }
            int currentNum = 1;
            for(int j = i + 1; j < time.size(); j++)
            {
                if((time[j] + time[i]) % 60 == 0)
                {
                    num += currentNum;
                }
                if(time[j] == time[i])
                {
                    ifChecked[j] = 1;
                    currentNum += 1;
                }
            }
        }
        return num;
    }
};
```
# Q3 1014. Capacity To Ship Packages Within D Days
## 题意
给定所有货物的重量`i`，求出运输船的最小载重量，使得这些货物能在`D`天内按顺序被运走
## 解法
直接枚举船的载重量，下限为最重的货物的的重量。（慢）
## 代码
```cpp
class Solution {
public:
    int shipWithinDays(vector<int>& weights, int D) {
        int max = 0;
        int capacity;
        for(int i = 0; i < weights.size(); i++)
        {
            if(weights[i] > max)
            {
                max = weights[i];
            }
        }
        for(capacity = max; ; capacity++)
        {
            int currentIndex = 0;
            for(int i = 0; i < D; i++)
            {
                int currentLoad = 0;
                for(int j = currentIndex; ; j++) 
                {
                    if(j >= weights.size())
                    {
                        return capacity;
                    }
                    currentLoad += weights[j];
                    if(currentLoad > capacity)
                    {
                        currentIndex = j;
                        break;
                    }
                }
            }
        }
    }
};
```
# Q4 1015. Numbers With Repeated Digits
## 题意
输入整数N，求不大于N的至少有一个重复数字的整数的数目。
## 解法
考虑DP，等我真的搞懂了再更新（