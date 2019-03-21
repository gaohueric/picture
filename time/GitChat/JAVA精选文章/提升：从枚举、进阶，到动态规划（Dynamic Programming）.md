### 内容提要

内容提要：

- 如果想看懂需要什么样的数学基础吗？这样的算法都可以用在哪些领域？
- 一般讲动态规划会被问到些什么问题？
- 哪些数据结构和算法是程序必须掌握的？学习算法和数据结构的方法和过程？
- 什么是动态规划DP？
- 爬到第 n 层的方法 f(n)=f(n−2)+f(n−1)，是怎么得出来的？
- 如何判断某个问题是否适合用DP求解？能否举出一个例子？
- 给定一个字符串，判断有效字符串时，迭代方程是怎么探索到的？
- DP 的中文书推荐和下载链接？

------

**问：如果想看懂需要什么样的数学基础吗？这样的算法都可以用在哪些领域？**

**答：**我们习惯在学习在某个知识的时候，先问问需要什么基础知识。其实动态规划技术，不需要特别的数学知识，因为它本来就是一个数学基础发展而来的技术思想。首先为了书写方便，我们将动态规划 Dynamic Programming，简写为 DP，并在以下书写中使用。DP 类似于数学归纳法，先找出状态初始时的解，然后找出 dp[i] 和 dp[i-1] 等的状态迭代关系。它类似于我们高中时代的数学归纳法，灵活地应用 DP 主要看大家分析问题的能力，善于发现状态演变的规律。

DP 广泛应用于各个领域，包括运筹学，比如常见的 VRP，TSP 问题；再比如，机器学习中训练的最重要技术：反向传播技术 (BP) 也可以理解为 DP 的应用。当然，还有各个其他领域也都能找到 DP 的应用。大家知道 DP 应用广泛就好，最重要的还是不断运用 DP 解决自己的项目和学习任务。

------

**问：一般讲动态规划会被问到些什么问题？**

**答：**在面试中经常会被问题 DP 问题，并且一旦问到 DP，你能准确回答出来，一般将会进入下一轮。因此我根据自己的一点经验，总结下面试中 DP 一般问什么。大家请看 Chat 文章的一道题，这个是我当年面试经历的一道题目：机器人行走路径。这道题做完后，面试官问了我这么1个问题：你是如何判断这个问题可以用 DP 求解的？ 说实话这个问题还是不太好回答的，因为面比较广。接下来，我会在 Chat 的中间环节回答此问题。面试官还会问：此题的状态方程你是如何分析出来的？ DP 常用的技巧或者牺牲掉了什么东西？

------

**问：哪些数据结构和算法是程序必须掌握的？学习算法和数据结构的方法和过程？**

**答：**对于专业写程序的小伙伴，平时工作忙于码代码，可能没有太多时间梳理提炼常用的数据结构和算法。其实，代码的本质不是 for、if 等，而是数据结构和算法。下面我谈下自己的浅显理解，计算机中最重要的数据结构：线性表，链表，二叉树，图结构。

需要掌握的算法可能还得以你的具体专业为导向。我列举几个最基础最通用最重要的几个算法：快速排序，插入排序，选择排序，堆排序，归并排序。这里面只列举排序算法，不是说其他的不重要，只能说排序算法体现的思想太重要了，以上几个算法涵盖了以下思想：分治法，贪心法，回溯法，深度搜索。掌握这些算法不是说要记忆它们，背过它们，而是要会灵活运用它们，在我们的工作学习中，工作中如果经常 for 循环嵌套，这时候你可能需要学习算法和数据结构了。

问题快速排序是怎么回事，我把它很快说出来，然后把其他的6、7种算法，详细回答了一遍，后来面试官就没问我别的了。所以面试算法工程师，可能第一关就是排序算法，面试经常问到。

关于学习数据结构和算法的方法，我想没有统一的方法，还得根据个人的实际情况，我说下自己的思路。一般来说算法的代码是比较抽象难懂的，别看只有几行代码，但是表达的内涵之丰富，思想之深远，需要仔细琢磨，结合里面的体现的数学公式。因此学习算法需要遇到一个，精确深刻掌握一个，包括理解算法伪代码，写出算法的实现源码，不是说遇到的每一个都这样，但是起码你得在差不多刚开始接触学习算法时，要这样做，并且这对是非常有效的，我曾经就是这样走过来的。

------

**问：什么是动态规划 DP？**

**答：**DP 首先它不是一个具体的算法，而是一个算法思想，算法框架，一项技术，它是一种通过寻找解空间的迭代方程求解问题的技术思想。在常用的算法思想：分治法，贪心法，回溯法，深度搜索中，DP 是比较难以运用的算法思想之一，它的难点是分辨出一个问题是否可以用 DP 求解；如果可以 DP 求解，怎么找出迭代方程。

------

**问：爬到第 n 层的方法 f(n)=f(n−2)+f(n−1)，是怎么得出来的？**

**答：**这个就是 DP 最本质的东西，如何构思出这种状态迭代方程。这个爬楼梯的例子可以说是入门 DP 的第一题了。迭代方程简单经典，大家想入门 DP 的，需要仔细体会这道题目。这个就是此数列。怎么构思出的这个迭代方程？

假设爬到顶层一共有 f(n) 种，则爬到第 n−1 层一共有 f(n−1) 种，爬到第 n−2 层一共有 f(n−2) 种，那么，f(n)、f(n−1)、f(n−2) 之间的关系可以这样想：爬到第 n 层，如果只经过 1 个步骤，一共有几种爬法？ 两种，如下：

- 从第 n−2层爬 2 步到第 n 层。
- 从第 n−1层爬 1 步到第 n 层。

所以我们就得到了上面的迭代方程。注意此处是必须经过1个步骤，有的小伙伴会不会这么想，我从第 n-2 层爬到第 n 层有2种可能？如果这么想，说明你快理解了，但是还不够，因为我们找迭代方程必须要经过一个迭代时步。

------

**问：如何判断某个问题是否适合用DP求解？能否举出一个例子？**

**答：**如果某个问题符合动态规划的主要条件，那么就可以尝试用动态规划。一般地，动态规划需要求解的问题具有 2 个核心特征：

1. optimal-substructure property：最优化的子结构属性。
2. overlapping subproblems：重叠的子问题集。

这的确有点难以理解，尤其是平时对算法不是很敏感的小伙伴。下面我举一个例子，来解释上面两个特征。

假如要求解某个最大连乘数组的最大值，发现子问题的最优解一定包括在原问题中，它是原问题的组成部分，并且，求以第 i + 1 个元素为索引的子数组一定要求出以第 i 个元素为索引的子数组的最大连乘。则这个问题可以用 DP 求解。这个例子出在我的公众号，大家可以看下上周日的推送文章，详细了解此题，里面带有更加详细的解释。

------

**问：给定一个字符串，判断有效字符串时，迭代方程是怎么探索到的？**

**答：**这是DP有相对难度的一个题目了，不过是非常有意思的问题，我在 Chat 文章里带有一个 gif 图片来阐述迭代方程的得来过程。在这里我做一点说明，也是最重要的解释。根据 DP 问题的特点，子问题的最优解一定被原问题的最优解包含，以此启发，我们定义一个最重要的变量：dp[i]，表示以索引 i 结尾的字符串的最长长度。就是这个 dp[i] 变量的引入，体现了 DP 思想精髓，子问题的最优解一定组成于原问题。再更详细的介绍，大家参考文章。

------

**问：DP 的中文书推荐和下载链接？**

**答：**比较基础入门的算法书籍，我先推荐给大家一本，经典书籍：《算法设计与分析基础》。下载链接： https://pan.baidu.com/s/1udUEP5LN6iIcilYCRbpwlg 密码：p1sp。



### 文章实录

- - - [1. 动态规划的概念](https://gitbook.cn/books/5b1eff3950a8df6692e025a4/index.html#undefined)
    - \2. 爬楼梯，初步领悟 DP
      - [2.1 分析](https://gitbook.cn/books/5b1eff3950a8df6692e025a4/index.html#undefined)
      - [2.2 递归版本1.0](https://gitbook.cn/books/5b1eff3950a8df6692e025a4/index.html#undefined)
      - 2.3 降低 O(2n)O(2n)
        - [2.3.1 版本 1.1](https://gitbook.cn/books/5b1eff3950a8df6692e025a4/index.html#undefined)
        - [2.3.2 版本1.2](https://gitbook.cn/books/5b1eff3950a8df6692e025a4/index.html#undefined)
    - \3. 机器人行走路径
      - [3.1 分析问题](https://gitbook.cn/books/5b1eff3950a8df6692e025a4/index.html#undefined)
      - [3.2 Python 代码实现](https://gitbook.cn/books/5b1eff3950a8df6692e025a4/index.html#undefined)
    - [4. DP 通用条件](https://gitbook.cn/books/5b1eff3950a8df6692e025a4/index.html#undefined)
    - \5. 深入 DP
      - [5.1 初步分析](https://gitbook.cn/books/5b1eff3950a8df6692e025a4/index.html#undefined)
      - [5.2 版本 1.0](https://gitbook.cn/books/5b1eff3950a8df6692e025a4/index.html#undefined)
      - [5.3 版本 2.0](https://gitbook.cn/books/5b1eff3950a8df6692e025a4/index.html#undefined)
    - [6. 总结 DP](https://gitbook.cn/books/5b1eff3950a8df6692e025a4/index.html#undefined)
    - [7. 展望 DP](https://gitbook.cn/books/5b1eff3950a8df6692e025a4/index.html#undefined)
    - [8. 参考文献](https://gitbook.cn/books/5b1eff3950a8df6692e025a4/index.html#undefined)

### 1. 动态规划的概念

动态规划的英文名称 dynamic programming，简称为 DP。《Introduction to algorithms》对动态规划的定义：

> A dynamic-programming algorithm solves each subproblem just once and then saves its answer in a table, thereby avoiding the work of recomputing the answer every time it solves each subproblem.

动态规划算法仅用一次解决每一个子问题，并将解保存在表中，因此避免了每次求解子问题时的重复计算。

### 2. 爬楼梯，初步领悟 DP

Climbing Stairs（爬楼梯）问题描述如下：每次只能爬 1 步 或 2 步，问有多少种爬到楼顶的方法，假设一共有 n 层。

#### 2.1 分析

假设爬到顶层一共有 f(n)f(n) 种，则爬到第 n−1n−1 层一共有 f(n−1)f(n−1) 种，爬到第 n−2n−2 层一共有 f(n−2)f(n−2) 种，那么，f(n)f(n)、f(n−1)f(n−1)、f(n−2)f(n−2) 之间有什么关系呢？

这样想：爬到第 n 层，如果只经过 1 个步骤，一共有几种爬法？ 两种，如下：

1. 从第 n−2n−2 层爬 2 步到第 n 层，又知道爬到第 n−2n−2 层共有 f(n−2)f(n−2) 种
2. 从第 n−1n−1 层爬 1 步到第 n 层，又知道爬到第 n−1n−1 层共有 f(n−1)f(n−1) 种

因此，爬到第 n 层的方法 f(n)=f(n−2)+f(n−1)f(n)=f(n−2)+f(n−1)。这一步，是 DP 最关键的一步，也就是找出递推公式。将求解到第 n 层的方法 f(n)f(n) 转化为 求解到第 n−1n−1 层 f(n−1)f(n−1) 和到 n−2n−2 层 f(n−2)f(n−2)。依次递归，直到 n 等于 2 或 1，且知道 f(2)=2f(2)=2， f(1)=1f(1)=1。

#### 2.2 递归版本1.0

根据上述思路，编写 Python 代码实现：

```
      class Solution:
            def climbStairs(self, n):
                if n==1:
                    return 1;
                if n==2:
                    return 2;
                return self.climbStairs(n-1) + self.climbStairs(n-2)

```

可以看到，代码非常简单。但是，这样求解算法的时间复杂度是多少呢？O(2n)O(2n)，指数级的时间复杂度显然不理想。

#### 2.3 降低 O(2n)O(2n)

这个世界上，没有免费的午餐，要想获得时间性能，你不得牺牲点啥。没错，动态规划法也不例外，它通过牺牲空间换来时间效率，这被称作 time-memory trade-off。所谓的空间，对于编码而言，就是申请一块内存空间。

##### **2.3.1 版本 1.1**

```
    class Solution:
        def climbStairs(self, n):
            dp = [1,2]
            for i in range(2,n):
                dp.append(dp[i-1] + dp[i-2])
            return dp[n-1]

```

这个算法的时间复杂度为 O(n)O(n)，空间复杂度为 O(n)O(n)。与版本 1.0 相比，这个已经很不错了，但是还有没有优化的空间呢？ 还是有的。注意观察，上述 1.1 代码，每次遍历，都会使得 DP 增加 1 个元素，实际情况是，我们只需要记忆 f(n−1)f(n−1) 和 f(n−2)f(n−2) 两个值就行，因此，代码进一步优化为：

##### **2.3.2 版本1.2**

```
    class Solution:
        def climbStairs(self, n):
            dp = [1,2]
            for i in range(2,n):
                tmp = dp[1]
                dp[1] = dp[0] + dp[1]
                dp[0] = tmp
            return dp[1] if n > 1 else 1

```

### 3. 机器人行走路径

这是一道面试题，计算从 start 到 finish 的最短路径，设机器人只能沿向下或向右方向行走。这道题的形象化表示如下所示：

![enter image description here](http://images.gitbook.cn/999f3cf0-6ddd-11e8-9ab5-a93e1135abc8)

#### 3.1 分析问题

要想从start 点到 finish 点，那么，必然经过 h1 或 h2 点，如下图所示：

![enter image description here](http://images.gitbook.cn/aeb1d490-6ddd-11e8-9ab5-a93e1135abc8)

所以，问题转化为：求 start 点到 h1 点，或到 h2 点的路径中的较小者，这相当于将问题域变小了 1，递归下去，直到问题域变为 1 个点。

总结出递归方程，分四种情况：

- 情况一：

![enter image description here](http://images.gitbook.cn/dc288020-6dda-11e8-9503-5f61421c20d6)

- 情况二：

![enter image description here](http://images.gitbook.cn/28540320-6ddb-11e8-9503-5f61421c20d6)

- 情况三：

![enter image description here](http://images.gitbook.cn/4c1944f0-6ddb-11e8-9503-5f61421c20d6)

- 情况四：

![enter image description here](http://images.gitbook.cn/8ec43990-6ddb-11e8-9a3a-91a7e0861642)

其中，dp[i,j]dp[i,j]：到第 i 行和第 j 列的最短距离；grid[i,j]grid[i,j]：网格的长度。注意每个网格长度可能不等，上面两幅图只是示意图。

有了状态转移方程，下面开始用 python 编码实现。输入的网格可能为，每一个元素代表网格的长度。

```
    Input:
    [
      [1,3,1],
      [1,5,1],
      [4,2,1]
    ]

```

#### 3.2 Python 代码实现

```
    class Solution:
        def minPathSum(self, grid):
            """
            :type grid: List[List[int]]
            :rtype: int
            """
            m, n = len(grid), len(grid[0])
            dp = [[] for _ in range(m) ] #记录到当前点的最短距离
            #边界情况
            dp[0].append(grid[0][0])
            for i in range(1,m):
                dp[i].append(dp[i-1][0] + grid[i][0])
            for j in range(1,n):
                dp[0].append(dp[0][j-1] + grid[0][j])
            # 迭代方程1
            for i in range(1,m):
                for j in range(1,n):
                    dp[i].append(min(dp[i-1][j],dp[i][j-1]) + grid[i][j])

            return dp[m - 1][n - 1]

```

```

```

在这个例子中，我们使用 `dp = [[] for _ in range(m) ]` ，记录到当前点的最短距离，这也是空间换取时间的经典DP 风格 （**time-memory trade-off**）。

### 4. DP 通用条件

DP 往往可以降低求解的时间复杂度，在面对一个问题时，我们怎么知道这个问题就能用 DP 来求解呢？

我们还得从面对的这个实际问题具有的特征入手，如果它符合动态规划的主要条件，那么就可以尝试用动态规划。

一般地，动态规划需要求解的问题具有 2 个核心特征：

1. optimal-substructure property：最优化的子结构属性
2. overlapping subproblems：重叠的子问题集

什么是最优化的子结构属性？ 如果一个问题的最优解包含在一系列的子问题集的最优解，那么它就称为具有最优化的子结构属性。

还是拿爬楼梯的例子，如果想求解爬到第 i 层楼梯的所有方法，不就是相当于求解以下这个子问题（subproblem）：

> 求爬到第 i-1 层的所有方法和爬到第 i-2层的所有方法两者的累计和，那么爬到第 i-1 层的所有方法，不又是一个子问题吗，相对于爬到第 i 层的方法不就叫 **subsubproblem** 吗！
>
> 这个爬楼梯问题是非常明显地具有最优化的子结构属性。

再来看下重叠的子问题是怎么定义的，算法导论上的解释如下：

> When a recursive algorithm revisits the same problem repeatedly, we say that the optimization problem has overlapping subproblems.

意思是说，当一个递归算法再次访问同一个问题时，这种事会出现一遍又一遍，我们说这个最优化问题有重叠的子问题集。

这个有点抽象，我试着解释下，还是爬楼梯，如果我们分别求出了爬到第 i-1、第 i-2 层的所有方法，那么在求解爬到第 i 层的所有方法时，我们还得去解决爬到第 i-1、第 i-2 层的所有方法，这就是重叠的子问题。

当然，记录下爬到第 i-1、第 i-2 层的所有方法，然后在用到的时候直接 look up 了，这也是提到的用空间换取时间的方法。

以上便是两个动态规划问题常常具备的特征，一般地，我们待解决的问题满足了这两个特征，就值得尝试动态规划

技术。

### 5. 深入 DP

先看原题，摘自 LeetCode 官网：

> Given a string containing just the characters '(' and ')', find the length of the longest valid (well-formed) parentheses substring. For "(()", the longest valid parentheses substring is "()", which has length = 2. Another example is ")()())", where the longest valid parentheses substring is "()()", which has length = 4.

题目的意思是求出有效括号的长度的最大值，举了几个例子：

```
(()   有效长度为2 
)()()) 有效长度为4 
((()()()) 有效长度为8

```

#### 5.1 初步分析

看到这个问题，首先要想一个问题，给定一个字符串，怎么判断这个字符串是否是有效的字符串括号呢？

根据直觉，我们可以想到借助栈，如果是 **(** 将之推到栈中； 如果是 **)** ，

- 栈顶如果是 **(** ，则表明找到一对有效的组合并出栈 **(**；
- 如果不是**(** ，说明这个不是有效字符括号串，或者栈为空了，进来一个 )，也表明不是有效的字符串。

找到判断一个字符串是否是有效的方法后，可以穷举这个字符串括号的所有子字符串，这个过程就得 O(n3)O(n3) 的时间复杂度。

#### 5.2 版本 1.0

```
    class Solution:
        def longestValidParentheses(self, s):
            def is_valid(sub):
                sta = []
                for it in sub:
                    if it == '(':sta.append('(')
                    elif len(sta) > 0 and sta[-1] == '(':sta.pop()
                    else: return False
                return len(sta)==0
            i= 0
            max_len = 0
            while i < len(s):
                j = i + 2
                while j <= len(s):
                    if is_valid(s[i:j]):
                        max_len = max(max_len, j-i)
                    j += 2  
                i += 1
            return max_len  

```

有没有更好的算法呢？上面解法尽管时间复杂度 O3O3，但判断字符串是否是有效字符仍有借鉴之处。基于此，借助动态规划技术，详细的分析过程如下：

1.如果一个括号串的结尾是 **(**，那么这个括号串一定不是有效的串；有效的括号串一定得以 **)** 结尾才行。

接下来，最重要的一个变量定义：

**dp[i]，表示以索引 i 结尾的字符串的最长长度**。此变量可以说是 此问题应用 DP 求解的关键一步，它也是 DP 通用条件的思想所在，原问题的最优解一定是由子问题的最优解组成的。

因为上文提到过，有效字符串一定得以 ) 结尾，并且研究两个连续相邻的字符，需要讨论两种情况，即

```
    (   ) 
    )   )

```

详细讨论如下，

2.如果 `str[i] = ')'`，并且 `str[i-1] = '('`，比如 `...( ) ( )`，在这种情况下的状态转换方程为

dp[i]=dp[i−2]+2dp[i]=dp[i−2]+2

3.如果 `str[i] = ')'`，并且 `str[i-1] = ')'`，比如 `...) )`，如果再满足 `str[ i - dp[i-1] - 1] = '('`，比如 `.....( ( ) ( ) )`，这种情况的状态转化方程为：

dp[i]=dp[i−1]+2+dp[i−dp[i−1]−2]dp[i]=dp[i−1]+2+dp[i−dp[i−1]−2]

这种情况不是很好理解，请参考下图理解，可以看到索引 `i` 此时等于 7，

dp[7]=dp[6]+2+dp[7−dp[6]−2]dp[7]=dp[6]+2+dp[7−dp[6]−2]

![enter image description here](http://images.gitbook.cn/dbe51c40-6ddf-11e8-9a3a-91a7e0861642)

至此这个问题，借助动态规划的思想，通过找到状态的转移方程，我们便找到了问题的解，这个问题的转化公式由以上两种情况组成。

因为这个问题相对抽象一点，为了更清楚地表达整个过程，特意放上一个 gif 演示：

![enter image description here](http://images.gitbook.cn/2c311970-6ddf-11e8-9503-5f61421c20d6)

#### 5.3 版本 2.0

```
    class Solution:
        def longestValidParentheses(self, s):
            maxlen = 0
            dp = [0 for i in range(len(s))]
            for i in range(1,len(s)):
                if s[i] == ')': #这是第一个大条件
                    if s[i-1] == '(':     #情况1
                        dp[i] = dp[i-2]+2 if i>=2 else 2 
                    elif i-dp[i-1] > 0 and s[i-dp[i-1]-1] == '(': #情况2
                        tmp = dp[i-dp[i-1]-2] if i-dp[i-1]>=2 else 0
                        dp[i] = dp[i-1] + 2 + tmp
                    maxlen = max(maxlen, dp[i]); 
            return maxlen

```

版本 2.0 的时间复杂度变为 O(n)。

### 6. 总结 DP

动态规划设计问题的一个关键步骤是找出问题实例的递推关系，该递推关系包含着该问题的更小子实例的解。最优解问题任意实例的最优解，都是由其子实例的最优解组成的，这被称为：**最优化法则** （principle of optimality）。

DP 问题为了在使用中得到最佳效果，需要保证对必要的子问题求解，且只解一次，它是以记忆功能（memory function）为基础的，通过维护一个自底向上 DP 算法使用的表格来实现。

### 7. 展望 DP

更多利用动态规划技术求解的经典问题，常见的还有：

1. 求解传递闭包的 Warshall 算法 ;
2. 求解完全最短路径问题的 Floyd 算法 ;
3. 最优查找二叉树的构造算法 ;
4. 除自身累乘算法（公号内推送过，也可以用 dfs 求解）

通常吃透以上这些算法后，就会对 DP 形成一个比较好的认识。DP 是相比于其他常用的分治，贪婪，DFS 是相对较难的技术，因此不管是在面试还是实际项目中，如果能很好的利用 DP 解决问题，说明你对算法的运用已经到了一个新的水平。

### 8. 参考文献

1. 《算法导论》
2. 《算法设计与分析基础》
3. <http://datastructur.es/sp16/>
4. <http://web.engr.illinois.edu/~jeffe/teaching/algorithms/all-algorithms.pdf>

最后推荐一本全面而基础的算法书（500+ pages），开源免费，Univ of Illinois 的 Prof. Erikson 编写的，包括：

> recursion, randomization, amortization, graph algorithms, network flows, hardness.
>
> 非常全面，强烈推荐！
>
> 下载地址：
>
> <http://web.engr.illinois.edu/~jeffe/teaching/algorithms/all-algorithms.pdf>