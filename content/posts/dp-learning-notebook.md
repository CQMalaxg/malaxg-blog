---

date = '2024-10-31T10:00:00+08:00'
draft = false
title = '动态规划学习笔记'

+++

# 前言

从考研的时候就想说好好地学学算法题，一直到研究生入学了，还是没有咋学，现在从头开始学学动态规划，希望不会烂尾！！

动态规划也就是大家常说地DP，大家都说DP是没有捷径的，只有猛猛刷题，光看人家解出来，是没有用的，得需要自己动手尝试！找了一份题单，从头开始学DP！在刷题的过程当中领悟DP的精髓。

# 动态规划入门

## 从记忆化搜索到递推

大佬的视频讲解：[动态规划入门：从记忆化搜索到递推_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Xj411K7oF/?vd_source=83f6b05813653ec99200e639b647b9af)

动态规划的核心就是**状态定义**和**状态转移方程**。

### **198. 打家劫舍**

**问题描述**

你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，**如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警**。

给定一个代表每个房屋存放金额的非负整数数组，计算你 **不触动警报装置的情况下** ，一夜之内能够偷窃到的最高金额。

**示例**

- 示例1

  输入：[1,2,3,1] 输出：4 解释：偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。     偷窃到的最高金额 = 1 + 3 = 4 。

- 示例2

  输入：[2,7,9,3,1] 输出：12 解释：偷窃 1 号房屋 (金额 = 2), 偷窃 3 号房屋 (金额 = 9)，接着偷窃 5 号房屋 (金额 = 1)。 偷窃到的最高金额 = 2 + 9 + 1 = 12 。

**解题思路**

这个题可以先看成一个回溯题，把一个大问题变成规模更小的子问题，从第一个房子或者最后一个房子开始思考比较容易（因为它们收到的约束最小）。

比如考虑最后一个房子：

- 不选
  - 问题就变成了n-1个房子的问题
- 选
  - 问题就变成了n-2个房子的问题

不断这样下去就可以得到一个搜索树！

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/2aa8704c-741a-4edc-a032-c1b55e765d46/image.png)

$$ dfs(i) = max(dfs(i-1),dfs(i-2)+nums[i]) $$

$dfs(i)$的含义指的是从前i个房子中得到的最大金额和。

先写回溯的代码：

```python
class Solution(object):
    def rob(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        n = len(nums)
        def dfs(i):
            if i<0: #没有房子可以选
                return  0
            res = max(dfs(i-1),dfs(i-2)+nums[i])
            return res
        return dfs(n-1)
```

回溯的时间复杂度是指数级别的，这样写会超时！接下来优化这份代码！

通过观察搜索树可以发现像dfs(2)这样的值会被重复计算，所以可以考虑将这样重复计算的值存入一个cache数组或者hash表中，这样在第二次算的时候就可以直接返回cache里面保存的结果了。优化后的搜索树长这样！

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/bc8fe2b1-809d-4550-9bbb-818ecb3cea9b/image.png)

$$ 递归搜索+保存计算结果= 记忆化搜索 $$

```python
class Solution(object):
    def rob(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        n = len(nums)
        cache = [-1] * n
        def dfs(i):
            if i<0: #没有房子可以选
                return  0
            if cache[i] != -1:
                return cache[i]
            res = max(dfs(i-1),dfs(i-2)+nums[i])
            cache[i] = res
            return res
        return dfs(n-1)
```

$$ 自顶向下算 = 记忆化搜索 \\ 自底向上算 = 递推 $$

怎么把记忆化搜索改成递推呢？方法就是把dfs改成数组，把递归改成循环就好了，这样写需要对i=0，i=1的情况特殊处理！因为这里会产生负数下标

$$ \begin{aligned}&dfs(i)=\max\left(dfs(i-1),dfs(i-2)+nums[i]\right)\\&f[i]=\max\left(f[i-1],f[i-2]+nums[i]\right)\\&f[i+2]=\max\left(f[i+1],f[i]+nums[i]\right)\end{aligned} $$

```python
class Solution(object):
    def rob(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        n = len(nums)
        f = [0] * (n+2)
        for i,x in enumerate(nums):
            f[i+2] = max(f[i+1],f[i] + x)
        return f[n+1]
```

这个代码的空间复杂度依然是O(n)，怎么改成O(1)？

换一个角度，`f[i]`是当前要算的，`f[i-1]`是上一个算出来的，`f[i-2]`是上上一个算出来的。也就是说要计算一个`f[i]`，只需要知道它的上一个状态和上上一个状态的值即可，对于`f[i+1]`来说，`f[i]`就是它的上一个状态，`f[i-1]`就是它的上上一个状态。那么用`f[0]`表示上上一个，`f[1]`表示上一个，就可以变成下面这个式子

$$ newF = max(f_1,f_0+nums[i]\\ f_0 = f_1 \\ f_1 = newF $$

```python
class Solution(object):
    def rob(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        n = len(nums)
        f0 = f1 =0
        for i,x in enumerate(nums):
            newf = max(f1,f0 + x)
            f0 = f1
            f1 = newf
        return newf
```

## 背包问题

0-1背包和完全背包是非常重要的DP模型，可以说它们是选或不选这个思想的代表。

### 0-1背包

有n个物品，第i个物品的体积为$w[i]$，价值为$v[i]$，每个物品至多选一个，求体积和不超过capacity的最大价值和。

先从回溯开始思考，每一个物品都对应着选和不选。对于第i个物品来说，不选的话，容量不变；选的话容量变小。这样就确定了递归参数中的$i$和$c$

$$ dfs(i,c)=max(dfs(i-1,c),dfs(i-1,c-w[i])+v[i]) $$

其对应的子问题就是在剩余容量为$c$时，从前i个物品中得到的最大价值和。

对于下一个子问题，下面分类讨论“

1. 不选。在剩余容量为$c$时，从前$i-1$个物品中得到的最大价值和
2. 选。在剩余容量为$c-w[i]$时，从前$i-1$个物品中得到的最大价值和

```python
 def zero_one_knapsack(capacity,w,v):
 #capacity:背包容量
 #w[i]:第 i 个物品的体积
 #v[i]:第 i 个物品的价值
    n=len(w)
    @cache#可替换为cache数组
    def dfs(i,c):
        if i<0:
            return 0
        if c < w[i]:#物品i已经超过了剩余容量c
            return dfs(i-1,c) #不选
        return max(dfs(i-1,c),dfs(i-1,c-w[i])+v[i])
    return dfs(n-1,capacity)
```

### **494. 目标和**

**问题描述**

给你一个非负整数数组 `nums` 和一个整数 `target` 。

向数组中的每个整数前添加 `'+'` 或 `'-'` ，然后串联起所有整数，可以构造一个 **表达式** ：

- 例如，`nums = [2, 1]` ，可以在 `2` 之前添加 `'+'` ，在 `1` 之前添加 `'-'` ，然后串联起来得到表达式 `"+2-1"` 。

返回可以通过上述方法构造的、运算结果等于 `target` 的不同 **表达式** 的数目。

**示例**

**示例1**

> 输入：nums = [1,1,1,1,1], target = 3 输出：5 解释：一共有 5 种方法让最终目标和为 3 。 -1 + 1 + 1 + 1 + 1 = 3 +1 - 1 + 1 + 1 + 1 = 3 +1 + 1 - 1 + 1 + 1 = 3 +1 + 1 + 1 - 1 + 1 = 3 +1 + 1 + 1 + 1 - 1 = 3

**示例2**

> 输入：nums = [1], target = 1 输出：1

**解题思路**

记所有数之和为`s`，记添加正号的所有数之和为`p` ，记添加负号的所有数之和为`s-p` ，记`target`为`t` 。那么最后的`t= p - (s-p)` 。则`2p = s+t`，则`p = (s+t)/2` 。

所以这个问题就可以转化为：从`nums`中选择一些数字，让他们之和等于`(s+t)/2` 的方案数。

需要注意的是：

1. `s+t`需要除以`2`，所以它一定是偶数。
2. 由于`nums[i]` 是非负数，所以无论怎么选，`s+t`都不可能为负数。

```python
class Solution:
    def findTargetSumWays(self, nums: List[int], target: int) -> int:
        #添加正号的数和：p
        #添加负号的数和：s-p  ——>s = sum(nums)
        #target：p - (s - p)=t
        #2p = s+t
        #p = (s+t)/2
        target += sum(nums) #s+t
        if target < 0 or target%2:
            return  0
        target //= 2 
        n=len(nums)
        @cache
        def dfs(i,c):
            if i<0:
                return 1 if c==0 else 0
            if c < nums[i]:
                return dfs(i-1,c)
            return dfs(i-1,c) + dfs(i-1,c-nums[i])
        return dfs(n-1,target)
```

这是记忆化搜索的写法，那么能不能把空间复杂度进一步优化呢。 把这个记忆化搜索改成递推看看。记忆化搜索可以1比1的翻译成递推，做法就是把`dfs`改成`f`数组，把递归改成循环就好了。

$$ dfs(i,c) = dfs(i-1,c)+dfs(i-1,c-w[i]) \\ f[i][c] = f[i-1,c] + f[i-1][c-w[i]]\\ f[i+1][c] = f[i][c]+ f[i][c-w[i]] $$

为了让避免出现负数下标，把`f[i]`改成`f[i+1]` ，`f[i-1]`改成`f[i]` 。

```python
class Solution:
    def findTargetSumWays(self, nums: List[int], target: int) -> int:
        target += sum(nums) #s+t
        if target < 0 or target%2:
            return  0
        target //= 2 
        n=len(nums)

        f = [[0]*(target+1) for _ in range(n+1)] #初始化f数组
        f[0][0] = 1

        for i,x in enumerate(nums):
            for c in range(target+1):
                if c<x:
                    f[i+1][c] = f[i][c]
                else:
                    f[i+1][c] = f[i][c]+f[i][c-x]
        return f[n][target]
```

改成递推以后，看看能不能做空间上的优化，先讲第一种。

每次把`f[i+1]`算完以后，后面就不会用到`f[i]`了，也就是说，每时每刻只有两个数组种的元素在参与状态转移。就干脆只用两个数组，比如把`f[1]`算完以后，在计算`f[2]`的时候就直接把结果天道`f[0]`当中计算`f[3]`的时候，就把结果填到`f[1]`当中（把所有`i`和`i+1`都模2即可）。

# 参考

[分享丨【题单】动态规划（入门/背包/状态机/划分/区间/状压/数位/树形/数据结构优化） - 力扣（LeetCode）](https://leetcode.cn/circle/discuss/tXLS3i/)