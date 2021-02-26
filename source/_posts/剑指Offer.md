---
title: 	剑指Offer(年轻人你渴望力量吗)
date: 2021-02-26 00:08:28
tags:
categories:
 - 算法
img: https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/02/11/lcof.png
---

### [剑指 Offer 03\. 数组中重复的数字](https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)

Difficulty: **简单**


找出数组中重复的数字。

在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

**示例 1：**

```
输入：
[2, 3, 1, 0, 2, 5, 3]
输出：2 或 3 
```

**限制：**

`2 <= n <= 100000`


#### Solution

Language: ****

```
​
```

### [剑指 Offer 04\. 二维数组中的查找](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)

Difficulty: **中等**


在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个高效的函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

**示例:**

现有矩阵 matrix 如下：

```
[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
```

给定 target = `5`，返回 `true`。

给定 target = `20`，返回 `false`。

**限制：**

`0 <= n <= 1000`

`0 <= m <= 1000`

**注意：**本题与主站 240 题相同：


####  菜鸡第一印象-暴力法
O(nm)
```c++
​class Solution {
public:
    bool findNumberIn2DArray(vector<vector<int>>& matrix, int target)
    {
        if (matrix.empty() || matrix[0].empty())
        {
            return false;
        }

        for (const auto& vec : matrix)
        {
            if (vec[0] > target)
            {
                return false;
            }
            for (const auto& num : vec)
            {
                if (num == target)
                {
                    return true;
                }
                if (num > target)
                {
                    break;
                }
            }
        }
        return false;
    }
};
```

#### 大神点拨-站在右上角看,这个矩阵其实就像是一个BST

```c++
class Solution {
public:
    bool findNumberIn2DArray(vector<vector<int>>& matrix, int target)
    {
        if (matrix.empty() || matrix[0].empty())
        {
            return false;
        }
        
        int row_max = matrix.size();

        int column = matrix[0].size() - 1;
        int row = 0;

        while (row < row_max && column >= 0)
        {
            if (matrix[row][column] == target)
            {
                return true;
            }
            else if (matrix[row][column] > target)
            {
                column--;
            }
            else
            {
                row++;
            }
        }
        return false;
    }
};
```

#### 总结

初看题目, 还是没有详细审题漏掉了`每一列都按照从上到下递增的顺序排序`. 然后按照暴力法做了出来, 看了眼评论第一上面那句话让我翻回去看了下绝了.


### [剑指 Offer 05\. 替换空格](https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/)

Difficulty: **简单**


请实现一个函数，把字符串 `s` 中的每个空格替换成"%20"。

**示例 1：**

```
输入：s = "We are happy."
输出："We%20are%20happy."
```

**限制：**

`0 <= s 的长度 <= 10000`


#### Solution - 额外字符串reserve版本

```c++
​class Solution {
public:
    string replaceSpace(string s)
    {
        string result;
        result.reserve(s.size() * 3);

        for (char c : s)
        {
            if (c == ' ')
            {
                result.append("%20");
            }
            else
            {
                result += c;
            }
        }

        return result;
    }
};
```

#### Solution - 精打细算 resize原地

```c++
class Solution {
public:
    string replaceSpace(string s)
    {
        int num = 0;
        for (int i = 0; i < s.size(); ++i)
        {
            if (s[i] == ' ')
            {
                num++;
            }
        }

        s.resize(num * 2 + s.size());

        int sub = num * 2 + s.size();
        for (int i = s.size() - 1; i >= 0; --i)
        {
            if (s[i] != ' ')
            {
                sub--;
                s[sub] = s[i];
            }
            else
            {
                sub -= 3;
                s[sub] = '%';
                s[sub + 1] = '2';
                s[sub + 2] = '0';
            }
        }
        return s;
    }
};
```

### [剑指 Offer 06\. 从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

Difficulty: **简单**


输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

**示例 1：**

```
输入：head = [1,3,2]
输出：[2,3,1]
```

**限制：**

`0 <= 链表长度 <= 10000`


#### Solution - 两次遍历
```c++
class Solution {
public:
    vector<int> reversePrint(ListNode* head)
    {
        if (!head)
        {
            return vector<int>();
        }

        int length = 0;
        ListNode* temp = head;
        while (temp)
        {
            length++;
            temp = temp->next;
        }

        vector<int> result;
        result.resize(length);
        for (int i = length - 1; i >= 0; --i)
        {
            result[i] = head->val;
            head = head->next;
        }
        return result;
    }
};
```

#### Solution - 递归
```c++
class Solution {
public:
    vector<int> reversePrint(ListNode* head)
    {
        vector<int> result;
        reversePrint(head, &result);
        return result;
    }

    void reversePrint(ListNode* head, vector<int>* result)
    {
        if (!head)
        {
            return;
        }

        reversePrint(head->next, result);
        result->push_back(head->val);
    } 
};
```

#### Solution - 栈
```c++
class Solution {
public:
    vector<int> reversePrint(ListNode* head)
    {
        stack<ListNode*> sta;

        ListNode* temp = head;
        while (temp)
        {
            sta.push(temp);
            temp = temp->next;
        }

        vector<int> result(sta.size());
        int sub = 0;
        while (!sta.empty())
        {
            result[sub++] = sta.top()->val;
            sta.pop();
        }
        return result;
    }
};
```

### [剑指 Offer 07\. 重建二叉树](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/)

Difficulty: **中等**


输入某二叉树的前序遍历和中序遍历的结果，请重建该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。

例如，给出

```
前序遍历 preorder = [3,9,20,15,7]
中序遍历 inorder = [9,3,15,20,7]
```

返回如下的二叉树：

```
    3
   / \
  9  20
    /  \
   15   7
```

**限制：**

`0 <= 节点个数 <= 5000`

**注意**：本题与主站 105 题重复：


#### Solution

```c++
​class Solution {
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder)
    {
        return buildTree(preorder, inorder, 0, 0, inorder.size());
    }

    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder, int pre_l, int in_l, int in_r)
    {
        if (in_l == in_r)
        {
            return nullptr;
        }
        
        TreeNode* node = new TreeNode(preorder[pre_l]);

        for (int i = in_l; i < in_r; ++i)
        {
            if (inorder[i] == preorder[pre_l])
            {
                node->left = buildTree(preorder, inorder, pre_l + 1, in_l, i);
                node->right = buildTree(preorder, inorder, pre_l + 1 + i - in_l, i + 1, in_r);
            }
        }

        return node;
    }
};
```

### [剑指 Offer 09\. 用两个栈实现队列](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)

Difficulty: **简单**


用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 `appendTail` 和 `deleteHead` ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，`deleteHead` 操作返回 -1 )

**示例 1：**

```
输入：
["CQueue","appendTail","deleteHead","deleteHead"]
[[],[3],[],[]]
输出：[null,null,3,-1]
```

**示例 2：**

```
输入：
["CQueue","deleteHead","appendTail","appendTail","deleteHead","deleteHead"]
[[],[],[5],[2],[],[]]
输出：[null,-1,null,null,5,2]
```

**提示：**

*   `1 <= values <= 10000`
*   `最多会对 appendTail、deleteHead 进行 10000 次调用`

#### Solution - 版本1 拷贝多

```c++
class CQueue {
public:
    CQueue() {

    }
    
    void appendTail(int value)
    {
        while (!core_stack.empty())
        {
            int temp = core_stack.top();
            core_stack.pop();
            temp_stack.push(temp);
        }

        temp_stack.push(value);
    }
    
    int deleteHead()
    {
        while (!temp_stack.empty())
        {
            int temp = temp_stack.top();
            temp_stack.pop();
            core_stack.push(temp);
        }
        
        int result = -1;
        if (!core_stack.empty())
        {
            result = core_stack.top();
            core_stack.pop();
        }
        return result;
    }

    stack<int> temp_stack;
    stack<int> core_stack;
};
```

#### Solution - 版本2 减少了拷贝

```c++
​class CQueue {
public:
    CQueue() {

    }
    
    void appendTail(int value)
    {
        temp_stack.push(value);
    }
    
    int deleteHead()
    {
        int result = -1;

        if (!core_stack.empty())
        {
            result = core_stack.top();
            core_stack.pop();
        }
        else
        {
            while (!temp_stack.empty())
            {
                int temp = temp_stack.top();
                temp_stack.pop();
                core_stack.push(temp);
            }

            if (!core_stack.empty())
            {
                result = core_stack.top();
                core_stack.pop();
            }
        }
        
        return result;
    }

    stack<int> temp_stack;
    stack<int> core_stack;
};
```
### [剑指 Offer 10- I. 斐波那契数列](https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof/)

Difficulty: **简单**


写一个函数，输入 `n` ，求斐波那契（Fibonacci）数列的第 `n` 项（即 `F(N)`）。斐波那契数列的定义如下：

```
F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
```

斐波那契数列由 0 和 1 开始，之后的斐波那契数就是由之前的两数相加而得出。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

**示例 1：**

```
输入：n = 2
输出：1
```

**示例 2：**

```
输入：n = 5
输出：5
```

**提示：**

*   `0 <= n <= 100`


#### Solution

```c++
class Solution {
public:
    int fib(int n)
    {
        int arr[max(n + 1, 2)];
        arr[0] = 0;
        arr[1] = 1;

        for (int i = 2; i <= n; ++i)
        {
            arr[i] = (arr[i - 1] + arr[i - 2]) % 1000000007;
        }

        return arr[n];
    }
};

```

### [剑指 Offer 10- II. 青蛙跳台阶问题](https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)

Difficulty: **简单**


一只青蛙一次可以跳上1级台阶，也可以跳上2级台阶。求该青蛙跳上一个 `n` 级的台阶总共有多少种跳法。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

**示例 1：**

```
输入：n = 2
输出：2
```

**示例 2：**

```
输入：n = 7
输出：21
```

**示例 3：**

```
输入：n = 0
输出：1
```

**提示：**

*   `0 <= n <= 100`

注意：本题与主站 70 题相同：


#### Solution

```c++
​class Solution {
public:
    int numWays(int n)
    {
        int dp[max(n + 1, 2)];
        dp[0] = 1;
        dp[1] = 1;

        for (int i = 2; i <= n; ++i)
        {
            dp[i] = (dp[i - 1] + dp[i - 2]) % 1000000007;
        }

        return dp[n];
    }
};
```

### [剑指 Offer 11\. 旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

Difficulty: **简单**


把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组 `[3,4,5,1,2]` 为 `[1,2,3,4,5]` 的一个旋转，该数组的最小值为1。  

**示例 1：**

```
输入：[3,4,5,1,2]
输出：1
```

**示例 2：**

```
输入：[2,2,2,0,1]
输出：0
```

注意：本题与主站 154 题相同：


#### Solution - 暴力法

```c++
class Solution {
public:
    int minArray(vector<int>& numbers)
    {
        int result = numbers[0];
        for (int i = 0; i < numbers.size() - 1; ++i)
        {
            if (numbers[i] > numbers[i + 1])
            {
                result = numbers[i + 1];
                break;
            }
        }

        return result;
    }
};​
```

#### Solution - 二分 (一个排序好的数组, 虽然打乱一次 但仍应该首先考虑二分)

```c++
class Solution {
public:
    int minArray(vector<int>& numbers)
    {
        int low = 0;
        int high = numbers.size() - 1;

        while (low < high)
        {
            int mid = (low + high) / 2;

            if (numbers[mid] > numbers[high])
            {
                low = mid + 1;
            }
            else if (numbers[mid] < numbers[high])
            {
                high = mid;
            }
            else
            {
                high--;
            }
        }
        return numbers[low];
    }
};
```

### [剑指 Offer 12\. 矩阵中的路径](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)

Difficulty: **中等**


请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一格开始，每一步可以在矩阵中向左、右、上、下移动一格。如果一条路径经过了矩阵的某一格，那么该路径不能再次进入该格子。例如，在下面的3×4的矩阵中包含一条字符串“bfce”的路径（路径中的字母用加粗标出）。

[["a","**b**","c","e"],  
["s","**f**","**c**","s"],  
["a","d","**e**","e"]]

但矩阵中不包含字符串“abfb”的路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入这个格子。

**示例 1：**

```
输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"
输出：true
```

**示例 2：**

```
输入：board = [["a","b"],["c","d"]], word = "abcd"
输出：false
```

**提示：**

*   `1 <= board.length <= 200`
*   `1 <= board[i].length <= 200`

注意：本题与主站 79 题相同：


#### Solution - dfs

Language: ****

```c++
​class Solution {
public:
    bool exist(vector<vector<char>>& board, string word)
    {
        for (int i = 0; i < board.size(); ++i)
        {
            for (int j = 0; j < board[0].size(); ++j)
            {
                if (word[0] == board[i][j])
                {
                    if (exist(board, word, i, j, 0))
                    {
                        return true;
                    }
                }
            }
        }
        return false;
    }

    bool exist(vector<vector<char>>& board, const string& word, int x, int y, int sub)
    {
        if (sub >= word.size())
        {
            return true;
        }

        if (x < 0 || y < 0 || x >= board.size() || y >= board[0].size())
        {
            return false;
        }

        if (board[x][y] == ' ')
        {
            return false;
        }

        if (board[x][y] != word[sub])
        {
            return false;
        }

        char back = board[x][y];
        board[x][y] = ' ';

        if (exist(board, word, x + 1, y, sub + 1) || 
                exist(board, word, x, y + 1, sub + 1) ||
                exist(board, word, x - 1, y, sub + 1) ||
                exist(board, word, x, y - 1, sub + 1))
        {
            return true;
        }

        board[x][y] = back;

        return false;
    }
};
```

### [剑指 Offer 13\. 机器人的运动范围](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)

Difficulty: **中等**


地上有一个m行n列的方格，从坐标 `[0,0]` 到坐标 `[m-1,n-1]` 。一个机器人从坐标 `[0, 0]` 的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），也不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入方格 [35, 37] ，因为3+5+3+7=18。但它不能进入方格 [35, 38]，因为3+5+3+8=19。请问该机器人能够到达多少个格子？

**示例 1：**

```
输入：m = 2, n = 3, k = 1
输出：3
```

**示例 2：**

```
输入：m = 3, n = 1, k = 0
输出：1
```

**提示：**

*   `1 <= n,m <= 100`
*   `0 <= k <= 20`


#### Solution - dfs

```c++
​class Solution {
public:
    bool Check(int x, int y, int k)
    {
        int temp = 0;

        while (x != 0)
        {
            temp += x % 10;
            x /= 10;
        }
        while (y != 0)
        {
            temp += y % 10;
            y /= 10;
        }
        return temp <= k;
    }

    int movingCount(int m, int n, int k)
    {
        vector<vector<int>> vec(m, vector<int>(n, 0));
        return movingCount(vec, 0, 0, k);
    }

    constexpr static int SS = 4;
    int xx[SS] = {1, -1, 0, 0};
    int yy[SS] = {0, 0, 1, -1};

    int movingCount(vector<vector<int>>& vec, int x, int y, int k)
    {
        if (!Check(x, y, k))
        {
            return 0;
        }

        if (x < 0 || y < 0 || x >= vec.size() || y >= vec[0].size())
        {
            return 0;
        }

        if (vec[x][y] == 1)
        {
            return 0;
        }
        vec[x][y] = 1;
        int result = 1;

        for (int i = 0; i < SS; ++i)
        {
            result += movingCount(vec, x + xx[i], y + yy[i], k);
        }

        return result;
    }
};
```

### [剑指 Offer 14- I. 剪绳子](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/)

Difficulty: **中等**


给你一根长度为 `n` 的绳子，请把绳子剪成整数长度的 `m` 段（m、n都是整数，n>1并且m>1），每段绳子的长度记为 `k[0],k[1]...k[m-1]` 。请问 `k[0]*k[1]*...*k[m-1]` 可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。

**示例 1：**

```
输入: 2
输出: 1
解释: 2 = 1 + 1, 1 × 1 = 1
```

**示例 2:**

```
输入: 10
输出: 36
解释: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36
```

**提示：**

*   `2 <= n <= 58`

注意：本题与主站 343 题相同：


#### Solution - dp

```c++
​class Solution
{
public:
    int cuttingRope(int n)
    {
        int* dp = new int[n + 1]{};
        // 初始状态
        dp[2] = 1;
        // 状态转移方程
        for(int i = 3; i < n + 1; i++)
        {
            for(int j = 2; j < i; j++)
            {
                dp[i] = max(dp[i], max(j * dp[i-j], j * (i - j)));
            }
        }
        // 返回值
        return dp[n];
    }
};
```
### [剑指 Offer 15\. 二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

Difficulty: **简单**


请实现一个函数，输入一个整数（以二进制串形式），输出该数二进制表示中 1 的个数。例如，把 9 表示成二进制是 1001，有 2 位是 1。因此，如果输入 9，则该函数输出 2。

**示例 1：**

```
输入：00000000000000000000000000001011
输出：3
解释：输入的二进制串 00000000000000000000000000001011 中，共有三位为 '1'。
```

**示例 2：**

```
输入：00000000000000000000000010000000
输出：1
解释：输入的二进制串 00000000000000000000000010000000 中，共有一位为 '1'。
```

**示例 3：**

```
输入：11111111111111111111111111111101
输出：31
解释：输入的二进制串 11111111111111111111111111111101 中，共有 31 位为 '1'。
```

**提示：**

*   输入必须是长度为 `32` 的 **二进制串** 。

注意：本题与主站 191 题相同：


#### Solution - 右移运算符 循环M次 M为二进制位数和

```c++
​class Solution {
public:
    int hammingWeight(uint32_t n)
    {
        int result = 0;

        while (n != 0)
        {
            result += n & 1;
            n = n >> 1;
        }
        return result;
    }
};
```

#### Solution - 更加高效的 循环M次 M为二进制1的位数

```c++
class Solution {
public:
    int hammingWeight(uint32_t n)
    {
        int result = 0;

        while (n != 0)
        {
            result++;
            n &= n - 1;
        }
        return result;
    }
};
```

### [剑指 Offer 16\. 数值的整数次方](https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/)

Difficulty: **中等**


实现函数double Power(double base, int exponent)，求base的exponent次方。不得使用库函数，同时不需要考虑大数问题。

**示例 1:**

```
输入: 2.00000, 10
输出: 1024.00000
```

**示例 2:**

```
输入: 2.10000, 3
输出: 9.26100
```

**示例 3:**

```
输入: 2.00000, -2
输出: 0.25000
解释: 2-2 = 1/22 = 1/4 = 0.25
```

**说明:**

*   -100.0 < _x_ < 100.0
*   _n_ 是 32 位有符号整数，其数值范围是 [−2<sup>31</sup>, 2<sup>31 </sup>− 1] 。

注意：本题与主站 50 题相同：


#### Solution - 快速幂

```c++
​class Solution {
public:
    double myPow(double x, int n)
    {
        if (n == 0)
        {
            return 1;
        }

        bool flag = true;
        if (n > 0)
        {
            flag = false;
            n = -n;
        }

        double result = x;
        vector<double> ex;
        while (n < -1)
        {
            if (n % 2 == -1)
            {
                ex.push_back(result);
            }
            result *= result;
            n = n / 2; 
        }

        while (!ex.empty())
        {
            result *= ex.back();
            ex.pop_back();
        }

        if (flag)
        {
            result = 1 / result;
        }

        return result;
    }
};
```

#### Solution - 快速幂 减少空间 去除vector

```c++
class Solution {
public:
    double myPow(double x, int n)
    {
        if (n == 0)
        {
            return 1;
        }

        bool flag = true;
        if (n > 0)
        {
            flag = false;
            n = -n;
        }

        double result = x;
        double ex = 1.0;
        while (n < -1)
        {
            if (n % 2 == -1)
            {
                ex *= result;
            }
            result *= result;
            n = n / 2; 
        }

        result *= ex;

        if (flag)
        {
            result = 1 / result;
        }

        return result;
    }
};
```

### [剑指 Offer 18\. 删除链表的节点](https://leetcode-cn.com/problems/shan-chu-lian-biao-de-jie-dian-lcof/)

Difficulty: **简单**


给定单向链表的头指针和一个要删除的节点的值，定义一个函数删除该节点。

返回删除后的链表的头节点。

**注意：**此题对比原题有改动

**示例 1:**

```
输入: head = [4,5,1,9], val = 5
输出: [4,1,9]
解释: 给定你链表中值为 5 的第二个节点，那么在调用了你的函数之后，该链表应变为 4 -> 1 -> 9.
```

**示例 2:**

```
输入: head = [4,5,1,9], val = 1
输出: [4,5,9]
解释: 给定你链表中值为 1 的第三个节点，那么在调用了你的函数之后，该链表应变为 4 -> 5 -> 9.
```

**说明：**

*   题目保证链表中节点的值互不相同
*   若使用 C 或 C++ 语言，你不需要 `free` 或 `delete` 被删除的节点


#### Solution 爆栈Warning

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* deleteNode(ListNode* head, int val)
    {
        if (!head)
        {
            return nullptr;
        }

        if (head->val == val)
        {
            return head->next;
        }
        
        head->next = deleteNode(head->next, val);
        return head;
    }
};
```


#### Solution - 保存last

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* deleteNode(ListNode* head, int val)
    {
        if (!head)
        {
            return nullptr;
        }

        if (head->val == val)
        {
            return head->next;
        }

        ListNode* last = head;
        ListNode* temp = head->next;

        while (temp)
        {
            if (temp->val == val)
            {
                last->next = temp->next;
            }
            last = temp;
            temp = temp->next;
        }

        return head;
    }
};
```

### [剑指 Offer 20\. 表示数值的字符串](https://leetcode-cn.com/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/)

Difficulty: **中等**


请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100"、"5e2"、"-123"、"3.1416"、"-1E-16"、"0123"都表示数值，但"12e"、"1a3.14"、"1.2.3"、"+-5"及"12e+5.4"都不是。


#### Solution if else - 我愿称其为最强题目 尝试了N遍才过

```c++
​class Solution {
public:
    bool isNumber(string s)
    {
        int begin = 0;
        int end = s.size();
        bool num = false;

        // 去除开头结尾空格
        while (begin < end && s[begin] == ' ')
        {
            begin++;
        }
        while (end > begin && s[end - 1] == ' ')
        {
            end--;
        }
        int sub = begin;
    
        // + - 打头直接去掉
        if (s[sub] == '+' || s[sub] == '-')
        {
            sub++;
        }

        // 判断+-之后是否有小数点
        bool has_point = false;
        if (s[sub] == '.')
        {
            has_point = true;
            sub++;
        }
        
        // 正常数字判断
        while (sub < end)
        {
            if (s[sub] == '.')
            {
                if (has_point)
                {
                    return false; // 小数点只能出现一次
                }
                has_point = true;
            }
            else if (s[sub] >= '0' && s[sub] <= '9')
            {
                num = true; // 是一个数字
            }
            else
            {
                break;
            }
            sub++;
        }

        // 科学计数法判断 E或e 不作为第一个或者最后一个字符
        if (sub > begin && sub < end - 1 && (s[sub] == 'e' || s[sub] == 'E'))
        {
            sub++;

            // 去掉E e之后紧接着的 +- 必须 +-只有还有数字
            if ((s[sub] == '+' || s[sub] == '-') && sub < end - 1)
            {
                sub++;
            }

            while (sub < end && s[sub] >= '0' && s[sub] <= '9')
            {
                sub++;
            }
        }
        return num && sub == end; // 含数字且已经遍历完毕
    }
};
```

#### Solution -  自动状态机

```c++

```

### [剑指 Offer 21\. 调整数组顺序使奇数位于偶数前面](https://leetcode-cn.com/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)

Difficulty: **简单**


输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有奇数位于数组的前半部分，所有偶数位于数组的后半部分。

**示例：**

```
输入：nums = [1,2,3,4]
输出：[1,3,2,4] 
注：[3,1,2,4] 也是正确的答案之一。
```

**提示：**

1.  `0 <= nums.length <= 50000`
2.  `1 <= nums[i] <= 10000`


#### Solution - 代码不简洁版本 双指针

```c++
​class Solution {
public:
    vector<int> exchange(vector<int>& nums)
    {
        if (nums.empty())
        {
            return nums;
        }

        int begin = 0;
        int end = nums.size() - 1;

        while (begin < end)
        {
            while (nums[end] % 2 == 0)
            {
                end--;
                if (begin > end)
                {
                    return nums;
                }
            }

            if (nums[begin] % 2 == 0)
            {
                swap(nums[begin], nums[end--]);
            }
            else
            {
                begin++;
            }
        }
        return nums;
    }
};
```

#### Solution - 使用continue简化边界判断 双指针

```c++
class Solution {
public:
    vector<int> exchange(vector<int>& nums)
    {
        int left = 0; 
        int right = nums.size() - 1;

        while (left < right)
        {
            if ((nums[left] & 1) != 0)
            {
                left ++;
                continue;
            }
            if ((nums[right] & 1) != 1)
            {
                right --;
                continue;
            }
            swap(nums[left++], nums[right--]);
        }
        return nums;
    }
};
```


#### Solution - 快慢指针

```c++
class Solution {
public:
    vector<int> exchange(vector<int>& nums)
    {
        int slow = 0;
        int fast = 0;

        while (fast < nums.size())
        {
            if ((nums[fast] & 1) == 0)
            {
                fast++;
                continue;
            }
            swap(nums[slow++], nums[fast++]);
        }
        return nums;
    }
};
```
慢指针负责指向偶数, 当快指针寻找到奇数时 进行替换 然后两者都向前移动

不过这个代码存在原地tp的问题 极端情况下如果全是奇数则会替换size次

### [剑指 Offer 22\. 链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)

Difficulty: **简单**


输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。

例如，一个链表有 `6` 个节点，从头节点开始，它们的值依次是 `1、2、3、4、5、6`。这个链表的倒数第 `3` 个节点是值为 `4` 的节点。

**示例：**

```
给定一个链表: 1->2->3->4->5, 和 k = 2.

返回链表 4->5.
```


#### Solution - 依旧是解决链表问题的通用方法 快慢指针

```c++
​class Solution {
public:
    ListNode* getKthFromEnd(ListNode* head, int k)
    {
        if (!head)
        {
            return nullptr;
        }

        ListNode* fast = head;
        for (int i = 0; i < k; ++i)
        {
            fast = fast->next;
        }

        ListNode* slow = head;
        while (fast)
        {
            slow = slow->next;
            fast = fast->next;
        }
        return slow;
    }
};
```

判断链表环依然采用快慢指针, 快指针一次走两步 注意判断两次fast不为nullptr 当fast==low的时候说明有环

判断环长度, 第一次相遇后 到 第二次相遇 慢指针走的次数即为长度. 快指针在相遇后正好比慢指针多走一圈


1->2->3->4

1->2->3->4->5

取中间节点 快指针走两步 慢指针走一步 当快指针无法走两步的时候 慢指针所指为中间

### [剑指 Offer 24\. 反转链表](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/)

Difficulty: **简单**


定义一个函数，输入一个链表的头节点，反转该链表并输出反转后链表的头节点。

**示例:**

```
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

**限制：**

`0 <= 节点个数 <= 5000`

**注意**：本题与主站 206 题相同：


#### Solution1 - 代码过长的递归


```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* reverseList(ListNode* head)
    {
        if (!head)
        {
            return nullptr;
        }
        
        ListNode* new_head = nullptr;
        ListNode* front = reverse(head, &new_head);
        front->next = nullptr;

        return new_head;
    }

    ListNode* reverse(ListNode* node, ListNode** head)
    {
        if (!node)
        {
            return nullptr;
        }

        ListNode* front = reverse(node->next, head);
        if (front)
        {
            front->next = node;
        }
        else
        {
            *head = node;
        }
        return node;
    }
};
```

#### Solution2 - 代码简洁的递归

```c++
class Solution {
public:
    ListNode* reverseList(ListNode* head)
    {
        if (!head || !head->next)
        {
            return head;
        }

        ListNode* ret = reverseList(head->next);

        head->next->next = head;
        head->next = nullptr;
        return ret;
    }
};
```
首先S2开头的`!head->next`的判断减少了一次了递归 同时也简化了S1中的`front`的判断 不会有空指针.

然后S1使用的返回值 返回的下一个节点. 然而实际上下一个节点已经可以通过本节点的`next`访问 通过`next->next`来倒转指向

这样将返回值省了出来 返回原链表的尾结点 作为新的头结点

#### Solution - 双指针 又双叒叕

```c++
class Solution {
public:
    ListNode* reverseList(ListNode* head)
    {
        if (!head)
        {
            return nullptr;
        }

        ListNode* node2 = head->next;
        ListNode* node1 = head;

        node1->next = nullptr;

        ListNode* back = nullptr;
        while (node2)
        {
            back = node2->next;

            node2->next = node1;
            node1 = node2;

            node2 = back;
        }
        return node1;
    }
};
```

### [剑指 Offer 25\. 合并两个排序的链表](https://leetcode-cn.com/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/)

Difficulty: **简单**


输入两个递增排序的链表，合并这两个链表并使新链表中的节点仍然是递增排序的。

**示例1：**

```
输入：1->2->4, 1->3->4
输出：1->1->2->3->4->4
```

**限制：**

`0 <= 链表长度 <= 1000`

注意：本题与主站 21 题相同：


#### Solution - 未使用虚拟头结点 代码较长且重复

```c++
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2)
    {
        if (!l1)
        {
            return l2;
        }

        if (!l2)
        {
            return l1;
        }

        ListNode* ret = nullptr;
        if (l1->val < l2->val)
        {
            ret = l1;
            l1 = l1->next;
        }
        else
        {
            ret = l2;
            l2 = l2->next;
        }

        ListNode* node = ret;
        while (l1 || l2)
        {
            if (!l1)
            {
                node->next = l2;
                break;
            }
            if (!l2)
            {
                node->next = l1;
                break;
            }

            if (l1->val < l2->val)
            {
                node->next = l1;
                l1 = l1->next;
            }
            else
            {
                node->next = l2;
                l2 = l2->next;
            }
            node = node->next;
        }
        return ret;
    }
};
```

#### Solution - 使用虚拟头结点 代码大幅简洁 去除了重复片段

```c++
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2)
    {
        ListNode* dumy = new ListNode(0);

        ListNode* node = dumy;
        while (l1 || l2)
        {
            if (!l1)
            {
                node->next = l2;
                break;
            }
            if (!l2)
            {
                node->next = l1;
                break;
            }

            if (l1->val < l2->val)
            {
                node->next = l1;
                l1 = l1->next;
            }
            else
            {
                node->next = l2;
                l2 = l2->next;
            }
            node = node->next;
        }
        return dumy->next;
    }
};
```

使用了虚拟头结点 直接简化了重复片段 tql

### [剑指 Offer 26\. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)

Difficulty: **中等**


输入两棵二叉树A和B，判断B是不是A的子结构。(约定空树不是任意一个树的子结构)

B是A的子结构， 即 A中有出现和B相同的结构和节点值。

例如:  
给定的树 A:

`     3  
    / \  
   4   5  
  / \  
 1   2`  
给定的树 B：

`   4   
  /  
 1`  
返回 true，因为 B 与 A 的一个子树拥有相同的结构和节点值。

**示例 1：**

```
输入：A = [1,2,3], B = [3,1]
输出：false
```

**示例 2：**

```
输入：A = [3,4,5,1,2], B = [4,1]
输出：true
```

**限制：**

`0 <= 节点个数 <= 10000`


#### Solution - 先序遍历

```c++
​class Solution {
public:
    bool isSubStructure(TreeNode* A, TreeNode* B)
    {
        if (!A || !B)
        {
            return false;
        }

        if (A->val == B->val)
        {
            if (cmpTree(A, B))
            {
                return true;
            }
        }

        return isSubStructure(A->left, B) ||
        isSubStructure(A->right, B);
    }

    bool cmpTree(TreeNode* A, TreeNode* B)
    {
        if (!B)
        {
            return true;
        }
        if (!A)
        {
            return false;
        }

        if (A->val == B->val)
        {
            return cmpTree(A->left, B->left) && cmpTree(A->right, B->right);
        }
        return false;
    }
};
```

#### Solution - 简洁了一些 但我认为一个ifelse比一个函数调用更加高效

```c++
class Solution {
public:
    bool isSubStructure(TreeNode* A, TreeNode* B)
    {
        if (!A || !B)
        {
            return false;
        }

        return cmpTree(A, B) || isSubStructure(A->left, B) ||
        isSubStructure(A->right, B);
    }

    bool cmpTree(TreeNode* A, TreeNode* B)
    {
        if (!B)
        {
            return true;
        }
        if (!A)
        {
            return false;
        }

        return (A->val == B->val) && cmpTree(A->left, B->left) && cmpTree(A->right, B->right);
    }
};
```

### [剑指 Offer 27\. 二叉树的镜像](https://leetcode-cn.com/problems/er-cha-shu-de-jing-xiang-lcof/)

Difficulty: **简单**


请完成一个函数，输入一个二叉树，该函数输出它的镜像。

例如输入：

`     4  
   /   \  
  2     7  
 / \   / \  
1   3 6   9`  
镜像输出：

`     4  
   /   \  
  7     2  
 / \   / \  
9   6 3   1`

**示例 1：**

```
输入：root = [4,2,7,1,3,6,9]
输出：[4,7,2,9,6,3,1]
```

**限制：**

`0 <= 节点个数 <= 1000`

注意：本题与主站 226 题相同：


#### Solution - 递归

```c++
​class Solution {
public:
    TreeNode* mirrorTree(TreeNode* root)
    {
        if (!root)
        {
            return nullptr;
        }
        
        swap(root->left, root->right);

        mirrorTree(root->left);
        mirrorTree(root->right);

        return root;
    }
};
```

#### Solution - 迭代

```c++
class Solution {
public:
    TreeNode* mirrorTree(TreeNode* root)
    {
        if (!root)
        {
            return nullptr;
        }

        deque<TreeNode*> nodes;
        nodes.push_back(root);

        while (!nodes.empty())
        {
            TreeNode* node = nodes.front();
            nodes.pop_front();
            swap(node->left, node->right);

            if (node->left)
            {
                nodes.push_back(node->left);
            }
            if (node->right)
            {
                nodes.push_back(node->right);
            }
        }

        return root;
    }
};
```

### [剑指 Offer 28\. 对称的二叉树](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/)

Difficulty: **简单**


请实现一个函数，用来判断一棵二叉树是不是对称的。如果一棵二叉树和它的镜像一样，那么它是对称的。

例如，二叉树 [1,2,2,3,4,4,3] 是对称的。

`    1  
   / \  
  2   2  
 / \ / \  
3  4 4  3`  
但是下面这个 [1,2,2,null,3,null,3] 则不是镜像对称的:

`    1  
   / \  
  2   2  
   \   \  
   3    3`

**示例 1：**

```
输入：root = [1,2,2,3,4,4,3]
输出：true
```

**示例 2：**

```
输入：root = [1,2,2,null,3,null,3]
输出：false
```

**限制：**

`0 <= 节点个数 <= 1000`

注意：本题与主站 101 题相同：


#### Solution - 递归

```c++
​class Solution {
public:
    bool isSymmetric(TreeNode* root)
    {
        if (!root)
        {
            return true;
        }

        return solve(root->left, root->right);
    }

    bool solve(TreeNode* lhs, TreeNode* rhs)
    {
        if (!lhs && !rhs)
        {
            return true;
        }
        if (!lhs || !rhs)
        {
            return false;
        }

        if (lhs->val == rhs->val)
        {
            return solve(lhs->left, rhs->right) && solve(lhs->right, rhs->left);
        }
        return false;
    }
};
```

#### Solution - 迭代

```c++
class Solution {
public:
    bool isSymmetric(TreeNode* root)
    {
        if (!root)
        {
            return true;
        }

        list<TreeNode*> nodes;
        nodes.push_back(root->left);
        nodes.push_back(root->right);
        
        while (!nodes.empty())
        {
            TreeNode* node1 = nodes.front();
            nodes.pop_front();
            TreeNode* node2 = nodes.front();
            nodes.pop_front();

            if (!node1 && !node2)
            {
                continue;
            }
            else if (node1 && node2)
            {
                if (node1->val != node2->val)
                {
                    return false;
                }
                nodes.push_back(node1->left);
                nodes.push_back(node2->right);
                nodes.push_back(node1->right);
                nodes.push_back(node2->left);
            }
            else
            {
                return false;
            }
        }
        return true;
    }
};
```

### [剑指 Offer 29\. 顺时针打印矩阵](https://leetcode-cn.com/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)

Difficulty: **简单**


输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字。

**示例 1：**

```
输入：matrix = [[1,2,3],[4,5,6],[7,8,9]]
输出：[1,2,3,6,9,8,7,4,5]
```

**示例 2：**

```
输入：matrix = [[1,2,3,4],[5,6,7,8],[9,10,11,12]]
输出：[1,2,3,4,8,12,11,10,9,5,6,7]
```

**限制：**

*   `0 <= matrix.length <= 100`
*   `0 <= matrix[i].length <= 100`

注意：本题与主站 54 题相同：


#### Solution - 额外MN空间

```c++
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix)
    {
        if (matrix.empty() || matrix[0].empty())
        {
            return vector<int>();
        }

        const int MX = matrix.size();
        const int MY = matrix[0].size();
        vector<vector<int>> flags(MX, vector<int>(MY, 0));

        int x = 0, y = 0;

        vector<int> result;
        result.reserve(MX * MY);

        int xx[] = {0, 1, 0, -1};
        int yy[] = {1, 0, -1, 0};

        int i = 0;

        while (x >= 0 && y >= 0 && x < MX && y < MY && flags[x][y] != 1)
        {
            for (;x >= 0 && y >= 0 && x < MX && y < MY && flags[x][y] != 1; 
                x += xx[i], y += yy[i])
            {
                flags[x][y] = 1;
                result.push_back(matrix[x][y]);
            }

            x -= xx[i];
            y -= yy[i];

            i = (i + 1) % 4;

            x += xx[i];
            y += yy[i];
        }
        return result;
    }
};
```

#### Solution - 边界控制 不使用额外空间

```c++
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix)
    {
        if (matrix.empty() || matrix[0].empty())
        {
            return vector<int>();
        }

        const int MX = matrix.size();
        const int MY = matrix[0].size();

        vector<int> result;
        result.reserve(MX * MY);

        int xx[] = {0, 1, 0, -1};
        int yy[] = {1, 0, -1, 0};

        // int nn[] = {1, 0, 0, 0};
        // int ee[] = {0, -1, 0, 0};
        // int ss[] = {0, 0, -1, 0};
        // int ww[] = {0, 0, 0, 1};

        int i = 0;
        int x = 0, y = 0;

        int e = MY, s = MX, w = 0, n = 0;

        while (x >= n && y >= w && x < s && y < e)
        {
            for (;x >= n && y >= w && x < s && y < e; 
                x += xx[i], y += yy[i])
            {
                result.push_back(matrix[x][y]);
            }

            x -= xx[i];
            y -= yy[i];
            

            // n += nn[i];
            // e += ee[i];
            // s += ss[i]; 更短的写法
            // w += ww[i];

            if (i == 0)
            {
                n++;
            }
            else if (i == 1)
            {
                e--;
            }
            else if (i == 2)
            {
                s--;
            }
            else
            {
                w++;
            }

            i = (i + 1) % 4;
            x += xx[i];
            y += yy[i];
        }
        return result;
    }
};
```

### [剑指 Offer 31\. 栈的压入、弹出序列](https://leetcode-cn.com/problems/zhan-de-ya-ru-dan-chu-xu-lie-lcof/)

Difficulty: **中等**


输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如，序列 {1,2,3,4,5} 是某栈的压栈序列，序列 {4,5,3,2,1} 是该压栈序列对应的一个弹出序列，但 {4,3,5,1,2} 就不可能是该压栈序列的弹出序列。

**示例 1：**

```
输入：pushed = [1,2,3,4,5], popped = [4,5,3,2,1]
输出：true
解释：我们可以按以下顺序执行：
push(1), push(2), push(3), push(4), pop() -> 4,
push(5), pop() -> 5, pop() -> 3, pop() -> 2, pop() -> 1
```

**示例 2：**

```
输入：pushed = [1,2,3,4,5], popped = [4,3,5,1,2]
输出：false
解释：1 不能在 2 之前弹出。
```

**提示：**

1.  `0 <= pushed.length == popped.length <= 1000`
2.  `0 <= pushed[i], popped[i] < 1000`
3.  `pushed` 是 `popped` 的排列。

注意：本题与主站 946 题相同：


#### Solution1 - X一样长的代码

```c++
class Solution {
public:
    bool validateStackSequences(vector<int>& pushed, vector<int>& popped)
    {
        stack<int> lstack;

        int i = 0, j = 0;

        while (i < pushed.size() && j < popped.size())
        {
            while (!lstack.empty())
            {
                if (lstack.top() == popped[j])
                {
                    lstack.pop();
                    j++;
                }
                else
                {
                    break;
                }
            }
            if (pushed[i] == popped[j])
            {
                i++;
                j++;
            }
            else
            {
                lstack.push(pushed[i]);
                i++;
            }
        }

        while (!lstack.empty())
        {
            if (lstack.top() == popped[j])
            {
                lstack.pop();
                j++;
            }
            else
            {
                return false;
            }
        }
        return true;
    }
};
```


#### Solution2 - 大佬精简的代码

```c++
class Solution {
public:
    bool validateStackSequences(vector<int>& pushed, vector<int>& popped)
    {
        stack<int> lstack;

        int j = 0;
        for (int num : pushed)
        {
            lstack.push(num);

            while (!lstack.empty() && lstack.top() == popped[j])
            {
                lstack.pop();
                j++;
            }
        }
        return j == popped.size();
    }
};
```

**精简思路**

源代码是栈不为空的时候 比较栈顶和`popped[i]`

如果相等则出栈

如果不相等 转去判断`pushed当前元素`和`popped[i]`

如果两者相等则后移

如果两者不相等则入栈当前元素.

*实际上这里就可以直接入栈元素然后再判断,如果相等了就出栈 如果不相等也恰好入栈了*

### [剑指 Offer 32 - I. 从上到下打印二叉树](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)

Difficulty: **中等**


从上到下打印出二叉树的每个节点，同一层的节点按照从左到右的顺序打印。

例如:  
给定二叉树: `[3,9,20,null,null,15,7]`,

```
    3
   / \
  9  20
    /  \
   15   7
```

返回：

```
[3,9,20,15,7]
```

**提示：**

1.  `节点总数 <= 1000`


#### Solution - deque

```c++
​class Solution {
public:
    vector<int> levelOrder(TreeNode* root)
    {
        if (!root)
        {
            return {};
        }
        deque<TreeNode*> nodes;

        nodes.push_back(root);

        vector<int> result;
        while (!nodes.empty())
        {
            TreeNode* node = nodes.front();
            nodes.pop_front();

            result.push_back(node->val);

            if (node->left)
            {
                nodes.push_back(node->left);
            }
            if (node->right)
            {
                nodes.push_back(node->right);
            }
        }
        return result;
    }
};
```

### [剑指 Offer 32 - II. 从上到下打印二叉树 II](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/)

Difficulty: **简单**


从上到下按层打印二叉树，同一层的节点按从左到右的顺序打印，每一层打印到一行。

例如:  
给定二叉树: `[3,9,20,null,null,15,7]`,

```
    3
   / \
  9  20
    /  \
   15   7
```

返回其层次遍历结果：

```
[
  [3],
  [9,20],
  [15,7]
]
```

**提示：**

1.  `节点总数 <= 1000`

注意：本题与主站 102 题相同：


#### Solution - 使用nullptr标记

```c++
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root)
    {
        if (!root)
        {
            return {};
        }

        deque<TreeNode*> nodes;

        nodes.push_back(root);
        nodes.push_back(nullptr);

        vector<vector<int>> result;
        result.push_back(vector<int>());

        while (!nodes.empty())
        {
            TreeNode* node = nodes.front();
            nodes.pop_front();

            if (node)
            {
                result.back().push_back(node->val);
                if (node->left)
                {
                    nodes.push_back(node->left);
                }
                if (node->right)
                {
                    nodes.push_back(node->right);
                }
            }
            else
            {
                if (!nodes.empty())
                {
                    result.push_back(vector<int>());
                    nodes.push_back(nullptr);
                }
            }
        }

        return result;
    }
};
```
首先想出的就是这个方法

#### - 使用for循环 提前保存变量解决size会改变的问题

```c++
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root)
    {
        if (!root)
        {
            return {};
        }

        deque<TreeNode*> nodes;

        nodes.push_back(root);

        vector<vector<int>> result;

        while (!nodes.empty())
        {
            result.push_back(vector<int>());

            int ss = nodes.size();
            for (int i = 0; i < ss; ++i)
            {
                TreeNode* node = nodes.front();
                nodes.pop_front();

                result.back().push_back(node->val);
                if (node->left)
                {
                    nodes.push_back(node->left);
                }
                if (node->right)
                {
                    nodes.push_back(node->right);
                }
            }
        }

        return result;
    }
};
```

写完S1就去看了看题解 没想到还有这种方法, 妙蛙种子在米奇妙妙屋吃妙脆角

### [剑指 Offer 32 - III. 从上到下打印二叉树 III](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/)

Difficulty: **中等**


请实现一个函数按照之字形顺序打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右到左的顺序打印，第三行再按照从左到右的顺序打印，其他行以此类推。

例如:  
给定二叉树: `[3,9,20,null,null,15,7]`,

```
    3
   / \
  9  20
    /  \
   15   7
```

返回其层次遍历结果：

```
[
  [3],
  [20,9],
  [15,7]
]
```

**提示：**

1.  `节点总数 <= 1000`


#### Solution - 设置标志位

```c++
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) 
    {
        if (!root)
        {
            return {};
        }

        deque<TreeNode*> nodes;
        nodes.push_back(root);

        bool right = true;

        vector<vector<int>> result;

        while (!nodes.empty())
        {
            result.emplace_back();

            int SS = nodes.size();

            if (right)
            {
                right = false;
                for (int i = 0; i < SS; ++i)
                {
                    TreeNode* node = nodes.front();
                    nodes.pop_front();
                    result.back().push_back(node->val);

                    if (node->left)
                    {
                        nodes.push_back(node->left);
                    }
                    if (node->right)
                    {
                        nodes.push_back(node->right);
                    }
                }
            }
            else
            {
                right = true;
                for (int i = SS - 1; i >= 0; --i)
                {
                    TreeNode* node = nodes.back();
                    nodes.pop_back();
                    result.back().push_back(node->val);
                    
                    if (node->right)
                    {
                        nodes.push_front(node->right);
                    }
                    if (node->left)
                    {
                        nodes.push_front(node->left);
                    }
                }
            }
        }
        return result;
    }
};​
```

#### Solution - 使用reverse

```c++
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) 
    {
        if (!root)
        {
            return {};
        }

        deque<TreeNode*> nodes;
        nodes.push_back(root);

        vector<vector<int>> result;
        bool even = false;

        while (!nodes.empty())
        {
            result.emplace_back();

            int SS = nodes.size();

            vector<int>& tmp = result.back();
            for (int i = 0; i < SS; ++i)
            {
                TreeNode* node = nodes.front();
                nodes.pop_front();
                tmp.push_back(node->val);

                if (node->left)
                {
                    nodes.push_back(node->left);
                }
                if (node->right)
                {
                    nodes.push_back(node->right);
                }
            }

            if (even)
            {
                std::reverse(tmp.begin(), tmp.end());
            }
            even = !even;
        }
        return result;
    }
};
```

### [剑指 Offer 33\. 二叉搜索树的后序遍历序列](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/)

Difficulty: **中等**


输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历结果。如果是则返回 `true`，否则返回 `false`。假设输入的数组的任意两个数字都互不相同。

参考以下这颗二叉搜索树：

```
     5
    / \
   2   6
  / \
 1   3
```

**示例 1：**

```
输入: [1,6,3,2,5]
输出: false
```

**示例 2：**

```
输入: [1,3,2,6,5]
输出: true
```

**提示：**

1.  `数组长度 <= 1000`


#### Solution - 递归

```c++
​class Solution {
public:
    bool verifyPostorder(vector<int>& postorder)
    {
        return verifyPostorder(postorder, 0, postorder.size());
    }

    bool verifyPostorder(vector<int>& postorder, int begin, int end)
    {
        if (end - begin <= 2)
        {
            return true;
        }

        int root_sub = end - 1;
        int r_begin = begin;
        for (int i = begin; i < root_sub; ++i)
        {
            if (postorder[i] < postorder[root_sub])
            {
                r_begin++;
            }
            else
            {
                break;
            }
        }

        for (int i = r_begin; i < root_sub; ++i)
        {
            if (postorder[i] < postorder[root_sub])
            {
                return false;
            }
        }
        return verifyPostorder(postorder, begin, r_begin) && 
            verifyPostorder(postorder, r_begin, root_sub);
    }
};
```

睡前来一道, 这是我第一次 一次通过的剑指里面的中等题目-_- 睡了睡了

#### Solution - 迭代 栈


### [剑指 Offer 34\. 二叉树中和为某一值的路径](https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/)

Difficulty: **中等**


输入一棵二叉树和一个整数，打印出二叉树中节点值的和为输入整数的所有路径。从树的根节点开始往下一直到叶节点所经过的节点形成一条路径。

**示例:**  
给定如下二叉树，以及目标和 `sum = 22`，

```
              5
             / \
            4   8
           /   / \
          11  13  4
         /  \    / \
        7    2  5   1
```

返回:

```
[
   [5,4,11,2],
   [5,8,4,5]
]
```

**提示：**

1.  `节点总数 <= 10000`

注意：本题与主站 113 题相同：


#### Solution - 回溯法

```c++
​class Solution {
public:

    vector<int> path;
    vector<vector<int>> result;

    vector<vector<int>> pathSum(TreeNode* root, int sum)
    {
        dfs(root, sum);

        return result; 
    }

    void dfs(TreeNode* root, int sum)
    {
        if (!root)
        {
            return;
        }

        path.push_back(root->val);
        if (sum == root->val && !root->left && !root->right)
        {
            result.push_back(path);
        }
        else
        {
            dfs(root->left, sum - root->val);
            dfs(root->right, sum - root->val);
        }
        path.pop_back();
    }
};
```



### [剑指 Offer 37\. 序列化二叉树](https://leetcode-cn.com/problems/xu-lie-hua-er-cha-shu-lcof/)

Difficulty: **困难**


请实现两个函数，分别用来序列化和反序列化二叉树。

**示例: **

```
你可以将以下二叉树：

    1
   / \
  2   3
     / \
    4   5

序列化为 "[1,2,3,null,null,4,5]"
```

注意：本题与主站 297 题相同：


#### Solution


```c++
class Codec {
public:

    // Encodes a tree to a single string.
    string serialize(TreeNode* root)
    {
        if (!root)
        {
            return "[null]";
        }

        deque<TreeNode*> nodes;
        nodes.push_back(root);

        string result;
        while (!nodes.empty())
        {
            TreeNode* node = nodes.front();
            nodes.pop_front();
            if (node)
            {
                result += "," + to_string(node->val);
                nodes.push_back(node->left);
                nodes.push_back(node->right);
            }
            else
            {
                result += ",null";
            }
        }

        result[0] = '[';
        result += ']';
        return result;
    }

    // Decodes your encoded data to tree.
    TreeNode* deserialize(string data)
    {
        deque<TreeNode*> nodes;
        ParseData(data, nodes);

        TreeNode* ret = nodes.front();
        nodes.pop_front();

        if (nodes.size() == 0)
        {
            return ret;
        }
        
        deque<TreeNode*> temp_nodes;
        temp_nodes.push_back(ret);

        while (!temp_nodes.empty())
        {
            const int SS = temp_nodes.size();

            if (nodes.size() < SS * 2)
            {
                break;
            }

            for (int i = 0; i < SS; ++i)
            {
                TreeNode* node = temp_nodes.front();
                temp_nodes.pop_front();

                node->left = nodes.front();
                nodes.pop_front();
                node->right = nodes.front();
                nodes.pop_front();

                if (node->left)
                {
                    temp_nodes.push_back(node->left);
                }
                if (node->right)
                {
                    temp_nodes.push_back(node->right);
                }
            }
        }

        return ret;
    }

    void ParseData(const string& data, deque<TreeNode*>& nodes)
    {
        int i = 1;

        int num = 0;
        bool empty = false;
        int flag = 1; // -

        while (i < data.size())
        {
            char c = data[i++];

            if (c >= '0' && c <= '9')
            {
                num = num * 10 + c - '0';
            }
            else if (c == ',' || c == ']')
            {
                if (empty)
                {
                    nodes.push_back(nullptr);
                    empty = false;
                }
                else
                {
                    nodes.push_back(new TreeNode(num * flag));
                    num = 0;
                    flag = 1;
                }
            }
            else if (c == 'n')
            {
                empty = true;
            }
            else if (c == '-')
            {
                flag = -1;
            }
        }
    }
};​
```