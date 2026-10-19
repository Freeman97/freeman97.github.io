---
title: 'LeetCode Weekly Contest #111 题解'
date: 2018-11-21
categories:
- 数据结构与算法
tags:
- LeetCode
- Algorithm
- 动态规划
- 减治法
- C++
---
**部分问题的题解是自己在比赛时实现的，有些题解的效率并不高，有些题解可能只是因为测试用例太弱而勉强AC，如果有人看到希望能够赐教。**
**所有题目都使用C++作答**
<!--more-->
# Q1 941. Valid Mountain Array 
## 题意
给出一个数组，判断其是否满足“山脉数组”的条件。（先单调增，到达最大元素，再单调减）
## 解法
暴力。
## 代码
```cpp
class Solution {
public:
    bool validMountainArray(vector<int>& A) {
        int n = A.size();
        if(n < 3)
        {
            return false;
        }
        bool a = true;
        int b = 0;
        bool c = true;
        int e = 0;
        for(int i = 0; i < n; i++)
        {
            if(A[i] > b)
            {
                b = A[i];
                e = i;
            }
        }
        if(e == 0 || e == (n - 1))
        {
            c = false;
        }
        for(int i = 0; i < e; i++)
        {
            if(A[i] >= A[i + 1])
            {
                c = false;
            }
        }
        cout << c << '\n';
        for(int i = e + 1; i < n; i++)
        {
            if(A[i - 1] <= A[i])
            {
                c = false;
            }
        }
        cout << c << '\n';
        return (a && c);
    }
};
```
# Q2 944. Delete Columns to Make Sorted
## 题意
给定字符串矩阵，删去不是正序排列的列，留下正序的列，求删去的列数。
## 解法
暴力。
## 代码
```cpp
class Solution {
public:
    int minDeletionSize(vector<string>& A) {
        int m = A.size();
        int n = A[0].size();
        int num = 0;
        for(int i = 0; i < n; i++)
        {
            for(int j = 0; j < m - 1; j++)
            {
                if(A[j][i] > A[j + 1][i])
                {
                    num++;
                    break;
                }
            }
        }
        return num;
    }
};
```
# Q3 942. DI String Match
## 题意
给定只包含"I"（增加）和"D"（减少）的长度为N的字符串，求符合该字符串增减规则的、由整数0至N构成的序列。
## 解法
### 个人解法
**非常暴力（很丑陋的暴力），非常弱智，非常慢**
#### 思路
1. 开一个长度为N+1的数组，令数组最左端的元素为0，其后元素按照DI字符串给出的规则直接模拟产生
2. 找到产生的序列中的最小元素，如果这个元素是负的，则数组中所有元素加上这个元素的绝对值进行修正
3. 循环寻找数组中剩余的最小元素，将整数0 ~ N中未被使用的数放置在对应位置。该过程中开了另一数组记录数字的使用情况（反思：即使要使用这种方法，利用bit mask的方式也更快，更省空间）


#### 代码
```cpp
class Solution {
public:
    vector<int> diStringMatch(string S) {
        int a[10002];
        int f[10001];
        int n = S.length();
        int b = 10000;
        int c = 0;
        vector<int> d;
        memset(a, 0, sizeof a);
        memset(f, 0, sizeof f);
        for(int i = 0; i < n; i++)
        {
            if(S[i] == 'I')
            {
                a[i + 1] = a[i] + 1;
            }
            else
            {
                a[i + 1] = a[i] - 1;
            }
        }
        for(int i = 0; i <= n; i++)
        {
            if(a[i] < b)
            {
                b = a[i];
                c = i;
            }
        }
        if(b < 0)
        {
            for(int i = 0; i <= n; i++)
            {
                a[i] -= b;
            }
        }
        f[c] = 1;
        // cout << c << "\n";
        for(int i = 1; i <= n; i++)
        {
            int e = 10000;
            c = 0;
            for(int j = 0; j <= n; j++)
            {
                if((a[j] <= e) && (f[j] == 0))
                {
                    e = a[j];
                    c = j;
                    // cout << "checked: " << c << " ";
                }
            }
            f[c] = 1;
            a[c] = i;
            // cout << "a[" << c << "] = " << i << "\n";
        }
        for(int i = 0; i <= n; i++)
        {
            d.push_back(a[i]);
            cout << d[i] << " ";
        }
        return d;
    }
};
```
### 解法 from 酒井算协
#### 解1
利用减治法，按照规则排列好前N-1个之后，如果第N个字符是增加，则直接把N放在末尾。 
如果第N个字符是减少，则前面所有数字+1，再把0放在末尾。
```cpp
class Solution {
    public:
    vector<int> diStringMatch(string s)
    {
        int n = s.size();
        vector<int> ans;
        ans.push_back(0);
        for(int i = 1; i <= n; i++)
        {
            if(s[i - 1] == 'I')
            {
                ans.push_back(i);
            }
            else
            {
                for(int j = 0; j < i; j++)
                {
                    ans[j]++;
                }
                ans.push_back(0);
            }
        }
        return ans;
    }
};
```
#### 解2
直接模拟
```cpp
class Solution {
    public:
    vector<int> diStringMatch(string s)
    {
        int n = s.size(), min_val = 1500, max_val = 1500;
        vector<int> ans;
        ans.resize(n + 1);
        ans[0] = 1500;
        for(int i = 1; i <= n; i++)
        {
            int tval = ans[i - 1];
            if(s[i - 1] == 'I')
            {
                tval = ++max_val;
            }
            else
            {
                tval = --min_val;
            }
            ans[i] = tval;
        }
        for(int i = 0; i <= n; i++)
        {
            ans[i] -= min_val;
        }
        return ans;
    }
};
```
# Q4 943. Find the Shortest Superstring
## 题意
给定一个字符串数组，求一个最短的字符串使得字符串数组中的每个字符串都是其子串。字符串数组的各个字符串数组之间不互为子串。
## 解法
动态规划。 
#### 大体思路
将所有字符串放在一行，使得这些字符串重叠的部分（overlap）最大。
#### 状态定义
`dp[mask][i]`。 `mask`是一个bit mask，表示了当前的串由哪些字符串组成（`mask`的第i位表示`A[i]`是否包含在当前字符串中，0为不包含，1为包含）。而`i`是指上一次被加入到当前串的字符串为`A[i]`。`dp[mask][i]`则表示了包含了由`mask`标识的字符串组成，且上一个被加入的字符串为`A[i]`的当前串的最大overlap。 
#### 状态转移
`dp[mask ^ (1 << j)][j] = max(overlap[A[i]][A[j]] + dp[mask][i]`。其中mask的第`j`位（bit）应该为0。（计算当前状态下加入新字符串`A[j]`所能得到的最大overlap，从状态`dp[mask][i]`转移到`dp[mask ^ (1 << j)][j]`）。
#### 初始状态
`dp`所有元素都为0
#### 算法步骤：
1. 对于所有可能的`i`，`j`排列，计算`overlap[i][j]`。
2. 对于所有可能出现的mask，计算`dp[mask][i]`，用`parent`二维数组对上述状态转移过程中`j`的上一个字符串下标`i`进行追踪
3. 利用`parent`中包含的信息重新构建出答案
## 代码
```cpp
class Solution {
    public:
    int dp[1 << 12][21], parent[1 << 12][21], overlap[21][21];
    string shortestSuperstring(vector<string>& A) {
        //compute overlap
        memset(dp, 0, sizeof(dp));
        int n = A.size();
        for(int i = 0; i < n; i++)
        {
            for(int j = 0; j < n; j++)
            {
                if(i == j)
                {
                    continue;
                }
                string s1 = A[i], s2 = A[j];
                for(int k1 = 0; k1 < s1.size(); k1++)
                {
                    bool ok = true;
                    for(int k2 = 0; k1 + k2 < s1.size() && k2 < s2.size(); k2++)
                    {
                        if(s1[k1 + k2] != s2[k2])
                        {
                            ok = false;
                            break;
                        }
                    }
                    if(ok)
                    {
                        overlap[i][j] = s1.size() - k1;
                        break;
                    }
                }
            }
        }
        for(int i = 1; i < (1 << n); i++)
        {
            // 此处的i即是思路中叙述的mask
            for(int j = 0; j <= n; j++)
            {
                // 若mask中的第j位是有被使用的，设此时状态为dp[mask][j]
                if((i >> j) & 1)
                {
                    // 从dp[mask][j]回退到A[j]并未被添加到当前串中的状态dp[mask ^ (1 << j)][i]，此时dp[mask ^ (1 << j)][i]是已经通过计算得出的
                    int pmask = i ^ (1 << j);
                    // 在pmask标识的状态中，上一个被加入的字符串是A[i]
                    // i的循环范围是pmask的各位中不为零的下标。因为当被使用的字符串由pmask标示时，
                    // 上一个被加入的字符串还没确定。选择一个i（此处循环使用了变量名k）
                    // 使得dp[mask][i] + overlap[i][j]最大，才能完成状态转移。
                    // 以下循环意图是求出dp[i][j]
                    for(int k = 0; k <= n; k++)
                    {
                        if((pmask >> k) & 1)
                        {
                            int val = dp[pmask][k] + overlap[k][j];
                            if(dp[i][j] == 0 || dp[i][j] < val)
                            {
                                dp[i][j] = val;
                                // 追踪：当已经使用的字符串被mask（此处为i）标示时，如果下一个加入的
                                // 字符串是A[j]，为了使dp的值最优（最大），上次被加入的字符串应该是
                                // A[k]
                                parent[i][j] = k;
                            }
                        }
                    }
                }
            }
        }
        // dp计算完成后，从最终状态（mask各位全为1），反推得最优解，最优的最终状态i用mark记录
        int mask = (1 << n) - 1;
        int val = dp[mask][0], mark = 0;
        for(int i = 0; i < n; i++)
        {
            if(dp[mask][i] > val)
            {
                val = dp[mask][i];
                mark = i;
            }
        }
        vector<int> seq;
        // 反推出答案
        while(mask)
        {
            seq.push_back(mark);
            int nxt = parent[mask][mark];
            mask ^= (1 << mark);
            mark = nxt;
        }
        string ans = "";
        for(int i = seq.size() - 1; i>= 0; i--)
        {
            string s = A[seq[i]];
            if(i == seq.size() - 1)
            {
                ans += s;
            }
            else
            {
                int over = overlap[seq[i + 1]][seq[i]];
                for(int j = over; j < s.size(); j++)
                {
                    ans += s[j];
                }
            }
        }
        return ans;
    }
};
```

