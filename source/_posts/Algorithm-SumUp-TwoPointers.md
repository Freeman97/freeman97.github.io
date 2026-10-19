---
title: 算法专题 - 双指针
date: 2020-10-04 17:34:41
categories: 数据结构与算法
tags:
- Algorithm
- Java
---
**分类刷题归集，总结遇到的各种双指针问题**
**题目描述和编号来自Leetcode中文站**
<!--more-->

# 左右指针系列问题
## Leetcode 11 盛最多水的容器
### 题目描述
https://leetcode-cn.com/problems/container-with-most-water/

给你 n 个非负整数 a1，a2，...，an，每个数代表坐标中的一个点 (i, ai) 。在坐标内画 n 条垂直线，垂直线 i 的两个端点分别为 (i, ai) 和 (i, 0)。找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。
说明：你不能倾斜容器，且 n 的值至少为 2。

### 解
一道比较典型的可以用双指针法处理的问题，可以假设最开始时两个指针分别处于数组的两端，按照一定的规则逐渐相向移动。对于任意时刻的两个不同位置指针`i`和`j`，不妨假设`i < j`，能够构成容量为`v1 = min(height[i], height[j]) * (j - i)`的容器。进一步假设`height[i] < height[j]`，则此时无论如何向左移动指针`j`，得到的新容器容量都不会超过`v1`，因为`min(height[i], height[j])`有可能与原来相等也有可能比原来小，但是`j - i`一定比原来小。由此可见，此时向右移动指针`i`才有可能得到容量更大的容器。该解法的时间复杂度为$O(n)$，空间复杂度为$O(1)$。
```java
class Solution {
    public int maxArea(int[] height) {
        int i = 0, j = height.length - 1, res = 0, current = 0;
        if(i >= j)
            return res;
        while(i < j) {
            current = Math.min(height[i], height[j]) * (j - i);
            res = Math.max(current, res);
            if(height[i] < height[j]) {
                i++;
            } else {
                j--;
            }
        }
        return res;
    }
}
```

# Sum系列问题(左右指针)
## Leetcode 16 最接近的三数之和
### 题目描述
https://leetcode-cn.com/problems/3sum-closest

给定一个包括`n`个整数的数组`nums`和 一个目标值`target`。找出`nums`中的三个整数，使得它们的和与`target`最接近。返回这三个数的和。假定每组输入只存在唯一答案。
示例：
```
输入：nums = [-1,2,1,-4], target = 1
输出：2
解释：与 target 最接近的和是 2 (-1 + 2 + 1 = 2) 。
```
提示：
* `3 <= nums.length <= 10^3`
* `-10^3 <= nums[i] <= 10^3`
* `-10^4 <= target <= 10^4`

### 解
最朴素的解法用暴力三重循环搜索所有组合，但是时间复杂度为$O(n^3)$。
* 首先考虑如何在遍历的同时避免重复
  考虑首先对数组进行排序（此处不妨设首先对数组进行非递减排序），并且对遍历增加条件：
  1. 每一重循环所枚举到的下标必须大于上一重循环枚举到的下标
  2. 在同一重循环中，如果当前元素和上一个元素相等，则跳过当前元素
  需要注意的是，如果用三元组`(i, p, q)`来表示遍历中的某一次访问的下标，条件2需要保证的事情是“三元组上每个位置的所有可能取值不会被重复计数”，编写程序时要注意不要让前一重循环的下标移动意外删除掉后一重循环的下标的可能取值。
* 然后考虑如何加速遍历
  最后两重循环可以使用双指针来合并成一重。当第一重循环分别取得下标`i`时，后一重循环初始化为`p = i + 1`, `q = nums.length - 1`。每次遍历可以得到和`sum = nums[i] + nums[p] + nums[q]`，当前已经遍历得到的最优值为`res`。如果有`abs(sum - target) < abs(res - target)`，则`res = sum`，否则考虑如何移动左右指针。
  1. 如果`sum < target`，注意此时肯定有`sum < res < target`，考虑到数组非递减排序，应该让左指针+1.
  2. 如果`sum > target`，注意此时肯定有`sum > res > target`，考虑到数组非递减排序，应该让右指针-1.
  需要注意的是，在移动双指针的时候同样要注意上一条的2。
  在此基础之上，可以进行一定的剪枝操作，详见代码。此时时间复杂度为$O(n^2)$，空间复杂度为$O(n)$。
```java
class Solution {
    public int threeSumClosest(int[] nums, int target) {
        int res = 1145141919;
        if(nums == null || nums.length < 3) return res;
        int n = nums.length, sum = 0;
        Arrays.sort(nums);
        for(int i = 0; i < n - 2; i++) {
            if(i > 0 && nums[i] == nums[i - 1]) continue;
            int p = i + 1, q = n - 1;
            if(nums[i] + nums[i + 1] + nums[i + 2] > res && res > target) break;
            if(nums[i] + nums[n - 2] + nums[n - 1] < res && res < target) continue;
            while(p < q) {
                sum = nums[i] + nums[p] + nums[q];
                if(sum == target) return sum;
                if(Math.abs(sum - target) < Math.abs(res - target)) {
                    res = sum;
                }
                if(sum - target < 0) {
                    p++;
                    while(p < q && nums[p] == nums[p - 1]) p++;
                } else {
                    q--;
                    while(p < q && nums[q] == nums[q + 1]) q--;
                }
            }
        }
        return res;
    }
}
```


## Leetcode 18 四数之和
### 题目描述
https://leetcode-cn.com/problems/4sum

给定一个包含 n 个整数的数组 nums 和一个目标值 target，判断 nums 中是否存在四个元素 a，b，c 和 d ，使得 a + b + c + d 的值与 target 相等？找出所有满足条件且不重复的四元组。答案中不可以包含重复的四元组。
示例：
```
给定数组 nums = [1, 0, -1, 0, -2, 2]，和 target = 0。

满足要求的四元组集合为：
[
  [-1,  0, 0, 1],
  [-2, -1, 1, 2],
  [-2,  0, 0, 2]
]
```

### 解
最朴素的解法是用暴力搜索所有组合，并用哈希表去重。但是遍历的时间复杂度为$O(n^4)$，并且去重较为困难。
* 首先考虑如何在遍历的同时避免重复
  考虑首先对数组进行排序（此处不妨设首先对数组进行非递减排序），并且对遍历增加条件：
  1. 每一重循环所枚举到的下标必须大于上一重循环枚举到的下标
  2. 在同一重循环中，如果当前元素和上一个元素相等，则跳过当前元素
  注意，2不能写成“在同一重循环中，如果当前元素和下一个元素相等，则跳过当前元素”。首先1既保证了下标不会重复，在数组有序的基础上还保证了不会出现“元素相同，顺序不同”的情况。假设4重循环的下标分别为`i, j, p, q`，那么每次遍历可以表示为一个四元组，2是为了保证这个四元组每个位置上的所有可能的取值不会被重复计数。但如果条件是“和下一个元素相等”，则上一重循环会排除掉下一重循环的某个取值，因为下一重循环的下标一定要比上一重循环的下标大。满足以上的条件，则可以认为在遍历过程中不会产生重复的结果。
* 然后考虑如何加速遍历
  前两重循环无法优化，后两重循环可以通过双指针合并为一重。当前两重循环分别取得下标`i, j`时，最后一重循环初始化为`p = i + 1`, `q = nums.length - 1`。每次遍历可以得到和`sum = nums[i] + nums[j] + nums[p] + nums[q]`。
  1. 如果`sum > target`，此时移动左指针只能让`sum`更大，移动右指针可以让`sum`更小，故此时应该让右指针-1。
  2. 如果`sum < target`，基于和1相似的考量，应该让左指针+1。
  需要注意的是，在移动双指针的时候同样要注意上一条的2。
  在此基础之上，可以进行一定的剪枝操作，详见代码。此时时间复杂度为$O(n^3)$，空间复杂度为$O(n^2)$。所有“n数之和等于某个target”的题都可以用这个模型来解决。
```java
class Solution {
    public List<List<Integer>> fourSum(int[] nums, int target) {
        List<List<Integer>> res = new ArrayList<>();
        if(nums == null || nums.length < 4) {
            return res;
        }
        int n = nums.length;
        // need to ensure that there's no duplicate four digits without using hashmap
        // first, sort the array
        Arrays.sort(nums);
        for(int i = 0; i < n - 3; i++) {
            // notice: don't try to write like this: i should be first examined and then jump over duplicate. Same with j.
            // if(i < n - 1 && nums[i] == nums[i + 1]) continue;
            if(i > 0 && nums[i] == nums[i - 1]) continue;
            // do some pruning
            if(nums[i] + nums[i + 1] + nums[i + 2] + nums[i + 3] > target) break;
            if(nums[i] + nums[n - 3] + nums[n - 2] + nums[n - 1] < target) continue;
            for(int j = i + 1; j < n - 2; j++) {
                // notice
                // if(j < n - 1 && nums[j] == nums[j + 1]) continue;
                if(j > i + 1 && nums[j] == nums[j - 1]) continue;
                // do some pruning
                if(nums[i] + nums[j] + nums[j + 1] + nums[j + 2] > target) break;
                if(nums[i] + nums[j] + nums[n - 2] + nums[n - 1] < target) continue;
                // then find the rest two digits using two pointers
                int p = j + 1, q = n - 1;
                while(p < q) {
                    int sum = nums[i] + nums[j] + nums[p] + nums[q];
                    if(sum == target) {
                        res.add(List.of(nums[i], nums[j], nums[p], nums[q]));
                        // it is no problem to write like this, because the value of current p and q is already examined!
                        while(p < q && nums[p] == nums[p + 1]) p++;
                        p++;
                        while(p < q && nums[q] == nums[q - 1]) q--;
                        q--;
                    } else if (sum < target) {
                        p++;
                    } else {
                        q--;
                    }
                }
            }
        }
        return res;
    }
}
```

## Leetcode 923 三数之和的多种可能
### 题目描述
https://leetcode-cn.com/problems/3sum-with-multiplicity

给定一个整数数组`A`，以及一个整数`target`作为目标值，返回满足`i < j < k`且`A[i] + A[j] + A[k] == target`的元组`i, j, k`的数量。

由于结果会非常大，请返回 结果除以`10^9 + 7`的余数。

 

示例 1：
```
输入：A = [1,1,2,2,3,3,4,4,5,5], target = 8
输出：20
解释：
按值枚举（A[i]，A[j]，A[k]）：
(1, 2, 5) 出现 8 次；
(1, 3, 4) 出现 8 次；
(2, 2, 4) 出现 2 次；
(2, 3, 3) 出现 2 次。
```
示例 2：
```
输入：A = [1,1,2,2,2,2], target = 5
输出：12
解释：
A[i] = 1，A[j] = A[k] = 2 出现 12 次：
我们从 [1,1] 中选择一个 1，有 2 种情况，
从 [2,2,2,2] 中选出两个 2，有 6 种情况。
```

提示：
```
3 <= A.length <= 3000
0 <= A[i] <= 100
0 <= target <= 300
```

### 解1 双指针
基本思路与#18类似。先将数组进行排序，遍历数组下标，对于每个`i`，用双指针来完成寻找`A[i] + A[j] + A[k] == target`的子任务。本题的难点在于如何完成对满足条件的重复元组进行计数的任务，在处理边界条件的时候要特别注意。设任意一个满足条件的下标组合表示为`i, j, k`：
* 当`A[j] != A[k]`时，可以使用双指针对值为`A[i]`和`A[j]`的元素进行计数，设计数结果为`m, n`，那么对于当前下标`i`和数字组合`A[i], A[j], A[k]`，可能性组合一共有`m * n`种。
* 当`A[j] == A[k]`时，由于数组是排序的，可以得知下标`j, j-1, ..., k`处的值都相等，因此此时需要计算的是“从这些下标中任取两个，能有多少种取法”，即$\binom{k - j + 1}{2}$。
* 需要注意的是，本题在最外层循环无法通过“对相同的值进行计数”来进行加速，因为有可能多算。例如`[1, 1, 1, 1], target = 3`，当`i = 0`时，计数结果是$\binom{3}{2} = 3$，但是当`i = 1`时，尽管有`A[1] == A[0]`，但是此时技术结果应该为$1$。本解法的时间复杂度为$O(N^2)$，空间复杂度为$O(1)$。
```java
class Solution {
    public int threeSumMulti(int[] A, int target) {
        int res = 0, sum = 0, n = A.length, mod = 1000000007;
        if(A == null || A.length < 3) return res;
        Arrays.sort(A);
        for(int i = 0; i < n - 2; i++) {
            if(A[i] + A[i + 1] + A[i + 2] > target) break;
            if(A[i] + A[n - 2] + A[n - 1] < target) continue;
            int p = i + 1, q = n - 1;
            while(p < q) {
                sum = A[i] + A[p] + A[q];
                if(sum < target) {
                    p++;
                } else if(sum > target) {
                    q--;
                } else if(A[p] != A[q]) {
                    int pCount = 1, qCount = 1;
                    while(p < q && A[p] == A[p + 1]) {
                        pCount++;
                        p++;
                    }
                    while(p < q && A[q] == A[q - 1]) {
                        qCount++;
                        q--;
                    }
                    p++;
                    q--;
                    res = (res + pCount * qCount) % mod;
                } else {
                    res = (res + (q - p + 1) * (q - p) / 2) % mod;
                    break;
                }
            }
        }
        return res;
    }
}
```

### 解2 数学法
思路是对数组`A`中出现的每一个数字进行计数，之后对每种`x + y + z == target`的组合进行计数。设`x`在`A`中出现了`count[x]`次，则：
* 若`x`， `y`， `z`各不相同，则这种组合会出现$count[x] \times count[y] \times count[z]$次。为了避免重复计数，应该保证`x < y < z`。
* 若`x == y`，且`x != z`，则这种组合会出现$\binom{count[x]}{2} \times count[z]$次（`x != y`且`x == z`的情况类似可以计算）。为了避免重复计数，应该保证`x == y < z`。
* 若`x`，`y`，`z`都相同，则这种组合会出现$\binom{count[x]}{3}$次。
若`A`中出现了`W`个不同的数字，则该算法的时间复杂度为$O(N + W^2)$，空间复杂度为$O(W)$。
* 需要注意的是，计算的中间结果可能会很大，要谨防溢出。
```java
class Solution {
    public int threeSumMulti(int[] A, int target) {
        long res = 0;
        int n = A.length, mod = 1000000007;
        long[] count = new long[101];
        if(A == null || A.length < 3) return (int)res;
        for(int i = 0; i < n; i++) {
            count[A[i]]++;
        }
        // all different
        for(int x = 0; x < 101; x++) {
            for(int y = x + 1; y < 101; y++) {
                int z = target - x - y;
                if(y < z && z <= 100) {
                    res += count[x] * count[y] * count[z];
                    res = res % mod;
                }
            }
        }
        // two different
        for(int x = 0; x < 101; x++) {
            int z = target - 2 * x;
            if(x < z && z <= 100) {
                // count[x] - 1 will be zero when count[x] < 2.
                res += count[x] * (count[x] - 1) / 2 * count[z];
                res = res % mod;
            }
        }
        for(int x = 0; x < 101; x++) {
            if((target - x) % 2 == 0) {
                int y = (target - x) / 2;
                if(x < y && y <= 100) {
                    res += count[x] * count[y] * (count[y] - 1) / 2;
                    res = res % mod;
                }
            }
        }
        // all same
        if(target % 3 == 0) {
            int x = target / 3;
            if(x >= 0 && x <= 100) {
                res += count[x] * (count[x] - 1) * (count[x] - 2) / 6;
                res = res % mod;
            }
        }
        return (int)res;
    }
}
```

# 滑动窗口(Sliding Window)问题
## Leetcode #3 无重复字符的最长字串
### 题目描述
https://leetcode-cn.com/problems/longest-substring-without-repeating-characters

给定一个字符串，请你找出其中不含有重复字符的 最长子串 的长度。
示例 1:
```
输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```
示例 2:
```
输入: "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
```
示例 3:
```
输入: "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
```
### 解
很显然，判断一个字符串是否含有重复字符可以用集合或者哈希表实现，我们考虑如何更快地寻找最长的子串。

假设我们在长度为`n`的字符串中找到了一个起点为`i`，终点为`j - 1`的子串，其中`j`处的字符和`k`处的字符相等，此处显然有`i <= k < j`。如果`j < n`，说明我们应该继续尝试寻找更长的字符串。显然这个更长的子串起点不可能在区间`[i, k]`之内，假设某个子串的起点在区间`[i, k]`之内，那么这个子串在`j`处就会碰到重复字符，得不到比原来更长的字符串。因此，可以考虑此时让右指针`i`逐步移动到`k + 1`的位置，为了能够方便地得到`k`的位置，我们需要使用哈希表记录每个字符出现的位置。`i`在移动时需要清除掉哈希表中记录的出现过的字符，因为它们暂时没有在下一个子串中出现。本算法的时间复杂度为$O(n)$，因为左指针`i`和右指针`j`都遍历了一次字符串。

假设所有可能出现过的字符有`M`种，那么空间复杂度为$O(M)$。默认字符集为ASCII能表示的字符的话，有$M = 128$。
```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        if(s == null) return 0;
        if(s.length() == 1) return 1;
        int n = s.length(), res = 0, i = 0, j = 0, cacheTail = 0, last = 0;
        HashMap<Character, Integer> m = new HashMap<>();
        while(i < n) {
            while(j < n && (!m.containsKey(s.charAt(j)) || m.get(s.charAt(j)) < cacheTail)) {
                m.put(s.charAt(j), j);
                j++;
            }
            res = (j - i + last > res) ? j - i + last : res;
            if(j >= n) {
                break;
            } else {
                cacheTail = m.get(s.charAt(j)) + 1;
                last = j - 1 - m.get(s.charAt(j));
                i = j;
            }
        }
        return res;
    }
}
```
现在我们考虑如何在以上算法的基础上进一步进行优化。

首先，如果字符是ASCII字符集，那哈希表可以使用数组来实现（而不是使用Java的`HashMap`），运行速度会有显著提升。

其次，我们让左指针`i`一步一步地移动，目的是为了删除掉哈希表中`[i, k]`上的字符记录，因为这些字符暂时还没有在下一个子串中出现。我们可以使用一种速度更快的方式：不手动删除这些记录，而是记录并更新下一个字符串的开始坐标。如果从哈希表中获得的字符记录其下标小于当前字符串的开始坐标，那么这个字符记录就应该被视为无效，并且可以在后续的匹配中被覆盖。这样一来，左指针`i`甚至能够直接跳跃到`j`的位置上，只需要记录下`(k, j)`的长度`last`，并在下一个子串需要进行长度计算的时候加上(`j - i + last`)即可。
```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        if(s == null) return 0;
        if(s.length() == 1) return 1;
        int n = s.length(), res = 0, i = 0, j = 0, cacheTail = 0, last = 0;
        //HashMap<Character, Integer> m = new HashMap<>();
        int[] map = new int[128];
        for(int k = 0; k < 128; k++) {
            map[k] = -1;
        }
        while(i < n) {
            // Add tail to disable the map records
            while(j < n && (map[(int)s.charAt(j)] == -1 || map[(int)s.charAt(j)] < cacheTail)) {
                map[(int)s.charAt(j)] = j;
                j++;
            }
            res = (j - i + last > res) ? j - i + last : res;
            if(j >= n) {
                break;
            } else {
                cacheTail = map[(int)s.charAt(j)] + 1;
                last = j - 1 - map[s.charAt(j)];
                i = j;
            }
        }
        return res;
    }
}
```

## Leetcode #26 删除排序数组中的重复项
### 题目描述
https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array

给定一个排序数组，你需要在 原地 删除重复出现的元素，使得每个元素只出现一次，返回移除后数组的新长度。

不要使用额外的数组空间，你必须在 原地 修改输入数组 并在使用 O(1) 额外空间的条件下完成。

示例 1:
```
给定数组 nums = [1,1,2], 

函数应该返回新的长度 2, 并且原数组 nums 的前两个元素被修改为 1, 2。 

你不需要考虑数组中超出新长度后面的元素。
```
示例 2:
```
给定 nums = [0,0,1,1,1,2,2,3,3,4],

函数应该返回新的长度 5, 并且原数组 nums 的前五个元素被修改为 0, 1, 2, 3, 4。

你不需要考虑数组中超出新长度后面的元素。
```
本题只需要将最终留下的元素移动到数组的前面，判题程序会输出原数组从0到新长度-1的所有元素。
### 解
本题可以使用类似滑动窗口的思想：先移动右指针，到右指针无法继续移动时再接着移动左指针。设左指针为`i`，右指针为`j`，则
* 当`nums[i] == nums[j]`时，可以知道此时右指针`j`如果加入到数组`[0, i]`中肯定会出现重复元素，此时需要向右移动`j`，直到遇到不重复的项。
* 当`nums[i] != nums[j]`时，跳过重复项的工作已经结束，此时`nums[j]`可以加入到不重复的新数组中去。因此我们需要让`i`加1，然后将`nums[j]`的值复制到`i`的位置上去。
* 当右指针`j`遍历完整个原数组，原数组中所有不重复的数字都已经找到，此时数组`[0,i]`是一个元素不重复的有序数组，其长度为`i + 1`。
设数组长度为$N$，则算法的时间复杂度为$O(N)$，空间复杂度为$O(1)$。
```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int i = 0;
        if(nums == null) return 0;
        for(int j = 1; j < nums.length; j++) {
            if(nums[i] != nums[j]) {
                i++;
                nums[i] = nums[j];
            }
        }
        return i + 1;
    }
}
```

## Leetcode #27 移除元素
### 题目描述
https://leetcode-cn.com/problems/remove-element

给你一个数组`nums`和一个值`val`，你需要 原地 移除所有数值等于`val`的元素，并返回移除后数组的新长度。

不要使用额外的数组空间，你必须仅使用$O(1)$额外空间并 原地 修改输入数组。

元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。

示例 1:
```
给定 nums = [3,2,2,3], val = 3,

函数应该返回新的长度 2, 并且 nums 中的前两个元素均为 2。

你不需要考虑数组中超出新长度后面的元素。
```
示例 2:
```
给定 nums = [0,1,2,2,3,0,4,2], val = 2,

函数应该返回新的长度 5, 并且 nums 中的前五个元素为 0, 1, 3, 0, 4。

注意这五个元素可为任意顺序。

你不需要考虑数组中超出新长度后面的元素。
```
### 解1
非常朴素，记录下当前位置之前出现了多少个值等于`val`的元素，设其为`valCount`，一直做`nums[i - valCount] = nums[i]`的操作即可，最终长度为`nums.length - valCount`。设数组长度为$N$，该算法时间复杂度$O(N)$，空间复杂度$O(1)$。代码略。
### 解2 双指针法
本题的思路和#26类似。设左指针为`i`，右指针为`j`。
* 当`nums[j] == val`时，代表此处的元素不应该被保留，因此继续移动右指针`j`。
* 当`nums[j] != val`时，此处的元素应该被保留，因此将`j`处的元素`nums[j]`复制到`i`处，然后移动左指针`i`。
设数组长度为$N$，该算法时间复杂度$O(N)$，空间复杂度$O(1)$。
```java
class Solution {
    public int removeElement(int[] nums, int val) {
        int i = 0, n = nums.length;
        for(int j = 0; j < n; j++) {
            if(nums[j] != val) {
                nums[i] = nums[j];
                i++;
            }
        }
        return i;
    }
}
```

## Leetcode #28 实现strStr()
### 题目描述
https://leetcode-cn.com/problems/implement-strstr/

给定一个`haystack`字符串和一个`needle`字符串，在`haystack`字符串中找出`needle`字符串出现的第一个位置 (从0开始)。如果不存在，则返回`-1`。

示例 1:
```
输入: haystack = "hello", needle = "ll"
输出: 2
```
示例 2:
```
输入: haystack = "aaaaa", needle = "bba"
输出: -1
```
说明:

对于本题而言，当 needle 是空字符串时我们应当返回 0 。这与C语言的 strstr() 以及 Java的 indexOf() 定义相符。

### 解1 KMP算法
难点在于如何正确而快速地构建PMT表（next数组与PMT表都可以拿来用，区别不会特别大）。
* 当`needle.charAt(i) == needle.charAt(pmt[i - 1])`时，表明新加入的字符使得相等前后缀的长度增加了，因此`pmt[i] = pmt[i - 1] + 1`
* 当`needle.charAt(i) != needle.charAt(pmt[i - 1])`时，表明新加入的字符串无法使得相等前后缀的长度增加，因此我们应该试图减小相等前后缀的长度。设`now = pmt[i - 1]`，可知`[0, now - 1]`和`[i - now, i - 1]`是相等的。要求`[0, now - 1]`和`[i - now + 1, i]`的相等前后缀，相当于将`now`位置上的元素替换为`i`位置上的元素后，再求`[0, now]`的相等前后缀长度。因此，可以不停地进行`now = pmt[now - 1]`操作，直到`needle.charAt(i) == needle.charAt(pmt[now - 1])`或`now == 0`为止。
* 设目标串的长度为$N$, 模式串的长度为$M$，该算法的时间复杂度为$O(N + M)$，空间复杂度为$O(M)$。
```java
class Solution {
    public int strStr(String haystack, String needle) {
        if(needle == null || needle.length() == 0) return 0;
        if(haystack == null || haystack.length() < needle.length()) return -1;
        int n = haystack.length(), m = needle.length(), j = 0;
        int[] pmt = new int[needle.length()];
        pmt[0] = 0;
        // construct pmt array
        for(int i = 1; i < m;) {
            if(needle.charAt(i) == needle.charAt(pmt[i - 1])) {
                pmt[i] = pmt[i - 1] + 1;
                i++;
            } else {
                j = pmt[i - 1];
                while(j != 0 && needle.charAt(i) != needle.charAt(pmt[j - 1])) {
                    j = pmt[j - 1];
                }
                pmt[i] = j;
                i++;
            }
        }
        j = 0;
        // match
        for(int i = 0; i < n; ) {
            if(j == m - 1 && haystack.charAt(i) == needle.charAt(j)) {
                return i - m + 1;
            } else if(haystack.charAt(i) != needle.charAt(j)) {
                j = j > 0 ? pmt[j - 1] : 0;
                if(j == 0 && haystack.charAt(i) != needle.charAt(j)) {
                    i++;
                }
            } else {
                j++;
                i++;
            }
        }
        return -1;
    }
}
```
### 解2 滑动窗口 + Rabin Karp算法
基本思路：先生成模式串`needle`的哈希码，然后使用滑动窗口方法，每次窗口滑动都重新计算窗口内子串的哈希码，将该哈希码和模式串的哈希码进行比较。问题在于，一般情况下生成一个长度为`L`的数组的哈希码需要的时间复杂度为$O(L)$，在使用滑动窗口方法的情况下能不能找到更快的计算哈希码的方法？  
**滚动哈希的大体思路**  
每次窗口滑动都会有一个元素进，一个元素出。设所有可能出现的字符共有$a$种，则可以对窗口字符串中的每个字符实行$a$进制编码。设窗口长度为$L$，窗口中的字符为$c_i$，那么滑动窗口内字符串的哈希值可以用以下公式计算：
$$h_k = \sum_{i=0}^{L-1}c_ia^{L-1-i} = c_0a^{L-1} + c_1a^{L-2} + ... + c_{L - 2}a^1 + c_{L-1}a^0$$
当窗口进行滑动时，最左边的字符移出窗口，最右边加入一个新字符，则滑动后窗口内子串的哈希值可以通过滑动前的哈希值来计算：
$$h_{k+1} = (h_k - c_0a^{L-1}) \times a + c_{L} \times a^0 = h_ka - c_0a^L + c_{L}$$
需要注意的是$a^L$有可能是个非常大的值，可以通过在计算过程中取模来防止溢出。设目标串的长度为$N$，则该算法的时间复杂度为$O(N)$，空间复杂度为$O(1)$。  
更具体的解释和举例可以参见：https://leetcode-cn.com/problems/implement-strstr/solution/shi-xian-strstr-by-leetcode/
```java
class Solution {
    public int strStr(String haystack, String needle) {
        if(needle == null) return 0;
        if(haystack == null || haystack.length() < needle.length()) return -1;
        long RKHash = 0, targetHash = 0, mod = (long)Math.pow(2, 31), pow = 1;
        char[] source = haystack.toCharArray(), pattern = needle.toCharArray();
        int n = haystack.length(), m = needle.length();
        // first, calculate the hash value of needle and first substring
        for(int i = 0; i < m; i++) {
            targetHash = (targetHash * 26 + (int)(pattern[i] - 97)) % mod;
            RKHash = (RKHash * 26 + (int)(source[i] - 97)) % mod;
            pow = (pow * 26) % mod;
        }
        if(RKHash == targetHash) return 0;
        // second, calculate the rolling hash value of substrings
        for(int i = 1; i < n - m + 1; i++) {
            RKHash = (RKHash * 26 - ((int)(source[i - 1]) - 97) * pow + (int)source[i + m - 1] - 97) % mod;
            if(RKHash == targetHash)
                return i;
        }
        return -1;
    }
}
```



# 快慢指针问题
* 快慢指针大概有两种思路：一是两个指针同时出发，步长不同；二是两个指针先后出发，步长相同。快慢指针经常被用在链表相关的问题中。
## Leetcode #19 删除链表的倒数第N个节点
### 题目描述
https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list

给定一个链表，删除链表的倒数第 n 个节点，并且返回链表的头结点。

示例：
```
给定一个链表: 1->2->3->4->5, 和 n = 2.

当删除了倒数第二个节点后，链表变为 1->2->3->5.
```
说明：

给定的 n 保证是有效的。

### 解
一道比较朴素的可以用快慢指针解决的问题，先让快指针`p`向前移动`n + 1`步，之后慢指针`q`再从第一个节点开始移动，当`p`移动到最后一个节点的后继(`null`)时，`q`恰好为待删除节点的前驱节点。设链表长度为`N`，则时间复杂度为$O(N)$，空间复杂度为$O(1)$。
* 巧妙地构造虚拟头节点，可以更方便地遍历不带额外头节点的链表
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode res = new ListNode(), p = res, q = res;
        res.next = head;
        while(n + 1 > 0 && p != null) {
            p = p.next;
            n--;
        }
        while(p != null) {
            p = p.next;
            q = q.next;
        }
        q.next = q.next.next;
        return res.next;
    }
}
```

## Leetcode #142 环形链表II
### 题目描述
https://leetcode-cn.com/problems/linked-list-cycle-ii

因为有图，本题不放出题目的详细描述，题目大意是判断链表中是否有环，如果有环则返回环的入口节点，没有则返回`null`。注意本题不允许对链表进行修改，也希望尽量不用额外的存储空间。

### 解
判断链表是否有环是一个非常经典的可以用快慢指针来解决的问题，但是在本题中还需要想到找出环的入口节点的办法。

首先，还是可以用快慢指针法初步确定链表中有没有环。假设慢指针为`p`，一次向前走一步，总步数为`pStep`，快指针为`q`，一次向前走两步，总步数为`qStep`。
* 当`p == q`时，说明链表中有环，同时我们可以得知环的长度为快慢指针的步数差`qStep - pStep`。
* 此时找入口节点的问题可以转化为“找链表的倒数第n个节点”问题：快慢指针每次都只向前走一步，快指针先走`qStep - pStep`步，然后慢指针从头节点出发和快指针一同前进。当快慢指针相遇时，快慢指针所处的节点就是环的入口节点。
* 但是且慢，得到`qStep - pStep`之后并不需要从头节点开始重新构造快慢指针。注意到`qStep == 2 * pStep`且`qStep - pStep == qStep - pStep == pStep`，也就是说`p`当前已经前进了`qStep - pStep`步，是一个现成的快指针，因此只需要让`q`指针回到头节点。这样就完成了快慢指针的“互换”。
假设链表有$N$个节点，则该算法的时间复杂度为$O(N)$，空间复杂度为$O(1)$。
```java
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode p = head, q = head;
        //int cycleLength = 0;
        int pStep = 0, qStep = 0;
        while(p != null && q != null && q.next != null) {
            p = p.next;
            pStep++;
            q = q.next.next;
            qStep += 2;
            if(p == q) {
                break;
            }
        }
        if(p == null || q == null || q.next == null) return null;
        // cycleLength = qStep - pStep;
        // Because qStep = 2 * pStep, we can infer that pStep = cycleLength, there's no need to
        // reconstruct fast and slow pointer. Now q is the slow one and p is the fast one.
        // p = head;
        q = head;
        // while(cycleLength-- > 0) {
        //     q = q.next;
        // }
        while(p != q) {
            p = p.next;
            q = q.next;
        }
        return p;
    }
}
```
