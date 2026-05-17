+++

date = '2024-03-25T20:52:27+08:00'
draft = false
title = 'LeetCode题库'

+++

# LeetCode题库

## 3.无重复字符最长子串

[3. 无重复字符的最长子串 - 力扣（LeetCode）](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

### 问题描述

> 给定一个字符串`s`，请你找出其中不含有重复字符的最长子串的长度

### 示例

- **示例 1**

  输入:s = "abcabcbb" 输出:3 解释: 因为无重复字符的最长子串是"abc"，所以其长度为 3。

- **示例 2**

  输入:s = "bbbbb" 输出:1 解释:因为无重复字符的最长子串是"b"，所以其长度为 1。

- **示例 3**

  输入:s = "pwwkew" 输出:3 解释:因为无重复字符的最长子串是"wke"，所以其长度为 3。请注意，你的答案必须是子串的长度，"pwke" 是一个子序列，不是子串。

### 解题思路

**错误想法**：最开始想成了计算不同字符总数，把字符映射到一个辅助数组里面，然后出现标记为1，最后计算数组当中有多少个1就好了

```c
int lengthOfLongestSubstring(char* s) {
    int i;
    int temp[26]={0};
    for(i=0;s[i]!='\\0';i++){
        temp[s[i] - 'a'] = 1;
    }

    int sum=0;
    for(int j=0;j<26;j++){
        sum += temp[j];
    }
    return sum;
}
```

对于字符串 **`"abcabcbb"`**，不同字符有**`'a'`**, \**`'b'`\**, \**`'c'`\**，所以 \**`sum`\** 会是3，而不是题目要求的最长无重复字符子串（**`"abc"`**）的长度，即3。 **正确思路**

同样采用辅助数组的方法解决，采用滑动窗口算法计算无重复字符子串的长度，使用了一个数组 **`lastSeen`** 来记录每个字符最后一次出现的位置，以及一个滑动窗口 **`start`** 来标记当前考虑的子串的开始位置，将**`lastSeen`**然后全部初始化为-1。滑动窗口遍历字符串。

滑动窗口算法的核心思想是维护一个窗口，这个窗口内的所有字符都是不重复的。窗口在字符串上滑动，即随着算法的执行逐渐从字符串的一端移动到另一端。窗口的大小（即长度）会根据遇到的字符动态调整。

向右移动窗口的右边界以包含更多的字符，直到遇到一个在当前窗口内已存在的字符。每次移动时，更新窗口内字符的最后出现位置。当我们遇到一个重复的字符时，这意味着当前窗口内已经存在该字符。为了维持窗口内字符的唯一性，我们需要移动窗口的左边界。

左边界应该移动到当前重复字符的上一个出现位置之后的位置。这样做可以确保窗口内不包含重复的字符。在每一步中，都检查当前窗口的大小，并与已知的最大窗口大小进行比较。如果当前窗口更大，则更新最大窗口大小。继续向右移动窗口的右边界并重复以上过程，直到遍历完整个字符串。

```c
int lengthOfLongestSubstring(char* s) {
    int maxLen = 0, start = 0;
    int lastSeen[128]; // 假设ASCII字符
    memset(lastSeen, -1, sizeof(lastSeen)); // 初始化为-1

    for (int i = 0; s[i] != '\\0'; i++) {
        if (lastSeen[s[i]] >= start) {
            start = lastSeen[s[i]] + 1;
        }
        lastSeen[s[i]] = i;
        maxLen = (i - start + 1) > maxLen ? (i - start + 1) : maxLen;
    }

    return maxLen;
}
```

**逐步分析**

- 开始时，窗口为空。随着窗口向右移动，窗口内的字符分别变为 **`'a'`**, **`'b'`**, **`'c'`**，此时都没有重复，最大长度更新为3。
- 当窗口试图包含第二个 **`'a'`** 时，我们发现 **`'a'`** 在窗口内重复了。为了保持窗口内的字符唯一，我们移动窗口的左边界到第一个 **`'a'`** 的右侧，现在窗口内的字符是 **`'b'`**, **`'c'`**, **`'a'`**。
- 继续这个过程，每次遇到重复字符时调整窗口的大小和位置，同时更新最大长度。

### 题解

[**数组hash+滑动窗口**](https://leetcode.cn/problems/longest-substring-without-repeating-characters/solutions/1192065/wu-zhong-fu-zi-fu-de-zui-chang-zi-chuan-u22kb/)

- **`left`** 和 **`right`** 分别表示滑动窗口的左右边界索引，初始都设为0。
- **`max`** 用来记录遍历过程中遇到的最大窗口大小（即无重复字符子串的最大长度）。
- **`len`** 是输入字符串的长度。
- **`haveSameChar`** 用于标记窗口扩展时是否遇到重复字符。

通过循环逐步移动**`right`**指针来扩展窗口，直到字符串末尾。在每一步中，都会检查从**`left`**到**`right`**窗口内是否有与**`s[right]`**相同的字符。如果找到重复字符，将**`haveSameChar`**设置为1，并记录重复字符的位置**`j`**。如果发现重复字符（**`haveSameChar == 1`**），窗口的左边界**`left`**更新为重复字符下一位置**`j + 1`**（这一步保证了窗口内始终维护着无重复字符的子串）。在每次迭代中，比较当前最大长度**`max`**与当前窗口大小**`right - left + 1`**，并更新**`max`**。

```c
int lengthOfLongestSubstring(char * s)
{
    //类似于hash的思想
    //滑动窗口维护
    int left = 0;
    int right = 0;
    int max = 0;
    int i,j;
    int len = strlen(s);
    int haveSameChar = 0;
    for(i =0; i < len ; i++  )
    {
        if(left <= right)
        {   
            //检测是否出现重复
             //循环遍历整个数组 left -> right
            haveSameChar = 0;
            for(j = left; j < right ; j++)
            {
                if(s[j] == s[right])
                {
                    haveSameChar = 1;
                    break;
                }
            }
            if(haveSameChar)
            {
                //指向下一个
                left = j +1;
            }
        }
        //统计最大的间距
        max = max < (right - left + 1) ? (right - left + 1): max;
        right++;
    }
    return max;
}
```

## 5.最长回文子串

[5. 最长回文子串 - 力扣（LeetCode）](https://leetcode.cn/problems/longest-palindromic-substring/description/)

### 问题描述

> 给你一个字符串 `s`，找到 `s` 中最长的回文子串。如果字符串的反序与原始字符串相同，则该字符串称为回文字符串。

### 示例

- **示例 1**

  输入：s = "babad" 输出："bab" 解释："aba" 同样是符合题意的答案

- **示例 2**

  输入：s = "cbbd" 输出："bb"

### 解题思路

暴力检查所有可能的子串。利用`length` 定义为子串窗口的长度，然后从第一个位置开始寻找子串，并判断是否子串是否回文，如果是回文的，将它的长度和`max_length` 比较，如果比`max_length` 大，那么更新最长回文子串结果。

结果超出了时间限制。

```c
char* longestPalindrome(char* s) {
    int i,length,max_length=0;
    int len = strlen(s);
    static char result[1001];
    for(i=0;i<len;i++){
        for(length=1;length<=len-i;length++){
            char temp[1001];
            strncpy(temp,s+i ,length);
            temp[length] = '\\0';
            if(Reverse(temp)){
                if(max_length<length){
                    max_length = length;
                    strncpy(result, temp, length + 1);
                }
            }
        }

    }
    result[max_length] = '\\0';
    return result;
}
int Reverse(char *s){
    int i=0,j=strlen(s)-1;
    while(i<j){
        if(s[i] != s[j])
            return 0;
        i++;
        j--;
    }
    return 1;
}
```

### 题解

[**动态规划**](https://leetcode.cn/problems/longest-palindromic-substring/solutions/1389771/by-frosty-kapitsatia-7z99/)

1. 初始化：

   - `n` 被赋值为字符串 `s` 的长度。
   - 如果字符串长度小于2，那么最长回文子串就是字符串本身，因为长度为1的字符串自然就是回文串。所以直接返回 `s`。
   - `dp` 是一个二维数组，用来存储从字符串的任意两个位置开始的最长回文子串的信息。
   - `maxLen` 用于记录最长回文子串的长度，`begin` 用于记录最长回文子串的起始位置。
   
2. 动态规划数组初始化：

   - `dp[i][i]` 被初始化为 `true`，因为任何单个字符都是回文的。

3. 动态规划过程：

   - 代码使用两层循环来枚举所有可能的子串开始和结束位置。

   - 外层循环 `L` 枚举子串的长度。

   - 内层循环 `i` 枚举子串的起始位置，然后通过 `j = L + i - 1` 计算子串的结束位置。

   - 如果 `s[i]` 和 `s[j]` 不相等，则 `dp[i][j]` 被设置为 `false`，因为子串不是回文的。

   - 如果 `s[i]`和`s[j]`相等，那么需要进一步判断：

     - 如果子串长度小于3（即只有两个字符或三个字符），则它一定是回文串，因为两个相同的字符或三个连续相同的字符肯定是回文串。
     - 如果子串长度大于等于3，那么它是否是回文串取决于它的内部子串 `dp[i+1][j-1]` 是否是回文串。
   
4. 记录最长回文子串：

   - 如果找到一个回文子串，并且它的长度大于当前记录的最长长度 `maxLen`，则更新 `maxLen` 和 `begin`。

5. 返回结果：

   - 最后，函数通过设置 `s[begin+maxLen]='\\0'` 来截断字符串，使其只包含最长回文子串。
   - 然后，通过 `s=&s[begin]` 将 `s` 指向最长回文子串的起始位置。
   - 最后返回更新后的 `s`。

这段代码的时间复杂度是 $O(n^2)$，空间复杂度也是 $O(n^2)$，其中 n 是字符串的长度。这是因为代码使用了二维数组 `dp` 来保存所有可能的子串的回文信息，并且对每个子串进行了判断。

```c
char * longestPalindrome(char * s){
    int n = strlen(s);
    if (n < 2) {
        return s;
    }

    int maxLen = 1;
    int begin = 0;
    // dp[i][j] 表示 s[i..j] 是否是回文串
    int dp[n][n];
    // 初始化：所有长度为 1 的子串都是回文串
    for (int i = 0; i < n; i++) {
        dp[i][i] = true;
    }

    // 递推开始
    // 先枚举子串长度
    for (int L = 2; L <= n; L++) {
        // 枚举左边界，左边界的上限设置可以宽松一些
        for (int i = 0; i < n; i++) {
            // 由 L 和 i 可以确定右边界，即 j - i + 1 = L 得
            int j = L + i - 1;
            // 如果右边界越界，就可以退出当前循环
            if (j >= n) {
                break;
            }

            if (s[i]!= s[j]) {
                dp[i][j] = false;
            } else {
                if (j - i < 3) {
                    dp[i][j] = true;
                } else {
                    dp[i][j] = dp[i + 1][j - 1];
                }
            }

            // 只要 dp[i][L] == true 成立，就表示子串 s[i..L] 是回文，此时记录回文长度和起始位置
            if (dp[i][j] && j - i + 1 > maxLen) {
                maxLen = j - i + 1;
                begin = i;
            }
        }
    }
    s[begin+maxLen]='\\0';
    s=&s[begin];
    return s;
}
```

## 6.Z字形变换(不太会)

### 问题描述

> 将一个给定字符串 `s` 根据给定的行数 `numRows` ，以从上往下、从左到右进行 Z 字形排列。比如输入字符串为 `"PAYPALISHIRING"` 行数为 `3` 时，排列如下： P   A   H   N A P L S I I G Y   I   R
>
> 之后，你的输出需要从左往右逐行读取，产生出一个新的字符串，比如：`"PAHNAPLSIIGYIR"`。
>
> 请你实现这个将字符串进行指定行数变换的函数： `string convert(string s, int numRows);`

### 示例

- **示例1**

  **输入**：s = "PAYPALISHIRING", numRows = 3 **输出**："PAHNAPLSIIGYIR"

- **示例2**

  **输入**：s = "PAYPALISHIRING", numRows = 4 **输出**："PINALSIGYAHRPI"

  **解释**：

  P     I      N A   L S    I G Y A   H  R P     I

- **示例3**

  输入：s = “A”,numRows = 1

  输出：“A”

### 解题思路（题解）

1. 首先，判断特殊情况，如果给定的行数 numRows 为 1，则直接返回原字符串，因为无需进行 Z 字形变换。
2. 创建一个二维字符数组 **`rows`**，其大小为 numRows 行，字符串长度加一列。这个数组用于存储按 Z 字形排列后的字符。
3. 遍历输入字符串 **`s`**，按照 Z 字形的规则将字符放入对应的行中。具体来说，可以利用一个变量 **`goingDown`** 来标识当前字符是向下还是向上移动，并通过 **`currentRow`** 来记录当前行数。
4. 将每行的字符按顺序连接起来，得到结果字符串。为了实现这一步，我们遍历二维字符数组 **`rows`**，按照顺序将非空字符加入结果字符串中。
5. 最后返回结果字符串。

```c
char *convert(char *s, int numRows) {
    if (numRows == 1) return s; // 如果只有一行，直接返回原字符串
    
    int len = strlen(s);
    char *result = (char *)malloc((len + 1) * sizeof(char)); // 为结果字符串分配内存
    result[len] = '\\0'; // 结尾添加空字符
    
    char rows[numRows][len + 1]; // 创建存储行的数组，行数为numRows，列数为字符串长度加一
    memset(rows, 0, sizeof(rows)); // 将数组初始化为空字符
    
    int currentRow = 0; // 当前行数
    int goingDown = 0; // 是否向下移动

    for (int i = 0; i < len; i++) {
        rows[currentRow][i] = s[i]; // 将字符加入当前行

        if (currentRow == 0 || currentRow == numRows - 1) // 如果当前在首行或尾行，改变移动方向
            goingDown = !goingDown;

        currentRow += goingDown ? 1 : -1; // 根据移动方向更新当前行数
    }

    int pos = 0;
    for (int i = 0; i < numRows; i++) {
        for (int j = 0; j < len; j++) {
            if (rows[i][j] != '\\0') {
                result[pos++] = rows[i][j]; // 按顺序连接每行的字符
            }
        }
    }
    
    return result;
}
```

## 455. 分发饼干

[455. 分发饼干 - 力扣（LeetCode）](https://leetcode.cn/problems/assign-cookies/description/)

### 问题描述

假设你是一位很棒的家长，想要给你的孩子们一些小饼干。但是，每个孩子最多只能给一块饼干。

对每个孩子 `i`，都有一个胃口值 `g[i]`，这是能让孩子们满足胃口的饼干的最小尺寸；并且每块饼干 `j`，都有一个尺寸 `s[j]` 。如果 `s[j] >= g[i]`，我们可以将这个饼干 `j` 分配给孩子 `i` ，这个孩子会得到满足。你的目标是满足尽可能多的孩子，并输出这个最大数值。

### 示例

- **示例1**

  **输入**：g = [1,2,3], s = [1,1]

  **输出**：1

  **解释**：你有三个孩子和两块小饼干，3个孩子的胃口值分别是：1,2,3。虽然你有两块小饼干，由于他们的尺寸都是 1，你只能让胃口值是 1 的孩子满足。所以你应该输出 1。

- **示例2**

  **输入**：g = [1,2], s = [1,2,3]

  **输出**：2

  **解释**：你有两个个孩子和三块小饼干，2个孩子的胃口值分别是：1,2。你拥有的饼干数量和尺寸都足以让所有孩子满足。所以你应该输出 1。

### 解体思路

**错误思路1：**将饼干中的最小值分给胃口最小的孩子，分完之后，将胃口最小的孩子和最小的饼干移除列表，重复这个过程，依次进行判断。

错误原因：由于可能一开始的最小饼干就不能满足胃口最小的孩子，但是并没有剔除胃口最小的孩子，所以需要将最小的饼干剔除，然后接着判断。能够完成测试用例，但是提交超时，需要改进，超时代码如下：

```python
import numpy as np
class Solution(object):
    def findContentChildren(self, g, s):
        """
        :type g: List[int]
        :type s: List[int]
        :rtype: int
        """
        count = 0
        for i in range(len(s)):
            for j in range(len(g)):
                if(len(s)!=0):
                    min_s = np.min(s)
                    min_g = np.min(g)
                    if(min_s >= min_g):
                        g.remove(min_g)
                        s.remove(min_s)
                        count+=1
                        break;
                    else:
                        s.remove(min_s)
        return count
```

### 题解

考虑贪心的策略，因为最小的孩子最容易吃饱，所以最先考虑最容易吃饱的孩子，为了使得剩下的饼干可以满足饥饿度更大的孩子，所以我们应该把大于等于这个孩子饥饿度的，且大小最小的饼干给这个孩子，满足这个孩子后采取相同的策略，考虑剩下的孩子饥饿度最小的孩子，直到没有满足条件的饼干存在。（这么看好像和我的思路差不多）

总的来说就是给剩余孩子里饥饿度最小的孩子分配最小的能饱腹的饼干！

实现问题！！！由于要获取大小关系，一个便捷的方法是把孩子和饼干分别排序。这样就能够从饥饿度最小的孩子和饱腹度最小的饼干出发，计算有多少个对子可以满足。

```python
class Solution(object):
    def findContentChildren(self, g, s):
        """
        :type g: List[int]
        :type s: List[int]
        :rtype: int
        """
        g.sort()
        s.sort()
        child_i,cookie_i = 0, 0
        n_children, n_cookies = len(g), len(s)
        while(child_i < n_children and cookie_i < n_cookies):
            if g[child_i] <= s[cookie_i]:
                child_i += 1
            cookie_i += 1
        return child_i
```

`注`：列表有自己的sort方法，其对列表进行原址排序，既然是原址排序，那显然元组不可能拥有这种方法，因为元组是不可修改的。

## 135.分发糖果

### 问题描述

`n` 个孩子站成一排。给你一个整数数组 `ratings` 表示每个孩子的评分。

你需要按照以下要求，给这些孩子分发糖果：

- 每个孩子至少分配到 `1` 个糖果。
- 相邻两个孩子评分更高的孩子会获得更多的糖果。

请你给每个孩子分发糖果，计算并返回需要准备的 **最少糖果数目** 。

### 示例

- 示例1

  **输入**：ratings = [1,0,2]

  **输出**：5

  **解释**：你可以分别给第一个、第二个、第三个孩子分发 2、1、2 颗糖果。

- 示例2

  **输入**：ratings = [1,2,2]

  **输出**：4

  **解释**：你可以分别给第一个、第二个、第三个孩子分发 1、2、1 颗糖果。 第三个孩子只得到 1 颗糖果，这满足题面中的两个条件。

### 解题思路

先把孩子按照评分排序， 然后每个孩子先给一颗，这样就能保证每个孩子均有一颗糖果，接下来要满足相邻两个孩子评分更高的孩子会获得更多的糖果，那就依次判断，第一个和第二个判断，第二个和第三个判断，依此类推直到判断结束。

这样做，官方给的两个例子都没问题！感觉好顺利！提交发现错了！！！

错误的输入：ratings　＝　[1,2,87,87,87,2,1]

预期输出13，实际输出9。

仔细一想，果然还是太年轻了！困难级别的题是这么好做的？顺序还是不能打乱，所以全部推到重新来！

思考不出来！！！！！

### 题解

存在比较关系的贪心策略并不一定需要排序。虽然这一道题也是运用贪心策略，但我们只需要简单的两次遍历即可:把所有孩子的糖果数初始化为1;先从左往右遍历一遍，如果右边孩子的评分比左边的高，则右边孩子的糖果数更新为左边孩子的糖果数加1;再从右往左遍历一遍,如果左边孩子的评分比右边的高，且左边孩子当前的糖果数不大于右边孩子的糖果数，则左边孩子的糖果数更新为右边孩子的糖果数加1。通过这两次遍历，分配的糖果就可以满足题目要求了。这里的贪心策略即为，在每次遍历中，只考虑并更新相邻一侧的大小关系。

```python
class Solution(object):
    def candy(self, ratings):
        """
        :type ratings: List[int]
        :rtype: int
        """
        result_list = [1]*len(ratings) #一个孩子一个糖果
        for i in range(1,len(ratings)):
            if ratings[i-1]<ratings[i]:
                result_list[i] = result_list[i-1]+1
        for i in range(len(ratings)-1,0,-1):
            if ratings[i]<ratings[i-1]:
                result_list[i-1] = max(result_list[i-1],result_list[i]+1)
        
        return sum(result_list)
```

## 435.无重叠区间

### 问题描述

给定一个区间的集合 `intervals` ，其中 `intervals[i] = [starti, endi]` 。返回需要移除区间的最小数量，使剩余区间互不重叠。

**注意：\**只在一点上接触的区间是\**不重叠的**。例如 `[1, 2]` 和 `[2, 3]` 是不重叠的。

### 示例

- **示例1**

  **输入**: intervals = [[1,2],[2,3],[3,4],[1,3]] **输出**: 1 **解释**: 移除 [1,3] 后，剩下的区间没有重叠。

- **示例2**

  **输入**: intervals = [ [1,2], [1,2], [1,2] ] **输出**: 2 **解释**: 你需要移除两个 [1,2] 来使剩下的区间没有重叠。

- **示例3**

  **输入**: intervals = [ [1,2], [2,3] ] **输出**: 0 **解释**: 你不需要移除任何区间，因为它们已经是无重叠的了。

### 解题思路(GPT)

这个问题可以用贪心算法来解决。为了使得移除的区间数量最小，我们应该优先保留那些结束时间较早的区间，因为这样可以为后续的区间留出更多空间。

具体步骤如下：

1. 将所有区间按结束时间升序排序。
2. 从第一个区间开始，依次检查后面的区间是否与当前区间重叠。如果不重叠，就把当前区间更新为新的参考区间；如果重叠，则移除该区间，记录移除的数量。
3. 最后返回需要移除的区间数量。

### 题解(关于题解的合理性)

来源：[435. 无重叠区间 - 力扣（LeetCode）](https://leetcode.cn/problems/non-overlapping-intervals/solutions/1263171/tan-xin-jie-fa-qi-shi-jiu-shi-yi-ceng-ch-i63h/)

**为什么按照区间右端点排序？**

官解里对这个描述的非常清楚了，这个题其实是`预定会议`的一个问题，给你若干时间的会议，然后去预定会议，那么能够预定的最大的会议数量是多少？核心在于我们要找到最大不重叠区间的个数。 如果我们把本题的区间看成是会议，那么按照右端点排序，我们一定能够找到一个最先结束的会议，而这个会议一定是我们需要添加到最终结果的的首个会议。（这个不难贪心得到，因为这样能够给后面预留的时间更长）。

**关于为什么是不能按照区间左端点排序？**

这里补充一下为什么不能按照区间左端点排序。同样地，我们把本题的`区间`看成是`会议`，如果“按照左端点排序，我们一定能够找到一个最先开始的会议”，但是最先开始的会议，不一定最先结束。。举个例子：

```
|_________|                  区间a
  |___|                      区间b       
       |__|                  区间c   
            |______|         区间d   
```

`区间a`是最先开始的，如果我们采用`区间a`作为放入最大不重叠区间的首个区间，那么后面我们只能采用`区间d`作为第二个放入最大不重叠区间的区间，但这样的话，最大不重叠区间的数量为2。但是如果我们采用区间b作为放入最大不重叠区间的首个区间，那么最大不重叠区间的数量为3，因为区间b是最先结束的。

```python
class Solution(object):
    def eraseOverlapIntervals(self, intervals):
        """
        :type intervals: List[List[int]]
        :rtype: int
        """
        intervals.sort(key=lambda x: x[1])
        removed, prev_end = 0, intervals[0][1]
        for i in range(1, len(intervals)):
            if prev_end > intervals[i][0]:
                removed += 1
            else:
                prev_end = intervals[i][1]
        return removed
```

## 605 种花问题

### 问题描述

假设有一个很长的花坛，一部分地块种植了花，另一部分却没有。可是，花不能种植在相邻的地块上，它们会争夺水源，两者都会死去。

给你一个整数数组 `flowerbed` 表示花坛，由若干 `0` 和 `1` 组成，其中 `0` 表示没种植花，`1` 表示种植了花。另有一个数 `n` ****，能否在不打破种植规则的情况下种入 `n` ****朵花？能则返回 `true` ，不能则返回 `false` 。

### 示例

- **示例1**

  **输入**：flowerbed=[1,0,0,0,1],n=1

  **输出**：true

- **示例2**

  **输入**：flowerbed = [1,0,0,0,1], n = 2

  **输出**：false

### 解题思路

错误思路：如果相邻的三朵元素均是0，那么中间那个一元素一定可以种花，所有只要满足相邻三个元素为0的，就讲n减1，表示已经种下一朵花了！

提交发现错了，GPT说原因如下

1. **相邻三个 0 的情况**：

   你假设只要遇到 `[0, 0, 0]`，就可以在中间种一朵花。但这个思路在**数组的边界**情况会出错。比如：

   - 如果是 `[0, 0]` 出现在花坛的**末尾**或**开头**，照你这种策略它会被忽略掉，而实际上在这种情况下也是可以种一朵花的（比如 `[0, 0, 1]` 开头可以种花）。
   - 你需要仔细检查花坛的首尾和每一段之间的 0 是否有花种。

2. **错误逻辑**：

   你没有在每次发现 0 的时候即时判断能否种花。单纯依赖“三个连续 0 才减 n”会漏掉一些情况，比如 `[1, 0, 0, 0]` 在末尾也可以种花。

```python
class Solution(object):
    def canPlaceFlowers(self, flowerbed, n):
        """
        :type flowerbed: List[int]
        :type n: int
        :rtype: bool
        """
        length = len(flowerbed)
        for i in range(length):
            # 判断当前位置是否为0，且左右两侧允许种花
            if flowerbed[i] == 0 and \\
            (i == 0 or flowerbed[i - 1] == 0) and \\
            (i == length - 1 or flowerbed[i + 1] == 0):
                # 种花，并将该位置标记为1
                flowerbed[i] = 1
                n -= 1  # 种了一朵花，减少需要种的数量
            # 如果所有花都已经种完了，直接返回True
            if n <= 0:
                return True
        # 如果遍历完数组后仍有未种完的花，返回False
        return n <= 0
```

### 题解

[605. 种花问题 - 力扣（LeetCode）](https://leetcode.cn/problems/can-place-flowers/solutions/2463018/ben-ti-zui-jian-dan-xie-fa-pythonjavacgo-6a6k/)

从左到右遍历数组，能种花就立刻种花。

如何判断能否种花？由于「花不能种植在相邻的地块上」，如果要在下标`i`处种花，需要满足 `flowerbed[i−1],flowerbed[i],flowerbed[i+1]` 均为0。

每种一朵花，就把 n 减一。如果最后 n≤0，则返回 true，否则返回 false。

**为了简化判断逻辑，可以在数组的开头和末尾各插入一个 0。**

```python
class Solution:
    def canPlaceFlowers(self, flowerbed: List[int], n: int) -> bool:
        flowerbed = [0] + flowerbed + [0]
        for i in range(1, len(flowerbed) - 1):
            if flowerbed[i - 1] == 0 and flowerbed[i] == 0 and flowerbed[i + 1] == 0:
                flowerbed[i] = 1  # 种花！
                n -= 1
        return n <= 0
```

## 452 用最少数量的箭引爆气球

### 问题描述

有一些球形气球贴在一堵用 XY 平面表示的墙面上。墙面上的气球记录在整数数组 `points` ，其中`points[i] = [xstart, xend]` 表示水平直径在 `xstart` 和 `xend`之间的气球。你不知道气球的确切 y 坐标。

一支弓箭可以沿着 x 轴从不同点 **完全垂直** 地射出。在坐标 `x` 处射出一支箭，若有一个气球的直径的开始和结束坐标为 `xstart`，`xend`， 且满足  `xstart ≤ x ≤ xend`，则该气球会被 **引爆** 。可以射出的弓箭的数量 **没有限制** 。 弓箭一旦被射出之后，可以无限地前进。

给你一个数组 `points` ，返回引爆所有气球所必须射出的 **最小** 弓箭数 。

### 示例

- **示例一**

  **输入**：points = [[10,16],[2,8],[1,6],[7,12]] **输出**：2 **解释**：气球可以用2支箭来爆破: -在x = 6处射出箭，击破气球[2,8]和[1,6]。 -在x = 11处发射箭，击破气球[10,16]和[7,12]。

- **示例二**

  **输入**：points = [[1,2],[3,4],[5,6],[7,8]] **输出**：4 **解释**：每个气球需要射出一支箭，总共需要4支箭。

- 示例三

  **输入**：points = [[1,2],[2,3],[3,4],[4,5]] **输出**：2 **解释**：气球可以用2支箭来爆破:

  - 在x = 2处发射箭，击破气球[1,2]和[2,3]。
  - 在x = 4处射出箭，击破气球[3,4]和[4,5]。

### 解题思路

题目的要求其实就是求公共区间数量，然后用区间数量减去公共区间的数量也就是所得到的答案，那么问题来了，怎么求得公共区间的数量？将区间按照区间的开头`xstart`进行排序，然后依次判断每一个区间的`xend` 与后续区间的`xstart` 的大小关系。如果大于等于后续元素`xstart` ，那么公共区间的个数就加一。最后返回区间个数减去公共区间个数即可。

但这样就会有个问题，只要两个区间的`xend`大于等于下一个区间的`xstart`，就计为一个公共区间。这种方法可能会多计或者漏计重叠区间，导致最终的结果不准确。例如：输入`[[1, 2], [2, 3], [3, 4], [4, 5]]`，每个区间和下一个都有部分重叠，但最优策略是只需要**2支箭**。你的方法会逐一计数，可能导致最终答案错误。 实际上，本题求解的是如何**用最少的箭数引爆所有气球**，也就是找到最少的重叠区域射出箭。正确的思路不需要遍历所有公共区间数量，而是需要找到**尽量多的重叠部分**，并以该部分的右边界作为箭射出的最佳位置。

**优化后的正确思路：**

1. 按气球右边界（xend）升序排序

   - 这样保证我们尽早处理当前气球，并且为下一次射箭提供最小的可行范围。
   
2. 贪心策略

   - 每次在当前气球的右边界射出一支箭，这样能保证尽量多地覆盖后续的气球，避免多余的箭。

```python
class Solution(object):
    def findMinArrowShots(slef,points):
        if not points:
            return 0

        # 按照气球的右边界xend升序排序
        points.sort(key=lambda x: x[1])

        arrows = 1  # 至少需要一支箭
        current_end = points[0][1]  # 记录当前箭的射出位置

        for start, end in points[1:]:
            # 如果当前气球的起点大于当前箭的射出位置，则需要新的箭
            if start > current_end:
                arrows += 1
                current_end = end  # 更新射出位置为当前气球的右边界

        return arrows
```

## 3200.三角形的最大高度

### 问题描述

给你两个整数 `red` 和 `blue`，分别表示红色球和蓝色球的数量。你需要使用这些球来组成一个三角形，满足第 1 行有 1 个球，第 2 行有 2 个球，第 3 行有 3 个球，依此类推。

每一行的球必须是 **相同** 颜色，且相邻行的颜色必须 **不同**。

返回可以实现的三角形的 **最大** 高度。

### 示例

- **示例1**

  **输入：** red = 2, blue = 4

  **输出：** 3

  **解释：**

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/dc70a931-c70a-4b0e-b36e-d260f06c4136/image.png)

- **示例2**

  **输入：** red = 2, blue = 1

  **输出：** 2

  **解释**：

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/c44a6488-ac4f-4410-9001-22892f67d47f/image.png)

  上图显示了唯一可能的排列方式。

- **示例3**

  **输入：** red = 1, blue = 1

  **输出：** 1

- **示例4**

  **输入：** red = 10, blue = 1

  **输出：** 2

  **解释：**

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/52a26af3-0db1-4515-b6a4-1105657d0220/image.png)

  上图显示了唯一可能的排列方式。

### 题解

递增地枚举三角形的高度，在第 i 行时，如果对应的颜色的剩余球数大于等于 i 个，那么就可以组成第 i 行，否则不能，三角形的最大高度为 i−1。

三角形的颜色布局有两种可能：即红蓝交替（第一行为红色）或者蓝红交替（第一行为蓝色），我们分别枚举这两种情况，并取二者高度的较大值即可。

作者：力扣官方题解 链接：https://leetcode.cn/problems/maximum-height-of-a-triangle/solutions/2946682/san-jiao-xing-de-zui-da-gao-du-by-leetco-vch2/

```python
class Solution(object):
    def maxHeightOfTriangle(self, red, blue):
        """
        :type red: int
        :type blue: int
        :rtype: int
        """
        def maxHeight(x,y):
            i = 1
            while True:
                if i % 2 == 1:
                    x -= i
                    if x < 0:
                        return i - 1
                else:
                    y -= i
                    if y < 0:
                        return i - 1
                i += 1
        
        return max(maxHeight(red, blue), maxHeight(blue, red))
        
```

## **3194.最小元素和最大元素的最小平均值**

### 问题描述

你有一个初始为空的浮点数数组 `averages`。另给你一个包含 `n` 个整数的数组 `nums`，其中 `n` 为偶数。

你需要重复以下步骤 `n / 2` 次：

- 从 `nums` 中移除 **最小** 的元素 `minElement` 和 **最大** 的元素 `maxElement`。
- 将 `(minElement + maxElement) / 2` 加入到 `averages` 中。

返回 `averages` 中的 **最小** 元素。

### 示例

- **示例1**

  **输入：** nums = [7,8,3,4,15,13,4,1]

  **输出：** 5.5

  **解释：**

  | **步骤** | **nums**            | **averages** |
  | -------- | ------------------- | ------------ |
  | 0        | [7,8,3,4,15,13,4,1] | []           |
  | 1        | [7,8,3,4,13,4]      | [8]          |
  | 2        | [7,8,4,4]           | [8,8]        |
  | 3        | [7,4]               | [8,8,6]      |
  | 4        | []                  | [8,8,6,5.5]  |

- **示例2**

  **输入：** nums = [1,9,8,3,10,5]

  **输出：** 5.5

  **解释：**

  | **步骤** | **nums**       | **averages** |
  | -------- | -------------- | ------------ |
  | 0        | [1,9,8,3,10,5] | []           |
  | 1        | [9,8,3,5]      | [5.5]        |
  | 2        | [8,5]          | [5.5,6]      |
  | 3        | []             | [5.5,6,6.5]  |

- **示例3**

  **输入：** nums = [1,2,3,7,8,9]

  **输出：** 5.0

  **解释：**

  | **步骤** | **nums**      | **averages** |
  | -------- | ------------- | ------------ |
  | 0        | [1,2,3,7,8,9] | []           |
  | 1        | [2,3,7,8]     | [5]          |
  | 2        | [3,7]         | [5,5]        |
  | 3        | []            | [5,5,5]      |

### 解题思路

按照题目给的思路，一步一步写下来。

```python
class Solution(object):
    def minimumAverage(self, nums):
        """
        :type nums: List[int]
        :rtype: float
        """
        averages = []
        while(len(nums)!=0):
            min_num = min(nums)
            max_num = max(nums)
            nums.remove(min_num)
            nums.remove(max_num)
            averages.append(float((min_num+max_num)/2.0))
        return min(averages)
```

### 题解

为方便计算，把 nums 从小到大排序。

排序后，对于 i=0,1,2,⋯,n/2，计算 nums[i]+nums[n−1−i] 的最小值。最后，返回最小值除以 2 的结果。

最后除以 2，这样可以避免在循环中做浮点运算，只在最后返回时做一次浮点运算。

注意题目保证 n 是偶数。

作者：灵茶山艾府 链接：https://leetcode.cn/problems/minimum-average-of-smallest-and-largest-elements/solutions/2819374/pai-xu-bian-li-pythonjavacgo-by-endlessc-x155/

```python
class Solution:
    def minimumAverage(self, nums: List[int]) -> float:
        nums.sort()
        return min(nums[i] + nums[-1 - i] for i in range(len(nums) // 2)) / 2
```

## **908. 最小差值I**

### 问题描述

给你一个整数数组 `nums`，和一个整数 `k` 。

在一个操作中，您可以选择 `0 <= i < nums.length` 的任何索引 `i` 。将 `nums[i]` 改为 `nums[i] + x` ，其中 `x` 是一个范围为 `[-k, k]` 的任意整数。对于每个索引 `i` ，最多 **只能** 应用 **一次** 此操作。

`nums` 的 **分数** 是 `nums` 中最大和最小元素的差值。

在对 `nums` 中的每个索引最多应用一次上述操作后，返回 `nums` 的最低 **分数** 。

### 示例

- **示例1**

  **输入**：nums = [1], k = 0 **输出**：0 **解释**：分数是 max(nums) - min(nums) = 1 - 1 = 0。

- **示例2**

  **输入**：nums = [0,10], k = 2 **输出**：6 **解释**：将 nums 改为 [2,8]。分数是 max(nums) - min(nums) = 8 - 2 = 6。

- **示例3**

  **输入**：nums = [1,3,6], k = 3 **输出**：0 **解释**：将 nums 改为 [4,4,4]。分数是 max(nums) - min(nums) = 4 - 4 = 0。

### 解题思路

nums的分数是nums中最大和最小元素之间的差值，所以说，要么二者相等，要么二者存在大小关系，前面一种情况的分数为0，后面一种情况的分数一定是一个正值。并且如果处在后一种情况的话，要求最小差值，那么就是最大值减去k，最小值加上k，也就是 `max-k-(min+k)`。

```python
class Solution(object):
    def smallestRangeI(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        return max(0, max(nums) - min(nums) - 2 * k)
```

### 题解

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/adde943a-e3f6-4b01-aedb-8e732e015d0c/image.png)

作者：灵茶山艾府

链接：[908. 最小差值 I - 力扣（LeetCode）](https://leetcode.cn/problems/smallest-range-i/solutions/2928878/tu-fen-lei-tao-lun-jian-ji-xie-fa-python-9phy/)

## **910. 最小差值 II**

### 问题描述

给你一个整数数组 `nums`，和一个整数 `k` 。

对于每个下标 `i`（`0 <= i < nums.length`），将 `nums[i]` 变成 ****`nums[i] + k` 或 `nums[i] - k` 。

`nums` 的 **分数** 是 `nums` 中最大元素和最小元素的差值。

在更改每个下标对应的值之后，返回 `nums` 的最小 **分数** 。

### 示例

- **示例1**

  输入：nums = [1], k = 0 输出：0 解释：分数 = max(nums) - min(nums) = 1 - 1 = 0 。

- **示例2**

  输入：nums = [0,10], k = 2 输出：6 解释：将数组变为 [2, 8] 。分数 = max(nums) - min(nums) = 8 - 2 = 6 。

- **示例3**

  输入：nums = [1,3,6], k = 3 输出：3 解释：将数组变为 [4, 6, 3] 。分数 = max(nums) - min(nums) = 6 - 3 = 3 。

### 题解

如 最小差值 I 问题的解决方法，较小的 `nums[i]` 将增加，较大的 `nums[i]` 将变小。

我们可以对上述想法形式化表述：如果 `nums[i]` 小于 `nums[j]`，我们不必考虑当 `nums[i]` 增大时 `nums[j]` 会减小。这是因为区间 `(nums[i]+k,nums[j]−k)` 是 `(nums[i]−k,nums[j]+k)` 的子集。其中当 a>b 时, (a,b) 表示 (b,a)。

这意味着对于 (up,down) 的选择一定不会差于 (down,up)。我们可以证明其中一个区间是另一个的子集，通过证明 `nums[i]+k` 和 `nums[j]−k` 是在 `nums[i]−k` 和 `nums[j]+k` 之间。

对于有序的数组，设 nums[i] 是最大的需要增长的 i，那么 `nums[0]+k`,`nums[i]+k`, `nums[i+1]−k`,`nums[n−1]−k` 就是计算结果的唯一值。

作者：力扣官方题解 链接：https://leetcode.cn/problems/smallest-range-ii/solutions/2928188/zui-xiao-chai-zhi-ii-by-leetcode-solutio-epd3/

```python
class Solution:
    def smallestRangeII(self, nums, k):
        nums.sort()
        mi, ma = nums[0], nums[-1]
        res = ma - mi
        for i in range(len(nums) - 1):
            a, b = nums[i], nums[i+1]
            res = min(res, max(ma - k, a + k) - min(mi + k, b - k))
        return res
```

## **3184. 构成整天的下标对数目 I**

### 问题描述

给你一个整数数组 `hours`，表示以 **小时** 为单位的时间，返回一个整数，表示满足 `i < j` 且 `hours[i] + hours[j]` 构成 **整天** 的下标对 `i`, `j` 的数目。

**整天** 定义为时间持续时间是 24 小时的 **整数倍** 。

例如，1 天是 24 小时，2 天是 48 小时，3 天是 72 小时，以此类推。

### 示例

- 示例一

  输入： hours = [12,12,30,24,24]

  输出： 2

  解释：

  构成整天的下标对分别是 (0, 1) 和 (3, 4)。

- 示例二

  输入： hours = [72,48,24,3]

  输出： 3

  解释：

  构成整天的下标对分别是 (0, 1)、(0, 2) 和 (1, 2)。

### 解题思路

两层循环，依次遍历，然后判断能否被24整除！

```python
class Solution(object):
    def countCompleteDayPairs(self, hours):
        """
        :type hours: List[int]
        :rtype: int
        """
        count = 0
        for i in range(len(hours)-1):
            for j in range(i+1,len(hours)):
                if (hours[i]+hours[j])%24 == 0:
                    count+=1
        return count
```

### 题解

类似于两数之和，三数之和的取模版本。将每个元素取模后的值存储进哈希表中，并且判断在哈希表中该余数对应的补数出现的个数，即为每次判断后应该增加的下标对数目。整个代码只需要一次循环即可完成

```python
class Solution(object):
    def countCompleteDayPairs(self, hours):

        """
        :type hours: List[int]
        :rtype: int
        """

        remainder_count = {}
        total_pairs = 0
        for h in hours:
            remainder = h % 24
            complement = (24-remainder) % 24
         
            if complement in remainder_count:
                total_pairs += remainder_count[complement]
            if remainder in remainder_count:
                remainder_count[remainder] += 1
            else:
                remainder_count[remainder] = 1
                
        return total_pairs
```

# **3185. 构成整天的下标对数目 II**

### 问题描述

给你一个整数数组 `hours`，表示以 **小时** 为单位的时间，返回一个整数，表示满足 `i < j` 且 `hours[i] + hours[j]` 构成 **整天** 的下标对 `i`, `j` 的数目。

**整天** 定义为时间持续时间是 24 小时的 **整数倍** 。

例如，1 天是 24 小时，2 天是 48 小时，3 天是 72 小时，以此类推。

### 示例

- 示例一

  输入： hours = [12,12,30,24,24]

  输出： 2

  解释：

  构成整天的下标对分别是 (0, 1) 和 (3, 4)。

- 示例二

  输入： hours = [72,48,24,3]

  输出： 3

  解释：

  构成整天的下标对分别是 (0, 1)、(0, 2) 和 (1, 2)。

### 提示

- `1 <= hours.length <= $5 * 10^5$`
- `1 <= hours[i] <= $10^9$`

### 解题思路

乍一看和原来的题没什么区别，但是仔细观察就会发现，给出的数据量不一样！所以原来的双重for循环的解法会导致超出时间限制！

所以还得优化算法！！

### 题解

借鉴 1. 两数之和 的思路，遍历 hours 的同时，用一个哈希表（或者数组）记录元素的出现次数。

举几个例子：

如果 hours[i]=1，那么需要知道左边有多少个模 24 是 23 的数，这些数加上 1 都是 24 的倍数。 如果 hours[i]=2，那么需要知道左边有多少个模 24 是 22 的数，这些数加上 2 都是 24 的倍数。 如果 hours[i]=26，那么需要知道左边有多少个模 24 是 22 的数，这些数加上 26 都是 24 的倍数。 一般地，对于 hours[i]，需要知道左边有多少个模 24 是 24−hours[i]mod24 的数。

特别地，如果 hours[i] 模 24 是 0，那么需要知道左边有多少个模 24 也是 0 的数。

这两种情况可以合并为：累加左边(24−hours[i]mod24)mod24的出现次数。

代码实现时，用一个长为 24 的数组 cnt 维护 hours[i]mod24 的出现次数。

作者：灵茶山艾府 链接：https://leetcode.cn/problems/count-pairs-that-form-a-complete-day-ii/solutions/2812385/tao-lu-mei-ju-you-wei-hu-zuo-pythonjavac-3vhv/

```python
class Solution(object):
    def countCompleteDayPairs(self, hours):

        """
        :type hours: List[int]
        :rtype: int
        """
        H = 24
        ans = 0
        cnt = [0] * H
        for t in hours:
            # 先查询 cnt，再更新 cnt，因为题目要求 i < j
            # 如果先更新，再查询，就把 i = j 的情况也考虑进去了
            ans += cnt[(H - t % H) % H]
            cnt[t % H] += 1
        return ans
```

## **3175. 找到连续赢 K 场比赛的第一位玩家**

### 问题描述

有 `n` 位玩家在进行比赛，玩家编号依次为 `0` 到 `n - 1` 。

给你一个长度为 `n` 的整数数组 `skills` 和一个 **正** 整数 `k` ，其中 `skills[i]` 是第 `i` 位玩家的技能等级。`skills` 中所有整数 **互不相同** 。

所有玩家从编号 `0` 到 `n - 1` 排成一列。

比赛进行方式如下：

- 队列中最前面两名玩家进行一场比赛，技能等级 **更高** 的玩家胜出。
- 比赛后，获胜者保持在队列的开头，而失败者排到队列的末尾。

这个比赛的赢家是 **第一位连续** 赢下 `k` 场比赛的玩家。

请你返回这个比赛的赢家编号。

### 示例

- 示例1

  **输入：**skills = [4,2,6,3,9], k = 2

  **输出：**2

  **解释：**

  一开始，队列里的玩家为 `[0,1,2,3,4]` 。比赛过程如下：

  - 玩家 0 和 1 进行一场比赛，玩家 0 的技能等级高于玩家 1 ，玩家 0 胜出，队列变为 `[0,2,3,4,1]` 。
  - 玩家 0 和 2 进行一场比赛，玩家 2 的技能等级高于玩家 0 ，玩家 2 胜出，队列变为 `[2,3,4,1,0]` 。
  - 玩家 2 和 3 进行一场比赛，玩家 2 的技能等级高于玩家 3 ，玩家 2 胜出，队列变为 `[2,4,1,0,3]` 。

  玩家 2 连续赢了 `k = 2` 场比赛，所以赢家是玩家 2 。

- 示例2

  **输入：**skills = [2,5,4], k = 3

  **输出：**1

  **解释：**

  一开始，队列里的玩家为 `[0,1,2]` 。比赛过程如下：

  - 玩家 0 和 1 进行一场比赛，玩家 1 的技能等级高于玩家 0 ，玩家 1 胜出，队列变为 `[1,2,0]` 。
  - 玩家 1 和 2 进行一场比赛，玩家 1 的技能等级高于玩家 2 ，玩家 1 胜出，队列变为 `[1,0,2]` 。
  - 玩家 1 和 0 进行一场比赛，玩家 1 的技能等级高于玩家 0 ，玩家 1 胜出，队列变为 `[1,2,0]` 。

  玩家 1 连续赢了 `k = 3` 场比赛，所以赢家是玩家 1 。

### 提示

- `n == skills.length`
- `2 <= n <= 105`
- `1 <= k <= 109`
- `1 <= skills[i] <= 106`
- `skills` 中的整数互不相同。

### 题解

我们注意到，每次会比较数组的前两个元素，不管结果怎么样，下一次的比较，一定是轮到了数组中的下一个元素和当前的胜者进行比较。因此，如果循环了 n−1 次，那么最后的胜者一定是数组中的最大元素。否则，如果某个元素连续胜出了 k 次，那么这个元素就是最后的胜者。

```python
class Solution:
    def findWinningPlayer(self, skills: List[int], k: int) -> int:
        n = len(skills)
        k = min(k, n - 1)
        i = cnt = 0
        for j in range(1, n):
            if skills[i] < skills[j]:
                i = j
                cnt = 1
            else:
                cnt += 1
            if cnt == k:
                break
        return i
```

作者：ylb 链接：https://leetcode.cn/problems/find-the-first-player-to-win-k-games-in-a-row/solutions/2962919/python3javacgotypescript-yi-ti-yi-jie-na-49i9/

## **684. 冗余连接**

### 问题描述

树可以看成是一个连通且 **无环** 的 **无向** 图。

给定往一棵 `n` 个节点 (节点值 `1～n`) 的树中添加一条边后的图。添加的边的两个顶点包含在 `1` 到 `n` 中间，且这条附加的边不属于树中已存在的边。图的信息记录于长度为 `n` 的二维数组 `edges` ，`edges[i] = [ai, bi]` 表示图中在 `ai` 和 `bi` 之间存在一条边。

请找出一条可以删去的边，删除后可使得剩余部分是一个有着 `n` 个节点的树。如果有多个答案，则返回数组 `edges` 中最后出现的那个。

### 示例

- 示例1

  输入: edges = [[1,2], [1,3], [2,3]] 输出: [2,3]

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/32c607e6-2bf3-42d4-ac45-e058b000565f/image.png)

- 示例2

  输入: edges = [[1,2], [2,3], [3,4], [1,4], [1,5]] 输出: [1,4]

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/6dacbb26-b6bd-42d9-b6cc-bf21d04db8da/image.png)

### 解题思路（题解）

题目意思就是删去一条边，然后变成一个无环图！可以使用并查集来解决。

在一棵树中，边的数量比节点的数量少 1。如果一棵树有 n 个节点，则这棵树有 n−1 条边。这道题中的图在树的基础上多了一条附加的边，因此边的数量也是 n。

树是一个连通且无环的无向图，在树中多了一条附加的边之后就会出现环，因此附加的边即为导致环出现的边。

可以通过并查集寻找附加的边。初始时，每个节点都属于不同的连通分量。遍历每一条边，判断这条边连接的两个顶点是否属于相同的连通分量。

- 如果两个顶点属于不同的连通分量，则说明在遍历到当前的边之前，这两个顶点之间不连通，因此当前的边不会导致环出现，合并这两个顶点的连通分量。
- 如果两个顶点属于相同的连通分量，则说明在遍历到当前的边之前，这两个顶点之间已经连通，因此当前的边导致环出现，为附加的边，将当前的边作为答案返回。

作者：力扣官方题解 链接：https://leetcode.cn/problems/redundant-connection/solutions/557616/rong-yu-lian-jie-by-leetcode-solution-pks2/

```
class Solution(object):
    def findRedundantConnection(self, edges):
        """
        :type edges: List[List[int]]
        :rtype: List[int]
        """
        n = len(edges)
        parent = list(range(n + 1))

        def find(index):
            if parent[index] != index:
                parent[index] = find(parent[index])
            return parent[index]
        
        def union(index1, index2):
            parent[find(index1)] = find(index2)

        for node1, node2 in edges:
            if find(node1) != find(node2):
                union(node1, node2)
            else:
                return [node1, node2]
        
        return []
```

## **3216. 交换后字典序最小的字符串**

### 问题描述

给你一个仅由数字组成的字符串 `s`，在最多交换一次 **相邻** 且具有相同 **奇偶性** 的数字后，返回可以得到的字典序最小的字符串。

如果两个数字都是奇数或都是偶数，则它们具有相同的奇偶性。例如，5 和 9、2 和 4 奇偶性相同，而 6 和 9 奇偶性不同。

### 示例

- 示例1

  输入： s = "45320" 输出： "43520" 解释： s[1] == '5' 和 s[2] == '3' 都具有相同的奇偶性，交换它们可以得到字典序最小的字符串。

- 示例2

  输入： s = "001" 输出： "001" 解释： 无需进行交换，因为 s 已经是字典序最小的。

### 解题思路

从头到尾遍历字符串，如果遇到两个相邻元素奇偶性相同就判断元素大小，然后根据大小关系交换！

```python
class Solution(object):
    def getSmallestString(self, s):
        """
        :type s: str
        :rtype: str
        """
        s = list(s)
        for i in range(len(s)-1):
            if(((int(s[i])%2==0 and int(s[i+1]))%2==0) or((int(s[i])%2==1 and int(s[i+1]))%2==1)):
                if(int(s[i]) > int(s[i+1])):
                    temp = s[i]
                    s[i] = s[i+1]
                    s[i+1] = temp
                    break
        return ''.join(s)
```

当输入s=‘10‘的时候，出错了！检查一下，发现是写错了！

```python
class Solution(object):
    def getSmallestString(self, s):
        """
        :type s: str
        :rtype: str
        """
        s = list(s)
        for i in range(len(s)-1):
            if i==len(s)-1:
                break
            if((int(s[i])%2==0 and int(s[i+1])%2==0) or((int(s[i])%2==1 and int(s[i+1]))%2==1)):
                if(int(s[i]) > int(s[i+1])):
                    temp = s[i]
                    s[i] = s[i+1]
                    s[i+1] = temp
                    break
        return ''.join(s)
```

### 题解

因为要字典序最小，所以我们要尽可能交换s前面的数字。 我们用类似于pairwise的方式遍历s，记遍历值为a, b，那么当我们最先遇到a与b奇偶性相同，且a > b时，交换a与b即可得到答案。

作者：倚窗同学 链接：https://leetcode.cn/problems/lexicographically-smallest-string-after-a-swap/solutions/2970673/pythonbian-li-by-qzddmyc-ksik/

## **3165. 不包含相邻元素的子序列的最大和**

### 问题描述

给你一个整数数组 `nums` 和一个二维数组 `queries`，其中 `queries[i] = [posi, xi]`。

对于每个查询 `i`，首先将 `nums[posi]` 设置为 `xi`，然后计算查询 `i` 的答案，该答案为 `nums` 中 **不包含相邻元素** 的子序列的**最大**和。

返回所有查询的答案之和。

由于最终答案可能非常大，返回其对 `$10^9 + 7$` **取余** 的结果。

**子序列** 是指从另一个数组中删除一些或不删除元素而不改变剩余元素顺序得到的数组。

### 示例

- 示例1

  输入：nums = [3,5,9], queries = [[1,-2],[0,-3]]

  输出：21

  解释： 执行第 1 个查询后，nums = [3,-2,9]，不包含相邻元素的子序列的最大和为 3 + 9 = 12。 执行第 2 个查询后，nums = [-3,-2,9]，不包含相邻元素的子序列的最大和为 9 。

- 示例2

  输入：nums = [0,-1], queries = [[0,-5]]

  输出：0

  解释： 执行第 1 个查询后，nums = [-5,-1]，不包含相邻元素的子序列的最大和为 0（选择空子序列）。

### 解题思路

先按照题目要求依次进行查询，然后求不包含相邻元素的子序列的最大和！难点就在如何求不包含相邻元素的子序列的最大和！这应该是一个动态规划的问题，早在考研的时候就想仔细学学动态规划，觉得写代码题会有用，但是也没学明白！干脆今天慢慢开始学！笔记在[动态规划学习笔记 | Malaxg](https://www.malaxg.top/article/dp-learning)

### 题解

本题是可以修改数组元素值的 198. 打家劫舍。

为了解决本题，首先来换一个角度，用分治的思想解决打家劫舍。

设 f(A) 为数组 A 的打家劫舍的答案。把 nums 从中间切开，分成左右两个子数组，分别记作 a 和 b。要计算 f(nums)，看上去，我们只需要分别计算 f(a) 和 f(b)。但这是不对的，万一同时选了 a 的最后一个数和 b 的第一个数，就不满足题目要求了。

怎么办？要么不选 a 的最后一个数，要么不选 b 的第一个数。所以加个约束，先来（非正式地）讨论一下：

约束 a 的最后一个数一定不选，即 $f(nums)=f(a′)+f(b)$，其中 $a′$ 是 a 去掉最后一个数后的数组。 约束 b 的第一个数一定不选，即 $f(nums)=f(a)+f(′b)$，其中$′b$ 表示 b 去掉第一个数后的数组。 两种情况取最大值，即$f(nums)=max(f(a′)+f(b),f(a)+f(′b))$ 继续计算下去，就需要进一步地把 a′,b,a, ′b 切开，继续分类讨论。

为方便描述，定义：

- $f_{00}(A)$表示在 *A* 第一个数一定不选，最后一个数也一定不选的情况下，打家劫舍的答案。
- $f_{01}(A)$表示在 *A* 第一个数一定不选，最后一个数可选可不选的情况下，打家劫舍的答案。
- $f_{10}(A)$表示在 *A* 第一个数可选可不选，最后一个数一定不选的情况下，打家劫舍的答案。
- $f_{11}(A)$表示在 *A* 第一个数可选可不选，最后一个数也可选可不选的情况下，打家劫舍的答案。

前文的分类讨论可以（正式地）表述为

$$ f_{11}(nums) = max(f_{10}(a) + f_{11}(a) + f_{01}(b)) $$

$f_{10}$和$f_{01}$又该如何计算呢？以$f_{10}(a)$为例，把$a$分成左右两个数组$p$和$q$，分类讨论：

- 约束$p$的最后一个数一定不选，即 $f_{10}(a) = f_{10}(p) + f_{10}(q)$，注意$q$的最后一个数也不能选，因为我们计算的是$f_{10}(a)$,$a$的最后一个数一定不能选。
- 约束$q$的第一个数一定不选，即$f_{10}(a) = f_{11}(p)+f_{00}(q)$

两种情况取最大值，得$f_{10}(a)=max(f_{10}(p)+f_{10}(q), f_{11}(p)+f_{00}(q))$ 同理可以得到 $f_{00}$ 和 $f_{01}$ 。

综上所述：

$$
 f_{00}(a)=\max(f_{00}(p)+f_{10}(q),\:f_{01}(p)+f_{00}(q))\\f_{01}(a)=\max(f_{00}(p)+f_{11}(q),\:f_{01}(p)+f_{01}(q))\\f_{10}(a)=\max(f_{10}(p)+f_{10}(q),\:f_{11}(p)+f_{00}(q))\\f_{11}(a)=\max(f_{10}(p)+f_{11}(q),\:f_{11}(p)+f_{01}(q))
$$
这样就可以分治计算 *nums* 的打家劫舍了。

递归边界：如果 a 的长度等于 1，那么按照定义，$f_{11}(a)=max(a[0],0)$，其余$f_{00}$,$f_{01}$,$f_{10}$均为0。

回到本题：

对于修改操作，我们可以用线段树的单点修改实现，线段树的每个节点维护对应区间的$f_{00} ,f_{01},f_{10},f_{11}$。 对于查询操作，由于询问的是整个数组，询问结果就是线段树根节点的$f_{11}$，累加到答案中。

作者：灵茶山艾府 链接：https://leetcode.cn/problems/maximum-sum-of-subsequence-with-non-adjacent-elements/solutions/2790603/fen-zhi-si-xiang-xian-duan-shu-pythonjav-xnhz/

## **3259. 超级饮料的最大强化能量**

### 问题描述

来自未来的体育科学家给你两个整数数组 `energyDrinkA` 和 `energyDrinkB`，数组长度都等于 `n`。这两个数组分别代表 A、B 两种不同能量饮料每小时所能提供的强化能量。

你需要每小时饮用一种能量饮料来 **最大化** 你的总强化能量。然而，如果从一种能量饮料切换到另一种，你需要等待一小时来梳理身体的能量体系（在那个小时里你将不会获得任何强化能量）。

返回在接下来的 `n` 小时内你能获得的 **最大** 总强化能量。

**注意** 你可以选择从饮用任意一种能量饮料开始。

### 示例

- 示例1

  **输入**：energyDrinkA = [1,3,1], energyDrinkB = [3,1,1] **输出**：5 **解释**： 要想获得 5 点强化能量，需要选择只饮用能量饮料 A（或者只饮用 B）。

- 示例2

  **输入：**energyDrinkA = [4,1,1], energyDrinkB = [1,1,3]

  **输出：**7

  **解释：**

  - 第一个小时饮用能量饮料 A。
  - 切换到能量饮料 B ，在第二个小时无法获得强化能量。
  - 第三个小时饮用能量饮料 B ，并获得强化能量。

### 解题思路/题解

给你两个数组 a 和 b。从 i=0 开始，要么选 a[i]，要么选 b[i]。如果你当前选了 a 中的元素，后面想选 b 中的元素，那么下一个元素必须不能选。例如现在选 a[i]，那么后面可以选 b[i+2]，但不能选 b[i+1]。

返回所选元素之和的最大值。

题目给定了两个长度为 n 的数组 energyDrinkA 和 energyDrinkB（我们这里简称 A 和 B），你需要从左到右依次取 n 个数字，每次可以从 A 取或者从 B 取，但如果上一次（第 i−1 次）是从 A 取的，那么第 i 次要切换就必须暂停一次，接着在第 i+1 次可以从 B 取。求可以取得的数字之和的最大值。

可以发现每次取数所得的和只与前面一步怎么决策有关，如果上一步切换，那么计算和的时候从上两步计算，否则从上一步计算，因此我们设计 d[i][0/1] 分别表示第 i 步从 A 或者从 B 取数时，可以获得的最大值。这样一来，我们可以得到相应的转移方程：

$$
d[i][0]=max(d[i−1][0],d[i−2][1])+A[i] d[i][1]=max(d[i−1][1],d[i−2][0])+B[i]
$$
注意 i 在 1 和 2 时的特殊情况（起始下标设置为 1）。 最终我们选择 max(d[n][0],d[n][1]) 作为答案。

作者：力扣官方题解 链接：https://leetcode.cn/problems/maximum-energy-boost-from-two-drinks/solutions/2969563/chao-ji-yin-liao-de-zui-da-qiang-hua-nen-7s2a/ 来源：力扣（LeetCode）

```python
class Solution(object):
    def maxEnergyBoost(self, energyDrinkA, energyDrinkB):
        """
        :type energyDrinkA: List[int]
        :type energyDrinkB: List[int]
        :rtype: int
        """
        n = len(energyDrinkA)
        d = [[0, 0] for _ in range(n + 1)]
        for i in range(1, n + 1):
            d[i][0] = d[i - 1][0] + energyDrinkA[i - 1]
            d[i][1] = d[i - 1][1] + energyDrinkB[i - 1]
            if i >= 2:
                d[i][0] = max(d[i][0], d[i - 2][1] + energyDrinkA[i - 1])
                d[i][1] = max(d[i][1], d[i - 2][0] + energyDrinkB[i - 1])
        return max(d[n][0], d[n][1])
```

## **638. 大礼包**

### 问题描述

在 LeetCode 商店中， 有 `n` 件在售的物品。每件物品都有对应的价格。然而，也有一些大礼包，每个大礼包以优惠的价格捆绑销售一组物品。

给你一个整数数组 `price` 表示物品价格，其中 `price[i]` 是第 `i` 件物品的价格。另有一个整数数组 `needs` 表示购物清单，其中 `needs[i]` 是需要购买第 `i` 件物品的数量。

还有一个数组 `special` 表示大礼包，`special[i]` 的长度为 `n + 1` ，其中 `special[i][j]` 表示第 `i` 个大礼包中内含第 `j` 件物品的数量，且 `special[i][n]` （也就是数组中的最后一个整数）为第 `i` 个大礼包的价格。

返回 **确切** 满足购物清单所需花费的最低价格，你可以充分利用大礼包的优惠活动。你不能购买超出购物清单指定数量的物品，即使那样会降低整体价格。任意大礼包可无限次购买。

### 示例

- 示例1

  输入：price = [2,5], special = [[3,0,5],[1,2,10]], needs = [3,2] 输出：14 解释：有 A 和 B 两种物品，价格分别为 ¥2 和 ¥5 。 大礼包 1 ，你可以以 ¥5 的价格购买 3A 和 0B 。 大礼包 2 ，你可以以 ¥10 的价格购买 1A 和 2B 。 需要购买 3 个 A 和 2 个 B ， 所以付 ¥10 购买 1A 和 2B（大礼包 2），以及 ¥4 购买 2A 。

- 示例2

  输入：price = [2,3,4], special = [[1,1,0,4],[2,2,1,9]], needs = [1,2,1] 输出：11 解释：A ，B ，C 的价格分别为 ¥2 ，¥3 ，¥4 。 可以用 ¥4 购买 1A 和 1B ，也可以用 ¥9 购买 2A ，2B 和 1C 。 需要买 1A ，2B 和 1C ，所以付 ¥4 买 1A 和 1B（大礼包 1），以及 ¥3 购买 1B ， ¥4 购买 1C 。 不可以购买超出待购清单的物品，尽管购买大礼包 2 更加便宜。

### 解题思路

回溯+ 备忘录优化

来源：[638. 大礼包 - 力扣（LeetCode）](https://leetcode.cn/problems/shopping-offers/solutions/618144/python3-95-by-distort-9d8n/)

```python
class Solution(object):
    def shoppingOffers(self, price, special, needs):
        """
        :type price: List[int]
        :type special: List[List[int]]
        :type needs: List[int]
        :rtype: int
        """
        memo = {}
        # 给定物品价格，大礼包分布和需要的物品数量，返回最少花费
        def shop(price, special, needs):
            if not any(needs):
                return 0
            if str(needs) in memo:
                return memo[str(needs)]
            # 不选择礼包
            cost = sum([ price[i] * needs[i] for i in range(len(needs))])
            # 选择大礼包
            for sp in special:
                need = []
                for i in range(len(needs)):
                    if needs[i] - sp[i] < 0:
                        break
                    else:
                        need.append(needs[i] - sp[i])
                if len(needs) == len(need):
                    cost = min(cost, sp[-1] + shop(price, special, need))
            memo[str(needs)] = cost
            return cost
        return shop(price, special, needs)
```

## **633. 平方数之和**

### 问题描述

给定一个非负整数 `c` ，你要判断是否存在两个整数 `a` 和 `b`，使得 `$a^2 + b^2 = c$`。

### 示例

- 示例1

  输入：c = 5 输出：true 解释：1 * 1 + 2 * 2 = 5

- 示例2

  输入：c = 3 输出：false

### 解题思路/题解

看到题目一想就是暴力，这样会超出时间限制！

```python
class Solution(object):
    def judgeSquareSum(self, c):
        """
        :type c: int
        :rtype: bool
        """
        for a in range(c+1):
            for b in range(c+1):
                # print(a,b)
                if(a*a+b*b==c):
                    return True
        return False
```

所以需要修改思路，不能采用暴力的思想，分为暴力枚举和双指针。

题解：[633. 平方数之和 - 力扣（LeetCode）](https://leetcode.cn/problems/sum-of-square-numbers/solutions/2887457/633-ping-fang-shu-zhi-he-by-stormsunshin-oxy4/)

```python
class Solution:
    def judgeSquareSum(self, c: int) -> bool:
        a, b = 0, floor(sqrt(c))
        while a <= b:
            sum = a ** 2 + b ** 2
            if sum == c:
                return True
            elif sum < c:
                a += 1
            else:
                b -= 1
        return False
```

## **3255. 长度为 K 的子数组的能量值 II**

### 问题描述

给你一个长度为 `n` 的整数数组 `nums` 和一个正整数 `k` 。

一个数组的 **能量值** 定义为：

- 如果 **所有** 元素都是依次 **连续** 且 **上升** 的，那么能量值为 **最大** 的元素。
- 否则为 -1 。

你需要求出 `nums` 中所有长度为 `k` 的子数组的能量值。

请你返回一个长度为 `n - k + 1` 的整数数组 `results` ，其中 `results[i]` 是子数组 `nums[i..(i + k - 1)]` 的能量值。

### 示例

- 示例一

  输入：nums = [1,2,3,4,3,2,5], k = 3

  输出：[3,4,-1,-1,-1]

  解释：

  nums 中总共有 5 个长度为 3 的子数组：

  [1, 2, 3] 中最大元素为 3 。 [2, 3, 4] 中最大元素为 4 。 [3, 4, 3] 中元素 不是 连续的。 [4, 3, 2] 中元素 不是 上升的。 [3, 2, 5] 中元素 不是 连续的。

- 示例二

  输入：nums = [2,2,2,2,2], k = 4 输出：[-1,-1]

### 解题思路

核心思路：找连续上升的段。如果段长至少是 k，那么这段中的所有长为 k 的子数组都是符合要求的，子数组的最后一个元素是最大的。

具体来说，遍历数组的同时，用一个计数器 cnt 统计连续递增的元素个数：

初始化 cnt=0。 如果 i=0 或者 nums[i]=nums[i−1]+1，把 cnt 增加 1；否则，把 cnt 置为 1。 如果发现 cnt≥k，那么下标从 i−k+1 到 i 的这个子数组的能量值为 nums[i]，即 ans[i−k+1]=nums[i]。 具体例子请看 视频讲解，欢迎关注~

作者：灵茶山艾府 链接：https://leetcode.cn/problems/find-the-power-of-k-size-subarrays-ii/solutions/2884189/on-jian-ji-xie-fa-pythonjavacgo-by-endle-gtek/ 来源：力扣（LeetCode） 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

```python
class Solution(object):
    def resultsArray(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: List[int]
        """
        ans = [-1] * (len(nums) - k + 1)
        cnt = 0
        for i, x in enumerate(nums):
            cnt = cnt + 1 if i == 0 or x == nums[i - 1] + 1 else 1
            if cnt >= k:
                ans[i - k + 1] = x
        return ans
```

## **3235. 判断矩形的两个角落是否可达**

### 问题描述

给你两个正整数 `xCorner` 和 `yCorner` 和一个二维整数数组 `circles` ，其中 `circles[i] = [xi, yi, ri]` 表示一个圆心在 `(xi, yi)` 半径为 `ri` 的圆。

坐标平面内有一个左下角在原点，右上角在 `(xCorner, yCorner)` 的矩形。你需要判断是否存在一条从左下角到右上角的路径满足：路径 **完全** 在矩形内部，**不会** 触碰或者经过 **任何** 圆的内部和边界，同时 **只** 在起点和终点接触到矩形。

如果存在这样的路径，请你返回 `true` ，否则返回 `false` 。

### 示例

- 示例1

  输入：X = 3, Y = 4, circles = [[2,1,1]] 输出：true 解释：黑色曲线表示一条从 `(0, 0)` 到 `(3, 4)` 的路径。

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/eb585d7d-0842-4669-92db-014a01f1a8c6/image.png)

- 示例2

  输入：X = 3, Y = 3, circles = [[1,1,2]] 输出：false 解释：不存在从 `(0, 0)` 到 `(3, 3)` 的路径。

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/32d623a1-2490-47c2-815b-eb875d32a178/image.png)

- 示例3

  输入：X = 3, Y = 3, circles = [[2,1,1],[1,2,1]] 输出：false 解释：不存在从 `(0, 0)` 到 `(3, 3)` 的路径。

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/ecb9f492-4934-4d0b-9cb4-023ce31726cc/image.png)

- 示例4

  输入：X = 4, Y = 4, circles = [[5,5,1]] 输出：true 解释：

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/149f8b66-938b-4dae-9186-ba3ff2b0b451/image.png)

### 解题思路/题解

首先我们考虑 circles 只有一个圆。在三种情况下，不存在这样的路径：

起点在圆内或圆上（圆内或圆上不影响结果，下文均简写为圆内）； 终点在圆内； 圆与矩形的左侧边或者上侧边相切或相交（相切或相交不影响结果，下文均简写为相交），并且圆与矩形的右侧边或者下侧边相交。 前两种情况好理解，题目规定路径不能触碰圆的边界或者内部，因此当起点或者终点位于圆内时，肯定不存在这样的路径。第三种情况，圆作为一个单一的障碍物，既与矩形的左上边界相交，又与矩形的右下边界相交，使得不存在一条路径可以从矩形内部经过，又不碰触圆的边界或者内部。

接下来，考虑 circles 中有多个圆的情况。类似的，上文讨论的前两种情况仍然成立，当起点或终点位于某一个圆内时，不存在这样的路径。当有多个圆时，圆可能会在矩形内相交。当圆在矩形内相交时，这些圆可以合并成为一个障碍物，当这个合并的障碍物也满足与矩形的左上边界相交，且与矩形的右下边界相交，那么就不存在所求的路径。当圆的相交区域在矩形之外时，这些圆不能进行合并，因为相交的区域不会阻止在矩形内部的路径的生成。这样，我们可以从与矩形的左上边界相交的圆形开始深度优先搜索，然后遍历可以与当前圆合并的相邻点，重复如此，直到遍历到与矩形的右下边界相交的圆，这样的情况下，就表示不存在所求的路径。如果遍历完所有的圆后，都无法遍历到与矩形的右下边界相交的圆，那么表示存在所求的路径。

特殊情况是圆的相交区域的一部分位于矩形内，另一部分位于矩形外，即圆的相交区域又和矩形的边相交。按照上述的推导，这些圆需要进行合并。在实际中，我们可以对它们进行合并，也可以不对它们进行合并。理由如下：当两个圆的相交区和矩形的边相交时，这两个圆本身就都与矩形的边相交。如果这两个圆本身就都与矩形的左上边界相交，那么我们可以不合并它们，因为它们可以各自作为起点。如果这两个圆本身就都与矩形的右下边界相交，那么我们可以不合并它们，因为它们可以各自作为终点。不存在其中一个圆仅与矩形的左上边界相交，另一个圆仅与矩形的右下边界相交的情况。这就得出了接下来的结论：判断两个相交的圆是否需要合并时，我们可以任取相交区域的一点，如果点在矩形内部，那么我们合并它们，否则不合并。

接下来，我们需要将上文提到的几何关系用代码来表示。

首先是判断点是否在圆内，这个我们可以用点到圆心的距离和半径进行比较。为了避免精度问题，我们可以将两边都进行平方操作。

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/5f41658b-01c8-4392-892c-af38c19aaefb/image.png)

接下来是判断圆是否与矩形的左上边界相交。记圆心的坐标为 (x,y)，半径为 r。圆心需要位于图中的两个实线矩形或者圆内。在已经满足圆不包含起点或者终点的情况下，前一句话的条件已经足够。

判断圆是否与右下边界相交，与上面的判断类似。

最后是判断两个相交的圆内的任意一点是否与位于矩形内部。记两个圆的圆心分别为$O_1$，$O_2$，坐标分别为($x_1$,$y_1$)，($x_2$$,$$y_2$)，半径分别为$r_1$,$r_2$。首先判断两圆是否相交，也可以用园心之间的距离与半径之和进行比较。如果相交，再在线段$O_1O_2$取点A满足$\frac{|O_1A|}{|O_1O_2|}=\frac{r_1}{r_1+r_2}$。这样点A一定满足在两圆的相交区域，因为点A到两个圆心的距离均小于等于对应的半径。计算得到点A的坐标为$(\frac{x_1\times r_2+x_2\times r_1}{r_1+r_2},\frac{y_1\times r_2+y_2\times r_1}{r_1+r_2})$。再判断这个点是否位于矩形内部即呵可。这样选取点A，也可以避免精度带来的误差。

```python
class Solution(object):
    def canReachCorner(self, xCorner, yCorner, circles):
        """
        :type xCorner: int
        :type yCorner: int
        :type circles: List[List[int]]
        :rtype: bool
        """
        def pointInCircle(px, py, x, y, r):
            return (x - px) ** 2 + (y - py) ** 2 <= r ** 2

        def circleIntersectsTopLeftOfRectangle(x, y, r, xCorner, yCorner):
            return (abs(x) <= r and 0 <= y <= yCorner) or (0 <= x <= xCorner and abs(y - yCorner) <= r) or pointInCircle(x, y, 0, yCorner, r)
        
        def circleIntersectsBottomRightOfRectangle(x, y, r, xCorner, yCorner):
            return (abs(y) <= r and 0 <= x <= xCorner) or (0 <= y <= yCorner and abs(x - xCorner) <= r) or (pointInCircle(x, y, xCorner, 0, r))

        def circlesIntersectInRectangle(x1, y1, r1, x2, y2, r2, xCorner, yCorner):
            return (x1 - x2) ** 2 + (y1 - y2) ** 2 <= (r1 + r2) ** 2 and x1 * r2 + x2 * r1 < (r1 + r2) * xCorner and y1 * r2 + y2 * r1 < (r1 + r2) * yCorner

        visited = [False] * len(circles)
        def dfs(i):
            x1, y1, r1 = circles[i]
            if circleIntersectsBottomRightOfRectangle(x1, y1, r1, xCorner, yCorner):
                return True
            visited[i] = True
            for j, (x2, y2, r2) in enumerate(circles):
                if (not visited[j]) and circlesIntersectInRectangle(x1, y1, r1, x2, y2, r2, xCorner, yCorner) and dfs(j):
                    return True
            return False

        for i, (x, y, r) in enumerate(circles):
            if pointInCircle(0, 0, x, y, r) or pointInCircle(xCorner, yCorner, x, y, r):
                return False
            if (not visited[i]) and circleIntersectsTopLeftOfRectangle(x, y, r, xCorner, yCorner) and dfs(i):
                return False
        return True
```

作者：力扣官方题解 链接：https://leetcode.cn/problems/check-if-the-rectangle-corner-is-reachable/solutions/2973328/pan-duan-ju-xing-de-liang-ge-jiao-luo-sh-ug24/ 来源：力扣（LeetCode）

## **540. 有序数组中的单一元素**

### 问题描述

给你一个仅由整数组成的有序数组，其中每个元素都会出现两次，唯有一个数只会出现一次。

请你找出并返回只出现一次的那个数。

你设计的解决方案必须满足 `O(log n)` 时间复杂度和 `O(1)` 空间复杂度。

### 示例

- 示例1

  输入: nums = [1,1,2,3,3,4,4,8,8] 输出: 2

- 示例2

  输入: nums =  [3,3,7,7,10,11,11] 输出: 10

### 解题思路/题解

按照两步一匹配的思路，第一个和第二个比，第三个和第四个比，这样一直比较下去，如果遇到不一样的就返回第一个。提交！发现错了，没考虑到只有1个元素的时候，也没考虑到元素在末尾的时候。

**题解：**

题目有两个已知条件：

数组是有序的。 除了一个数出现一次外，其余每个数都出现两次。 第二个条件意味着，数组的长度一定是奇数。

第一个条件意味着，出现两次的数，必然相邻，不可能出现 1,2,1 这样的顺序。

这也意味着，只出现一次的那个数，一定位于偶数下标上。

这启发我们去检查偶数下标 2k。

示例 1 的 nums=[1,1,2,3,3,4,4,8,8]：

如果 $nums[2k]=nums[2k+1]$，说明只出现一次的数的下标 >2k。 如果 $nums[2k] \neq nums[2k+1]$，说明只出现一次的数的下标 ≤2k。 也就是说，随着 k 的变大，不等式 $nums[2k]\neq nums[2k+1]$ 越可能满足，有单调性，可以二分。

```python
class Solution(object):
    def singleNonDuplicate(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        left, right = -1, len(nums) // 2
        while left + 1 < right:
            mid = (left + right) // 2
            if nums[mid * 2] != nums[mid * 2 + 1]:
                right = mid
            else:
                left = mid
        return nums[right * 2]
```

作者：灵茶山艾府 链接：https://leetcode.cn/problems/single-element-in-a-sorted-array/solutions/2983333/er-fen-xing-zhi-fen-xi-jian-ji-xie-fa-py-0rng/

## **3258. 统计满足 K 约束的子字符串数量 I**

### 问题描述

给你一个 **二进制** 字符串 `s` 和一个整数 `k`。

如果一个 **二进制字符串** 满足以下任一条件，则认为该字符串满足 **k 约束**：

- 字符串中 `0` 的数量最多为 `k`。
- 字符串中 `1` 的数量最多为 `k`。

返回一个整数，表示 `s` 的所有满足 **k 约束** 的子字符串的数量。

### 示例

- 示例1

  输入：s = "10101", k = 1

  输出：12

  解释：s 的所有子字符串中，除了 "1010"、"10101" 和 "0101" 外，其余子字符串都满足 k 约束。

- 示例2

  输入：s = "1010101", k = 2

  输出：25

  解释：s 的所有子字符串中，除了长度大于 5 的子字符串外，其余子字符串都满足 k 约束。

- 示例3

  输入：s = "11111", k = 1 输出：15 解释：s 的所有子字符串都满足 k 约束。

### 解题思路

首先找到字符串s的所有子串，然后判断每一个子串是否满足条件！

```python
class Solution(object):
    def countKConstraintSubstrings(self, s, k):
        """
        :type s: str
        :type k: int
        :rtype: int
        """
        result = 0
        s_1 = [s[i:i + x + 1] for x in range(len(s)) for i in range(len(s) - x)]
        for t in s_1:
            if t.count('1')<=k or t.count('0')<=k:
                result+=1
        return result
```

### 题解

计算以 0 为右端点的合法子串个数，以 1 为右端点的合法子串个数，……，以 *n*−1 为右端点的合法子串个数。 我们需要知道以 *i* 为右端点的合法子串，其左端点最小是多少。 由于随着 *i* 的变大，窗口内的字符数量变多，越不能满足题目要求，所以最小左端点会随着 *i* 的增大而增大，有**单调性**，因此可以用 [滑动窗口](https://leetcode.cn/link/?target=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2FBV1hd4y1r7Gq%2F) 计算。 设以 *i* 为右端点的合法子串，其左端点**最小**是 $left_i$ 那么以 *i* 为右端点的合法子串，其左端点可以是 $left_i$,$left_i+1$,……,$i$，一共$i-left+1$个，累加到答案中。

细节：字符 0 的 ASCII 值是偶数，字符 1 的 ASCII 值是奇数，所以可以用 ASCII 值 cmod2 得到对应的数字。这也等价于和 1 计算 AND。

作者：灵茶山艾府 链接：https://leetcode.cn/problems/count-substrings-that-satisfy-k-constraint-i/solutions/2884495/on-hua-dong-chuang-kou-pythonjavacgo-by-keubv/

## **661. 图片平滑器**

### 问题描述

**图像平滑器** 是大小为 `3 x 3` 的过滤器，用于对图像的每个单元格平滑处理，平滑处理后单元格的值为该单元格的平均灰度。

每个单元格的 **平均灰度** 定义为：该单元格自身及其周围的 8 个单元格的平均值，结果需向下取整。（即，需要计算蓝色平滑器中 9 个单元格的平均值）。

如果一个单元格周围存在单元格缺失的情况，则计算平均灰度时不考虑缺失的单元格（即，需要计算红色平滑器中 4 个单元格的平均值）。

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/7f07b0dd-6df3-47ad-80f6-faee1e9482e8/image.png)

给你一个表示图像灰度的 `m x n` 整数矩阵 `img` ，返回对图像的每个单元格平滑处理后的图像 。

### 示例

- 示例1

  输入:img = [[1,1,1],[1,0,1],[1,1,1]] 输出:[[0, 0, 0],[0, 0, 0], [0, 0, 0]] 解释: 对于点 (0,0), (0,2), (2,0), (2,2): 平均(3/4) = 平均(0.75) = 0 对于点 (0,1), (1,0), (1,2), (2,1): 平均(5/6) = 平均(0.83333333) = 0 对于点 (1,1): 平均(8/9) = 平均(0.88888889) = 0

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/e68c25c0-b3be-439e-87b3-c87b6acbd48e/image.png)

- 示例2

  输入: img = [[100,200,100],[200,50,200],[100,200,100]] 输出: [[137,141,137],[141,138,141],[137,141,137]] 解释: 对于点 (0,0), (0,2), (2,0), (2,2): floor((100+200+200+50)/4) = floor(137.5) = 137 对于点 (0,1), (1,0), (1,2), (2,1): floor((200+200+50+200+100+100)/6) = floor(141.666667) = 141 对于点 (1,1): floor((50+200+200+200+200+100+100+100+100)/9) = floor(138.888889) = 138

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/026712c6-c00e-4596-94ed-eb0062b79d8a/image.png)

### 解题思路/题解

按照题目的要求，我们直接依次计算每一个位置平滑处理后的结果即可。

具体地，对于位置 (i,j)，我们枚举其周围的九个单元是否存在，对于存在的单元格，我们统计其数量 num 与总和 sum，那么该位置平滑处理后的结果即为 $\lfloor\frac{sum}{num}\rfloor$

作者：力扣官方题解 链接：https://leetcode.cn/problems/image-smoother/solutions/1358687/tu-pian-ping-hua-qi-by-leetcode-solution-9oi5/

```python
class Solution(object):
    def imageSmoother(self, img):
        """
        :type img: List[List[int]]
        :rtype: List[List[int]]
        """
        m, n = len(img), len(img[0])
        ans = [[0] * n for _ in range(m)]
        for i in range(m):
            for j in range(n):
                tot, num = 0, 0
                for x in range(max(i - 1, 0), min(i + 2, m)):
                    for y in range(max(j - 1, 0), min(j + 2, n)):
                        tot += img[x][y]
                        num += 1
                ans[i][j] = tot // num
        return ans
```

## **3243. 新增道路查询后的最短距离 I**

### 问题描述

给你一个整数 `n` 和一个二维整数数组 `queries`。

有 `n` 个城市，编号从 `0` 到 `n - 1`。初始时，每个城市 `i` 都有一条**单向**道路通往城市 `i + 1`（ `0 <= i < n - 1`）。

`queries[i] = [ui, vi]` 表示新建一条从城市 `ui` 到城市 `vi` 的**单向**道路。每次查询后，你需要找到从城市 `0` 到城市 `n - 1` 的**最短路径**的**长度**。

返回一个数组 `answer`，对于范围 `[0, queries.length - 1]` 中的每个 `i`，`answer[i]` 是处理完**前** `i + 1` 个查询后，从城市 `0` 到城市 `n - 1` 的最短路径的*长度*。

### 示例

- 示例1

  **输入：** n = 5, queries = [[2, 4], [0, 2], [0, 4]]

  **输出：** [3, 2, 1]

  **解释：**

  ![img](https://assets.leetcode.com/uploads/2024/06/28/image8.jpg)

  新增一条从 2 到 4 的道路后，从 0 到 4 的最短路径长度为 3。

  ![img](https://assets.leetcode.com/uploads/2024/06/28/image9.jpg)

  新增一条从 0 到 2 的道路后，从 0 到 4 的最短路径长度为 2。

  ![img](https://assets.leetcode.com/uploads/2024/06/28/image10.jpg)

  新增一条从 0 到 4 的道路后，从 0 到 4 的最短路径长度为 1。

- 示例2

  **输入：** n = 4, queries = [[0, 3], [0, 2]]

  **输出：** [1, 1]

  **解释：**

  ![img](https://assets.leetcode.com/uploads/2024/06/28/image11.jpg)

  新增一条从 0 到 3 的道路后，从 0 到 3 的最短路径长度为 1。

  ![img](https://assets.leetcode.com/uploads/2024/06/28/image12.jpg)

  新增一条从 0 到 2 的道路后，从 0 到 3 的最短路径长度仍为 1。

### 解题思路

暴力。每次加边后，重新跑一遍 BFS，求出从 0 到 n−1 的最短路。为避免反复创建 vis 数组，可以在 vis 中保存当前节点是第几次询问访问的。

作者：灵茶山艾府 链接：https://leetcode.cn/problems/shortest-distance-after-road-addition-queries-i/solutions/2869215/liang-chong-fang-fa-bfs-dppythonjavacgo-mgunf/

```python
class Solution(object):
    def shortestDistanceAfterQueries(self, n, queries):
        """
        :type n: int
        :type queries: List[List[int]]
        :rtype: List[int]
        """
        g = [[i + 1] for i in range(n - 1)]
        vis = [-1] * (n - 1)

        def bfs(i):
            q = deque([0])
            for step in count(1):
                tmp = q
                q = []
                for x in tmp:
                    for y in g[x]:
                        if y == n - 1:
                            return step
                        if vis[y] != i:
                            vis[y] = i
                            q.append(y)
            return -1

        ans = [0] * len(queries)
        for i, (l, r) in enumerate(queries):
            g[l].append(r)
            ans[i] = bfs(i)
        return ans
```

## **3248. 矩阵中的蛇**

### 问题描述

大小为 `n x n` 的矩阵 `grid` 中有一条蛇。蛇可以朝 **四个可能的方向** 移动。矩阵中的每个单元格都使用位置进行标识： `grid[i][j] = (i * n) + j`。

蛇从单元格 0 开始，并遵循一系列命令移动。

给你一个整数 `n` 表示 `grid` 的大小，另给你一个字符串数组 `commands`，其中包括 `"UP"`、`"RIGHT"`、`"DOWN"` 和 `"LEFT"`。题目测评数据保证蛇在整个移动过程中将始终位于 `grid` 边界内。

返回执行 `commands` 后蛇所停留的最终单元格的位置。

### 示例

- 示例1

  **输入：**n = 2, commands = ["RIGHT","DOWN"]

  **输出：**3

  **解释：**

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/08165c5a-abf1-4edf-8130-8a0d57f3c081/image.png)

- 示例2

  **输入：**n = 3, commands = ["DOWN","RIGHT","UP"]

  **输出：**1

  **解释：**

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/cd19e6af-c545-4377-a43a-d85f6df82c46/image.png)

### 解题思路

先生成这样一个数组，然后再根据commands的操作进行依次的操作！

```python
class Solution(object):
    def finalPositionOfSnake(self, n, commands):
        """
        :type n: int
        :type commands: List[str]
        :rtype: int
        """
        grid = [[0] * n for _ in range(n)]
        k=0
        for i in range(n):
            for j in range(n):
                grid[i][j] = k
                k+=1
        op1 = op2 = 0
        for control in commands:
            if control == 'UP':
                op1 -= 1
            if control == 'DOWN':
                op1 += 1
            if control == 'RIGHT':
                op2 += 1
            if control == 'LEFT':
                op2 -= 1
        return grid[op1][op2]
```

**注意！！！**

------

初始化二维数组的时候，不能用`[[0]*]*n`，这样的写法会导致所有行共享同一个引用，即所有行指向相同的内存地址。这会引起修改一个元素时，所有行的的相应元素都会被修改的现象。要避免这个问题，需要确保每一行都是**独立的列表对象**。以下是正确的初始化方式：

```python
grid = [[0] * n for _ in range(n)]
```

- `[0] * n` 创建一个独立的列表。
- `for _ in range(n)` 会为每一行创建独立的列表对象。

### 题解

初始化 $i=j=0$，按题意要求上下左右移动即可，注意题目保证不会出界。

最后返回 $*i⋅n+j*$。

不需要初始化二维数组！！！！

```python
class Solution(object):
    def finalPositionOfSnake(self, n, commands):
        """
        :type n: int
        :type commands: List[str]
        :rtype: int
        """
        op1 = op2 = 0
        for control in commands:
            if control == 'UP':
                op1 -= 1
            if control == 'DOWN':
                op1 += 1
            if control == 'RIGHT':
                op2 += 1
            if control == 'LEFT':
                op2 -= 1
        return op1*n+op2
```

## **3233. 统计不是特殊数字的数字数量**

### 问题描述

给你两个 **正整数** `l` 和 `r`。对于任何数字 `x`，`x` 的所有正因数（除了 `x` 本身）被称为 `x` 的 **真因数**。

如果一个数字恰好仅有两个 **真因数**，则称该数字为 **特殊数字**。例如：

- 数字 4 是 **特殊数字**，因为它的真因数为 1 和 2。
- 数字 6 不是 **特殊数字**，因为它的真因数为 1、2 和 3。

返回区间 `[l, r]` 内 **不是 特殊数字** 的数字数量。

### 示例

- 示例1

  **输入：** l = 5, r = 7

  **输出：** 3

  **解释：**

  区间 `[5, 7]` 内不存在特殊数字。

- 示例2

  **输入：** l = 4, r = 16

  **输出：** 11

  **解释：**

  区间 `[4, 16]` 内的特殊数字为 4 和 9。

### 解题思路/题解

特殊数字首先是一个平方数，并且除去自身和 1 之后的另一个因子一定是一个质数。这是因为：

因子一般是成双成对的，若一个数字有奇数个因子，那么该数一定是平方数。 该数除去自身和 1 仅有一个因子，因此该因子一定是质数。 因此，我们可以在$[1,\sqrt{r}]$ 的范围内遍历所有质数（使用质数筛，具体方法可以参考题解 204. 计数质数），然后将它们的平方从 $[l,r]$ 的范围中去除即可。

由于 r 的范围不超过 $10^9$  因此质数的遍历范围不超过 31622，而使用很简单的埃氏筛（复杂度为 O(nlognlogn)，其中 n 为质数遍历范围）就可以轻松通过本题。

作者：力扣官方题解 链接：https://leetcode.cn/problems/find-the-count-of-numbers-which-are-not-special/solutions/2984412/tong-ji-bu-shi-te-shu-shu-zi-de-shu-zi-s-kq6j/

```
class Solution(object):
    def nonSpecialCount(self, l, r):
        """
        :type l: int
        :type r: int
        :rtype: int
        """
        n = int(math.sqrt(r))
        v = [0] * (n + 1)
        res = r - l + 1
        for i in range(2, n + 1):
            if v[i] == 0:
                if l <= i * i <= r:
                    res -= 1
                for j in range(i * 2, n + 1, i):
                    v[j] = 1
        return res
```

## **743. 网络延迟时间**

### 问题描述

有 `n` 个网络节点，标记为 `1` 到 `n`。

给你一个列表 `times`，表示信号经过 **有向** 边的传递时间。 `times[i] = (ui, vi, wi)`，其中 `ui` 是源节点，`vi` 是目标节点， `wi` 是一个信号从源节点传递到目标节点的时间。

现在，从某个节点 `K` 发出一个信号。需要多久才能使所有节点都收到信号？如果不能使所有节点收到信号，返回 `-1` 。

### 示例

- 示例1

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/7d22e3b4-771f-45ab-8394-de2caf3b8481/image.png)

  输入：times = [[2,1,1],[2,3,1],[3,4,1]], n = 4, k = 2 输出：2

- 示例2

  输入：times = [[1,2,1]], n = 2, k = 1 输出：1

- 示例3

  输入：times = [[1,2,1]], n = 2, k = 2 输出：-1

### 解题思路

看到题目第一个想法就是遍历图，然后计算某一个节点到遍历所有节点的时间，看最后遍历完图的最长时间！问题就转化从k节点出发，遍历带权有向图的最长时间。这就很容易想到了Dijkstra算法。

**Dijkstra算法介绍**

最短路径解决问题中包含两种算法，一种是Dijkstra算法和Floyd算法。二者解决的方式不一样，解决的结果也不一样。Dijkstra算法能够解决单源点到其他所有顶点的最短路径，但是Floyd算法只能够解决单源点到目标源点的最短路径。

Dijkstra算法的核心思想是维护一个顶点集合，该集合包含从源点出发已知最短路径的顶点。算法开始时，这个集合只包含源点本身，其距离为0，其他所有顶点的距离都设为无穷大。然后，算法不断选择距离源点最近的未被访问的顶点，更新其相邻顶点的距离，直到所有顶点都被访问过。算法步骤如下：

1. **初始化**：将源点到自身的距离设为0，到其他所有顶点的距离设为无穷大。创建一个未访问顶点集合，包含图中所有顶点。
2. **选择顶点**：从未访问顶点集合中选择一个距离源点最近的顶点，设为当前顶点。
3. **更新距离**：对于当前顶点的每一个邻接顶点，计算通过当前顶点到达该邻接顶点的距离。如果这个距离小于之前记录的距离，就更新这个邻接顶点的距离。
4. **标记访问**：将当前顶点标记为已访问，从未访问顶点集合中移除。
5. **重复**：重复步骤2到4，直到所有顶点都被访问过或者未访问顶点集合为空。

------

回到本题，看看灵神的解析[743. 网络延迟时间 - 力扣（LeetCode）](https://leetcode.cn/problems/network-delay-time/)。

定义一个二维数组，$g[i][j]$表示节点$i$到节点$j$这条边的边权。如果没有$i$到$j$的边，则$g[i][j]= \infty$。

定义另外一个二维数组，$dis[i]$ 表示起点$k$到节点$i$的最短路径长度，一开始 $dis[k]=0$，其余$dis[i]=\infty$表示尚未计算出。我们目标是计算出最终的dis数组！

- 首先更新起点 $k$ 到其邻居  $y$  的最短路，即更新 $dis[y]$ 为 $g[k][y]$。
- 然后取除了起点 $k$ 以外的 $dis[i]$ 的最小值，假设最小值对应的节点是 3。此时可以断言：$dis[3]$ 已经是 k 到 3 的最短路长度，不可能有其它 k 到 3 的路径更短！反证法：假设存在更短的路径，那我们一定会从 k 出发经过一个点 u，它的 $dis[u]$ 比 $dis[3]$ 还要小，然后再经过一些边到达 3，得到更小的 $dis[3]$。但 $dis[3]$ 已经是最小的了，并且图中没有负数边权，所以 u 是不存在的，矛盾。故原命题成立，此时我们得到了 $dis[3]$  的最终值。
- 用节点 3 到其邻居 y 的边权 $g[3][y]$ 更新 $dis[y]$：如果 $dis[3]+g[3][y]<dis[y]$，那么更新 $dis[y] 为 dis[3]+g[3][y]$，否则不更新。
- 然后取除了节点 k,3 以外的 $dis[i]$ 的最小值，重复上述过程。
- 由数学归纳法可知，这一做法可以得到每个点的最短路。当所有点的最短路都已确定时，算法结束。

```python
class Solution(object):
    def networkDelayTime(self, times, n, k):
        """
        :type times: List[List[int]]
        :type n: int
        :type k: int
        :rtype: int
        """
        g = [[float('inf')] * n for _ in range(n)]
        for x, y, time in times:
            g[x - 1][y - 1] = time

        dist = [float('inf')] * n
        dist[k - 1] = 0
        used = [False] * n
        for _ in range(n):
            x = -1
            for y, u in enumerate(used):
                if not u and (x == -1 or dist[y] < dist[x]):
                    x = y
            used[x] = True
            for y, time in enumerate(g[x]):
                dist[y] = min(dist[y], dist[x] + time)

        ans = max(dist)
        return ans if ans < float('inf') else -1
```

## **3208. 交替组 II**

### 问题描述

给你一个整数数组 `colors` 和一个整数 `k` ，`colors`表示一个由红色和蓝色瓷砖组成的环，第 `i` 块瓷砖的颜色为 `colors[i]` ：

- `colors[i] == 0` 表示第 `i` 块瓷砖的颜色是 **红色** 。
- `colors[i] == 1` 表示第 `i` 块瓷砖的颜色是 **蓝色** 。

环中连续 `k` 块瓷砖的颜色如果是 **交替** 颜色（也就是说除了第一块和最后一块瓷砖以外，中间瓷砖的颜色与它 **左边** 和 **右边** 的颜色都不同），那么它被称为一个 **交替** 组。

请你返回 **交替** 组的数目。

**注意** ，由于 `colors` 表示一个 **环** ，**第一块** 瓷砖和 **最后一块** 瓷砖是相邻的。

### 示例

- 示例1

  **输入：**colors = [0,1,0,1,0], k = 3

  **输出：**3

  **解释：**

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/d692dc17-6862-408e-83e8-25723fd8f368/image.png)

  交替组包括：

  ![img](https://assets.leetcode.com/uploads/2024/05/28/screenshot-2024-05-28-182844.png)

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/d9a49e0e-fa85-4edd-a9bf-2a961d479144/image.png)

  ![img](https://assets.leetcode.com/uploads/2024/05/28/screenshot-2024-05-28-183057.png)

- 示例2

  **输入：**colors = [0,1,0,0,1,0,1], k = 6

  **输出：**2

  **解释：**

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/e7d52341-121d-49cc-b9f2-9118158c3fbb/image.png)

  交替组包括：

  ![img](https://assets.leetcode.com/uploads/2024/06/19/screenshot-2024-05-28-184128.png)

  ![img](https://assets.leetcode.com/uploads/2024/06/19/screenshot-2024-05-28-184240.png)

- 示例3

  **输入：**colors = [1,1,0,1], k = 4

  **输出：**0

  **解释：**

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/5adb9a63-25db-4bce-819c-f4468ee22f66/54b8ab09-c9de-400d-be65-38b2a0f6b227/image.png)

### 解题思路

遍历数组 colors，用一个整数 cnt 代表遍历到当前元素时，已经有的连续交替瓷砖的数量。如果当前元素与前一个元素不同，则将 cnt 加 1，否则将其置为 1。 如果当前 cnt 大于等于 k，则将结果加 1。注意到瓷砖是环形的，因此，在遍历到第一个数时，我们就需要知道当前的 cnt。为了得到初始的 cnt 值，我们需要将遍历的起点往前推 k−2 步，这样在遍历到数组的第一个元素时，我们就可以知道当前是否有 k 块连续的交替瓷砖。最后返回结果。

```python
class Solution(object):
    def numberOfAlternatingGroups(self, colors, k):
        """
        :type colors: List[int]
        :type k: int
        :rtype: int
        """
        n = len(colors)
        res, cnt = 0, 1
        for i in range(-k + 2, n, 1):
            if colors[i] != colors[i - 1]:
                cnt += 1
            else:
                cnt = 1
            if cnt >= k:
                res += 1
        return res
```

## **2712. 使所有字符相等的最小成本**

### 问题描述

给你一个下标从 **0** 开始、长度为 `n` 的二进制字符串 `s` ，你可以对其执行两种操作：

- 选中一个下标 `i` 并且反转从下标 `0` 到下标 `i`（包括下标 `0` 和下标 `i` ）的所有字符，成本为 `i + 1` 。
- 选中一个下标 `i` 并且反转从下标 `i` 到下标 `n - 1`（包括下标 `i` 和下标 `n - 1` ）的所有字符，成本为 `n - i` 。

返回使字符串内所有字符 **相等** 需要的 **最小成本** 。

**反转** 字符意味着：如果原来的值是 '0' ，则反转后值变为 '1' ，反之亦然。

### 示例

**示例 1：**

> 输入：s = "0011" 输出：2 解释：执行第二种操作，选中下标i = 2 ，可以得到s = "0000" ，成本为 2 。可以证明 2 是使所有字符相等的最小成本。

**示例 2：**

> 输入：s = "010101" 输出：9 解释：执行第一种操作，选中下标 i = 2 ，可以得到 s = "101101" ，成本为 3 。 执行第一种操作，选中下标 i = 1 ，可以得到 s = "011101" ，成本为 2 。 执行第一种操作，选中下标 i = 0 ，可以得到 s = "111101" ，成本为 1 。 执行第二种操作，选中下标 i = 4 ，可以得到 s = "111110" ，成本为 2 。 执行第二种操作，选中下标 i = 5 ，可以得到 s = "111111" ，成本为 1 。 使所有字符相等的总成本等于 9 。可以证明 9 是使所有字符相等的最小成本。

### 题解

如果 $s[i−1] \neq s[i]$，那么必须反转，不然这两个字符无法相等。

- 要么反转前缀 s[0] 到 s[i−1]，成本为 i。
- 要么反转后缀 s[i] 到 s[n−1]，成本为 n−i。

反转后 s[i−1]=s[i]。

注意到，反转操作只会改变 s[i−1] 与 s[i] 是否相等，不会改变其他相邻字符是否相等（相等的都反转还是相等，不相等的都反转还是不相等），所以每对相邻字符其实是互相独立的，我们只需要分别计算这些不相等的相邻字符的最小成本，即 min(i,n−i)，累加即为答案。

作者：灵茶山艾府 链接：https://leetcode.cn/problems/minimum-cost-to-make-all-characters-equal/solutions/2286922/yi-ci-bian-li-jian-ji-xie-fa-pythonjavac-aut0/

```python
class Solution:
    def minimumCost(self, s: str) -> int:
        n = len(s)
        ans = 0
        for i in range(1, n):
            if s[i - 1] != s[i]:
                ans += min(i, n - i)
        return ans
```

## **1493. 删掉一个元素以后全为 1 的最长子数组**

### 问题描述

给你一个二进制数组 `nums` ，你需要从中删掉一个元素。

请你在删掉元素的结果数组中，返回最长的且只包含 1 的非空子数组的长度。

如果不存在这样的子数组，请返回 0 。

### 示例

**示例1**

> 输入：nums = [1,1,0,1] 输出：3 解释：删掉位置 2 的数后，[1,1,1] 包含 3 个 1 。

**示例2**

> 输入：nums = [0,1,1,1,0,1,1,0,1] 输出：5 解释：删掉位置 4 的数字后，[0,1,1,1,1,1,0,1] 的最长全 1 子数组为 [1,1,1,1,1] 。

**示例3**

> 输入：nums = [1,1,1] 输出：2 解释：你必须要删除一个元素。

### 解题思路

这道题是一道不定长滑窗的题，第一个想法大概的思路应该就是双层循环？然后第一层循环代表的是删除每一个位置的情况，第二层循环再来统计最后的答案。仔细想想感觉不对，如果`nums` 里面包含0，那么肯定是删除0的位置，如果删除1的话只会让长度更小，不符合题意。

不定长滑窗应该是当窗口内的元素满足条件时（全为1），右边界就往右继续移动，直到不能满足为止，接着左边界移动到当前右边界的位置，右边界继续往右边移动。直到整个`nums`遍历结束。

但是这个思路是不是有问题呢？DeepSeek-R1回答如下：

1. “删除0的位置更优”的误区： 如果数组中存在0，删除一个0可能确实能拼接两段连续的1，但并非所有情况下最优解都来自删除0。例如：

   1. 数组 [1,1,0,1,1,0,1,1]：删除中间的0可以合并左右两段1，得到更长的子数组。
   2. 全1数组 [1,1,1]：必须删除一个1，此时最长子数组长度为2。此时没有0可删，但必须删一个元素。
   3. 多个0的情况：若数组中有多个0，可能需要选择删除哪个0能最大化结果（如示例2）。

   因此，不能简单认为“只删除0的位置”，而需要综合考虑所有可能的删除位置（包括0和1）。

2. 滑动窗口的条件设定问题：

   你提到的滑动窗口策略是“窗口全为1时右移右边界，否则左边界移到右边界”。这种策略无法处理允许删除一个元素的情况。正确的条件应为：窗口内最多允许一个0（即允许删除一个元素），而非窗口全为1。

具体思路如下:

我们可以使用滑动窗口技术来解决这个问题。具体思路如下：

1. **滑动窗口维护最多一个0**：我们维护一个窗口，允许其中包含最多一个0。这样可以确保窗口内的元素在删除一个0后形成全1子数组。
2. **窗口扩展与收缩**：遍历数组，扩展窗口的右边界。当窗口内包含超过一个0时，收缩左边界直到窗口内只含有一个0。
3. **计算最大长度**：在遍历过程中，记录窗口的最大长度。最终结果为最大窗口长度减一，因为我们必须删除一个元素。

这种方法的时间复杂度为O(n)，空间复杂度为O(1)，非常高效。

```python
class Solution(object):
    def longestSubarray(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        left = 0
        zeros = 0
        max_len = 0
        n = len(nums)
        
        for right in range(n):
            if nums[right] == 0:
                zeros += 1
            
            # 当窗口内0的数量超过1，需要收缩左边界
            while zeros > 1:
                if nums[left] == 0:
                    zeros -= 1
                left += 1
            
            # 更新最大窗口长度
            current_len = right - left + 1
            if current_len > max_len:
                max_len = current_len
        
        # 处理全为1的情况，必须删除一个元素，所以结果是max_len -1
        # 但若max_len为0，说明全为0，返回0
        if max_len != 0 :
            return max_len - 1
        else:
            return 0
      
```

## 1208. 尽可能使字符串相等

### 问题描述

给你两个长度相同的字符串，`s` 和 `t`。

将 `s` 中的第 `i` 个字符变到 `t` 中的第 `i` 个字符需要 `|s[i] - t[i]|` 的开销（开销可能为 0），也就是两个字符的 ASCII 码值的差的绝对值。

用于变更字符串的最大预算是 `maxCost`。在转化字符串时，总开销应当小于等于该预算，这也意味着字符串的转化可能是不完全的。

如果你可以将 `s` 的子字符串转化为它在 `t` 中对应的子字符串，则返回可以转化的最大长度。

如果 `s` 中没有子字符串可以转化成 `t` 中对应的子字符串，则返回 `0`。

### 示例

**示例1**

> 输入：s = "abcd", t = "bcdf", maxCost = 3 输出：3 解释：s中的"abc" 可以变为 "bcd"。开销为 3，所以最大长度为 3。

**示例2**

> 输入：s = "abcd", t = "cdef", maxCost = 3 输出：1 解释：s 中的任一字符要想变成 t 中对应的字符，其开销都是 2。因此，最大长度为 1。

**示例3**

> 输入：s = "abcd", t = "acde", maxCost = 0 输出：1 解释：a -> a, cost = 0，字符串未发生变化，所以最大长度为 1。

### 题解

为了解决这个问题，我们需要找到一个方法来确定在给定预算内，可以将字符串 `s` 的子字符串转换为字符串 `t` 对应的子字符串的最大长度。我们将使用滑动窗口技术来高效地解决这个问题。

**方法思路**

1. **计算转换开销**：首先，对于每个字符位置 `i`，计算从 `s[i]` 转换到 `t[i]` 的开销，即两个字符的 ASCII 码值的差的绝对值，存储在一个数组 `costs` 中。

2. 滑动窗口技术

   ：使用滑动窗口技术来找到最长的连续子数组，使得其开销总和不超过给定的预算 

   ```
   maxCost
   ```

   。具体步骤如下：

   - 维护两个指针 `left` 和 `right`，分别表示当前窗口的左右边界。
   - 扩展右指针 `right`，累加当前开销到当前总和 `current_sum`。
   - 如果当前总和超过预算 `maxCost`，则移动左指针 `left` 直到总和不超过预算。
   - 在每一步更新最大长度。

```python
class Solution(object):
    def equalSubstring(self, s, t, maxCost):
        """
        :type s: str
        :type t: str
        :type maxCost: int
        :rtype: int
        """
        n = len(s)
        costs = [abs(ord(s[i]) - ord(t[i])) for i in range(n)]
        max_len = 0
        current_sum = 0
        left = 0
        
        for right in range(n):
            current_sum += costs[right]
            
            # 如果当前总和超过预算，移动左指针直到总和不超过预算
            while current_sum > maxCost:
                current_sum -= costs[left]
                left += 1
            
            # 更新最大长度
            max_len = max(max_len, right - left + 1)
        
        return max_len
```

## **904. 水果成篮**

### 问题描述

你正在探访一家农场，农场从左到右种植了一排果树。这些树用一个整数数组 `fruits` 表示，其中 `fruits[i]` 是第 `i` 棵树上的水果 **种类** 。

你想要尽可能多地收集水果。然而，农场的主人设定了一些严格的规矩，你必须按照要求采摘水果：

- 你只有 **两个** 篮子，并且每个篮子只能装 **单一类型** 的水果。每个篮子能够装的水果总量没有限制。
- 你可以选择任意一棵树开始采摘，你必须从 **每棵** 树（包括开始采摘的树）上 **恰好摘一个水果** 。采摘的水果应当符合篮子中的水果类型。每采摘一次，你将会向右移动到下一棵树，并继续采摘。
- 一旦你走到某棵树前，但水果不符合篮子的水果类型，那么就必须停止采摘。

给你一个整数数组 `fruits` ，返回你可以收集的水果的 **最大** 数目。

### 示例

**示例1**

> 输入：fruits = [1,2,1] 输出：3 解释：可以采摘全部 3 棵树。

**示例2**

> 输入：fruits = [0,1,2,2] 输出：3 解释：可以采摘 [1,2,2] 这三棵树。 如果从第一棵树开始采摘，则只能采摘 [0,1] 这两棵树。

**示例3**

> 输入：fruits = [1,2,3,2,2] 输出：4 解释：可以采摘 [2,3,2,2] 这四棵树。 如果从第一棵树开始采摘，则只能采摘 [1,2] 这两棵树。

**示例4**

> 输入：fruits = [3,3,3,1,2,1,1,2,3,3,4] 输出：5 解释：可以采摘 [1,2,1,1,2] 这五棵树。

### 题解

用不定长的滑窗来解决，利用left和right来控制左右边界，当滑窗内满足条件时（滑窗内只有 两种元素），right就一直往右边移动，直到滑窗不满足条件时，left就移动到去除第一种水果的位置，保持右指针持续右移，仅调整左指针，从而高效维护窗口，确保算法在线性时间内完成。

## **416. 分割等和子集**

### 问题描述

给你一个 **只包含正整数** 的 **非空** 数组 `nums` 。请你判断是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。

### 示例

**示例1**

> 输入：nums = [1,5,11,5] 输出：true 解释：数组可以分割成 [1, 5, 5] 和 [11] 。

**示例2**

> 输入：nums = [1,2,3,5] 输出：false 解释：数组不能分割成两个元素和相等的子集。

### 题解

没有思路！赶紧看看灵神的题解（[416. 分割等和子集 - 力扣（LeetCode）](https://leetcode.cn/problems/partition-equal-subset-sum/solutions/2785266/0-1-bei-bao-cong-ji-yi-hua-sou-suo-dao-d-ev76/?envType=daily-question&envId=2025-04-07)），设`nums` 的元素和为$s$ ，两个子集相等意味着：

- 把`nums` 分成两个子集，每一个子集的元素和恰好是$\frac{s}{2}$。
- $s$必须是偶数

这就可以用恰好装满型的0-1背包问题解决。定义$dfs(i,j)$ 表示能否从$nums[0]$到$nums[i]$中选出一个和恰好等于j的子序列。

考虑$nums[i]$选或者不选：

- 选：问题变成能否从$nums[0]$到$nums[i-1]$中选出一个和恰好等于$j-nums[i]$的子序列，即$dfs(i-1,j-nums[i])$。
- 不选：问题变成能否从$nums[0]$到$nums[i-1]$中选出一个和恰好等于$j$的子序列，即$dfs(i-1,j)$

这两个只要有一个成立，$dfs(i,j)$就是`True` 。所以有

$$ dfs(i,j)=dfs(i-1,j-nums[i]) \vee dfs(i-1,j) $$

其中$\vee$用编程语言中的`||` 。代码实现时，可以只在$j\geq nums[i]$时才调用$dfs(i-1,j-nums[i])$，因为任何子序列的和都不会是负的。

递归边界：$*dfs(−1,0)=true, dfs(−1,>0)=false$。*如果所有数都考虑完了，且 $*j=0*$，表示找到了一个和为 $s/2$ 的子序列，返回`true`，否则返回 `false`。

**递归入口**：$*dfs(n−1,s/2)*$，即答案。

## **3396. 使数组元素互不相同所需的最少操作次数**

### 问题描述

给你一个整数数组 `nums`，你需要确保数组中的元素 **互不相同** 。为此，你可以执行以下操作任意次：

- 从数组的开头移除 3 个元素。如果数组中元素少于 3 个，则移除所有剩余元素。

**注意：**空数组也视作为数组元素互不相同。返回使数组元素互不相同所需的 **最少操作次数** 。

### 示例

**示例1**

> **输入：** nums = [1,2,3,4,2,3,3,5,7]
>
> **输出：** 2
>
> **解释：**
>
> - 第一次操作：移除前 3 个元素，数组变为 `[4, 2, 3, 3, 5, 7]`。
> - 第二次操作：再次移除前 3 个元素，数组变为 `[3, 5, 7]`，此时数组中的元素互不相同。
>
> 因此，答案是 2。

**示例2**

> **输入：** nums = [4,5,6,4,4] **输出：** 2 **解释：** 第一次操作：移除前 3 个元素，数组变为 `[4, 4]`。 第二次操作：移除所有剩余元素，数组变为空。 因此，答案是 2。

**示例3**

> **输入：** nums = [6,7,8,9]
>
> **输出：** 0
>
> **解释：**
>
> 数组中的元素已经互不相同，因此不需要进行任何操作，答案是 0。

### 题解

就是模拟，没有啥说的，但是时间和空间太拉跨了，还可以优化！看看灵神的解析！

每次操作，移除的都是 nums 的开头的元素，或者说移除的是 nums 的前缀。所以移除后，剩余的元素一定是 nums 的后缀。所以本质上，我们要找的是 nums 的一个后缀，其中没有重复元素。为了最小化操作次数，这个后缀越长越好。

所以问题相当于：

- nums 的最长无重复元素后缀。

```python
class Solution(object):
    def minimumOperations(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        seen = set()
        for i in range(len(nums) - 1, -1, -1):
            x = nums[i]
            if x in seen:
                return i // 3 + 1
            seen.add(x)
        return 0
```

## **1534. 统计好三元组**

### 问题描述

给你一个整数数组 `arr` ，以及 `a`、`b` 、`c` 三个整数。请你统计其中好三元组的数量。

如果三元组 `(arr[i], arr[j], arr[k])` 满足下列全部条件，则认为它是一个 **好三元组** 。

- `0 <= i < j < k < arr.length`
- `|arr[i] - arr[j]| <= a`
- `|arr[j] - arr[k]| <= b`
- `|arr[i] - arr[k]| <= c`

其中 `|x|` 表示 `x` 的绝对值。

返回 **好三元组的数量** 。

### 示例

**示例1**

> 输入：arr = [3,0,1,1,9,7], a = 7, b = 2, c = 3 输出：4 解释：一共有 4 个好三元组：[(3,0,1), (3,0,1), (3,1,1), (0,1,1)] 。

**示例2**

> 输入：arr = [3,0,1,1,9,7], a = 7, b = 2, c = 3 输出：4 解释：一共有 4 个好三元组：[(3,0,1), (3,0,1), (3,1,1), (0,1,1)] 。

### 题解

直接模拟：

```python
class Solution(object):
    def countGoodTriplets(self, arr, a, b, c):
        """
        :type arr: List[int]
        :type a: int
        :type b: int
        :type c: int
        :rtype: int
        """
        count = 0
        for i in range(len(arr)-2):
            for j in range(i+1,len(arr)-1):
                for k in range(j+1,len(arr)):
                    if abs(arr[i] - arr[j]) <= a and abs(arr[j] - arr[k]) <= b and abs(arr[i] - arr[k]) <= c:
                        count+=1
        return count
```

## **2179. 统计数组中好三元组数目**

### 问题描述

给你两个下标从 **0** 开始且长度为 `n` 的整数数组 `nums1` 和 `nums2` ，两者都是 `[0, 1, ..., n - 1]` 的 **排列** 。

**好三元组** 指的是 `3` 个 **互不相同** 的值，且它们在数组 `nums1` 和 `nums2` 中出现顺序保持一致。换句话说，如果我们将 `$pos1_v$` 记为值 `v` 在 `nums1` 中出现的位置，`$pos2_v$` 为值 `v` 在 `nums2` 中的位置，那么一个好三元组定义为 `0 <= x, y, z <= n - 1` ，且 `pos1x < pos1y < pos1z` 和 `pos2x < pos2y < pos2z` 都成立的 `(x, y, z)` 。

请你返回好三元组的 **总数目** 。

### 示例

**示例1**

> 输入：nums1 = [2,0,1,3], nums2 = [0,1,2,3] 输出：1 解释： 总共有 4 个三元组 (x,y,z) 满足 pos1x < pos1y < pos1z，分别是 (2,0,1) ，(2,0,3) ，(2,1,3) 和 (0,1,3) 。 这些三元组中，只有 (0,1,3) 满足 pos2x < pos2y < pos2z 。所以只有 1 个好三元组。

**示例2**

> 输入：nums1 = [4,0,1,3,2], nums2 = [4,1,0,2,3] 输出：4 解释：总共有 4 个好三元组 (4,0,3) ，(4,0,2) ，(4,1,3) 和 (4,1,2) 。

### 题解

为了解决这个问题，我们需要找到两个排列中满足顺序一致的三元组。具体来说，我们需要找到三个元素，使得它们在两个数组中的位置都是递增的。我们可以通过预处理每个元素的位置，并使用树状数组来高效地计算满足条件的数目。

### **方法思路**

1. **预处理位置信息**：首先，我们预处理每个元素在两个数组中的位置，存储在**`pos1`**和**`pos2`**数组中。
2. **排序元素**：按照元素在第一个数组中的位置进行排序，以便后续处理。
3. **计算左侧符合条件的元素数目**：使用树状数组统计在第一个数组中位置较小的元素中，第二个数组位置也较小的数目。
4. **计算右侧符合条件的元素数目**：类似地，统计在第一个数组中位置较大的元素中，第二个数组位置也较大的数目。
5. **计算总数目**：对于每个元素，将左侧和右侧的数目相乘并累加，得到最终结果。

```python
class BIT:
    def __init__(self, size):
        self.size = size
        self.tree = [0] * (self.size + 2)  # 1-based indexing
    
    def update(self, idx):
        i = idx + 1  # convert 0-based to 1-based
        while i <= self.size:
            self.tree[i] += 1
            i += i & -i
    
    def query(self, idx):
        res = 0
        i = idx + 1  # convert 0-based to 1-based
        while i > 0:
            res += self.tree[i]
            i -= i & -i
        return res

class Solution:
    def goodTriplets(self, nums1: List[int], nums2: List[int]) -> int:
        n = len(nums1)
        pos1 = [0] * n
        pos2 = [0] * n
        for i in range(n):
            pos1[nums1[i]] = i
        for i in range(n):
            pos2[nums2[i]] = i
        
        # Calculate left counts
        sorted_by_p1 = sorted(range(n), key=lambda x: pos1[x])
        left = [0] * n
        bit_left = BIT(n)
        for v in sorted_by_p1:
            p2 = pos2[v]
            left_count = bit_left.query(p2 - 1)
            left[v] = left_count
            bit_left.update(p2)
        
        # Calculate right counts
        sorted_by_p1_desc = sorted(range(n), key=lambda x: -pos1[x])
        right = [0] * n
        bit_right = BIT(n)
        count = 0
        for v in sorted_by_p1_desc:
            p2 = pos2[v]
            sum_le = bit_right.query(p2)
            right_count = count - sum_le
            right[v] = right_count
            bit_right.update(p2)
            count += 1
        
        total = 0
        for v in range(n):
            total += left[v] * right[v]
        
        return total
```

## **1399. 统计最大组的数目**

### 问题描述

给你一个整数 `n` 。请你先求出从 `1` 到 `n` 的每个整数 10 进制表示下的数位和（每一位上的数字相加），然后把数位和相等的数字放到同一个组中。

请你统计每个组中的数字数目，并返回数字数目并列最多的组有多少个。

### 示例

**示例1**

> 输入：n = 13 输出：4 解释：总共有 9 个组，将 1 到 13 按数位求和后这些组分别是： [1,10]，[2,11]，[3,12]，[4,13]，[5]，[6]，[7]，[8]，[9]。总共有 4 个组拥有的数字并列最多。

**示例2**

> 输入：n = 2 输出：2 解释：总共有 2 个大小为 1 的组 [1]，[2]。

**示例3**

> 输入：n = 15 输出：6

**示例4**

> 输入：n = 24 输出：5

### 题解

暴力枚举。

```
class Solution:
    def countLargestGroup(self, n: int) -> int:
        count = defaultdict(int)
        for i in range(1, n+1):
            s =  sum(map(int, str(i)))
            count[s] +=1
            max_count = max(count.values())
        result = sum(1 for v in count.values() if v == max_count)
        return result
```

## **2799. 统计完全子数组的数目**

### 问题描述

给你一个由 **正** 整数组成的数组 `nums` 。

如果数组中的某个子数组满足下述条件，则称之为 **完全子数组** ：

- 子数组中 **不同** 元素的数目等于整个数组不同元素的数目。

返回数组中 **完全子数组** 的数目。

**子数组** 是数组中的一个连续非空序列。

### 示例

**示例1**

> 输入：nums = [1,3,1,2,2] 输出：4 解释：完全子数组有：[1,3,1,2]、[1,3,1,2,2]、[3,1,2] 和 [3,1,2,2] 。

**示例2**

> 输入：nums = [5,5,5,5] 输出：10 解释：数组仅由整数 5 组成，所以任意子数组都满足完全子数组的条件。子数组的总数为 10 。

### 题解

第一感觉这是一个滑窗的题。看看灵神的题解，果然是！

子数组越长，越能满足题目要求，子数组越短，越不能满足。所以枚举子数组右端点`right`。同时用一个哈希表`cnt`维护子数组内每个元素出现的次数。

如果`nums[right]`加入哈希表以后，发现哈希表的大小等于`k`，说明子数组满足要求（`nums`中不同元素个数为`k`）。移动子数组左端点`left`，把`nums[left]`的出现次数减1。如果`nums[left]`的出现次数变0，则从`cnt`中去掉，表示子数组内少一种元素。

内层循环结束以后，`[left,right]`这个子数组是不满足题目要求的，但是在退出循环之前的最后一轮循环，`[left-1,right]`是满足题目要求的（哈希表大小等于k）。由于子数组越长越能满足题目要求，所以除了`[left-1,right]`，还有`[left-2,right]`,`[left-3,right]`…,`[0,right]`都是满足要求的。也就是说，当右端点固定在`right`时，左端点在`0`,`1`,`2`,…,`left-1`的所有子数组都是满足要求的，这一共有`left`个。

```python
class Solution:
    def countCompleteSubarrays(self, nums: List[int]) -> int:
        k = len(set(nums))
        cnt = defaultdict(int)
        ans = left = 0
        for x in nums:
            cnt[x]+=1
            while(len(cnt)==k):
                out = nums[left]
                cnt[out] -= 1
                if cnt[out] == 0:
                    del cnt[out]
                left+=1
            ans += left
        return ans
```

# 参考

https://github.com/changgyhub/leetcode_101/