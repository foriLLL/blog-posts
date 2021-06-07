第二篇leetcode刷题笔记  。

## 7. 整数反转　　
### 题目　　
给你一个 32 位的有符号整数 x ，返回将 x 中的数字部分反转后的结果。　　

如果反转后整数超过 32 位的有符号整数的范围 [−231,  231 − 1] ，就返回 0。　　

假设环境不允许存储 64 位整数（有符号或无符号）。　　

### 思路：　
这是一道简单题，之所以放在这里，是因为其中题解提供的一些思路很值得学习，并且在其中我遇到的一些小问题也很值得记录下来以便以后回忆。  

#### 不借助栈的“弹出”和“推入”  
首先这个题第一思路是需要借助栈翻转，但作为数字，我们可以不使用栈就对其数字加以翻转。这是值得记录回忆的小技巧之一。  

```java
// 弹出 x 的末尾数字 digit
digit = x % 10
x /= 10

// 将数字 digit 推入 rev 末尾
rev = rev * 10 + digit
```

其次需要判断反转后的数字是否溢出，因为我们只能使用32位整型，所以不能使用`rev*10+digit >　Integer.MAX_VALUE`这样的语句来判断，但我们可以反其道而行，对`Integer.MAX_VALUE`除10来判断，这里题解中有一个值得学习的推导思路：  

考虑 x>0 的情况，记 $INT\\_MAX=2^{31}-1=2147483647$ ，由于

```latex
\begin{aligned} \textit{INT\_MAX}&=\lfloor\dfrac{\textit{INT\_MAX}}{10}\rfloor\cdot10+(\textit{INT\_MAX}\bmod10)\\ &=\lfloor\dfrac{\textit{INT\_MAX}}{10}\rfloor\cdot10+7 \end{aligned}
```

则不等式$rev⋅10+digit≤INT\\_MAX$

等价于

```latex
\textit{rev}\cdot10+\textit{digit}\le\lfloor\dfrac{\textit{INT\_MAX}}{10}\rfloor\cdot10+7
```

移项得

```latex
(\textit{rev}-\lfloor\dfrac{\textit{INT\_MAX}}{10}\rfloor)\cdot10\le7-\textit{digit}
```

讨论该不等式成立的条件：

若 $rev>\lfloor\cfrac{{INT\\_MAX}}{10}\rfloor$ ，由于 ${digit}\ge0$，不等式不成立。  

若 $rev=\lfloor\cfrac{{INT\\_MAX}}{10}\rfloor$
 ，当且仅当 ${digit}\le7$ 时，不等式成立。  


若 $rev<\lfloor\cfrac{{INT\\_MAX}}{10}\rfloor$
​，由于 ${digit}\le9$，不等式成立。

因此判定条件可简化为：当且仅当 $rev\le\lfloor\cfrac{{INT\\_MAX}}{10}\rfloor$ 时，不等式成立。

x<0 的情况类似.

**综上所述**，判断不等式

```latex
-2^{31}\le\textit{rev}\cdot10+\textit{digit}\le2^{31}-1
```

是否成立，可改为判断不等式
```latex
\lfloor\cfrac{-2^{31}}{10}\rfloor\le\textit{rev}\le\lfloor\dfrac{2^{31}-1}{10}\rfloor
```

是否成立，若不成立则返回 0。

---

解决了边界问题这个重点问题，剩余的部分就很简单了，需要判断结尾的`0`以及开头的`-`，在这里就不详细分析了。



## 1310. 子数组异或查询  　
### 题目　　
有一个正整数数组 arr，现给你一个对应的查询数组 queries，其中 queries[i] = [Li, Ri]。

对于每个查询 i，请你计算从 Li 到 Ri 的 XOR 值（即 arr[Li] xor arr[Li+1] xor ... xor arr[Ri]）作为本次查询的结果。

并返回一个包含给定查询 queries 所有结果的数组。

输入：arr = [1,3,4,8], queries = [[0,1],[1,2],[0,3],[3,3]]  
输出：[2,7,14,8]   
解释：  
数组中元素的二进制表示形式是：  
1 = 0001  
3 = 0011  
4 = 0100  
8 = 1000  
查询的 XOR 值为：  
[0,1] = 1 xor 3 = 2   
[1,2] = 3 xor 4 = 7   
[0,3] = 1 xor 3 xor 4 xor 8 = 14   
[3,3] = 8  


### 思路：
最初写了一个最简单的遍历想解决问题，设`arr`的长度为`n`，`queries`的长度为`m`，这样时间复杂度为$O(mn)$，结果运行超时，我开始思考怎么解决问题。  
后来看到题解“前缀异或”，突然想到，异或这东西不是有`x^x=0`这种好用的性质吗，题目要求的是对arr中间一段进行异或，那不是就可以后面的结果异或抵消前面的结果嘛。  
这里有一个示意可以方便理解。  

```
--------
^
---
=
   -----
```

这样只需要遍历一遍arr，得到arr.size()个结果保存起来，剩下的就是做异或相抵消得问题了。  
上代码：  
```cpp
class Solution {
	public:
		vector<int> xorQueries(vector<int>& arr, vector<vector<int> >& queries) {
			int len = arr.size();
			vector<int> rec(len+1, 0);
			//遍历arr得到arr个结果（第一个0作为初始值，作为最左值之前的结果）
			for(int i=0; i<len; i++) {
				rec[i+1] = (rec[i]^arr[i]);
			}
			vector<int> res;
			//右下标异或左下标之前的结果作抵消
			for(vector<int> lr: queries) {
				res.push_back(rec[lr[1]+1]^rec[lr[0]]);
			}
			return res;
		}
};
```


## 62. 不同路径 　
### 题目　　
一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。

问总共有多少条不同的路径？


### 思路：
#### 方法1：数学法
这个题一看到最初的思路是数学法，利用组合数 $C_{m+n-2}^{m-1}$ 。可以理解为一共有`m+n-2`个空，往其中填入`m-1`个黑棋，剩下的全部填入白棋，便得到了最终的结果，但实际写出来发现存在的问题是如果先计算分母极容易溢出。  
如果使用的语言有关于组合数的API直接调用即可，如果没有，可以考虑
```cpp
for (int x = n, y = 1; y < m; ++x, ++y) {
            ans = ans * x / y;
        }
```
(来自力扣官方题解)  

最初纠结这样一个一个分数会不会产生分数，后来看到网友的评论突然想明白了。

<img src="https://img.foril.fun/%E7%BB%84%E5%90%88%E6%95%B0.jpg" width='1000'/>

#### 方法2：动态规划
不多说废话，具体动态规划的要点可以查看我的另一篇博客 [专题：动态规划](https://www.foril.space/article/21)。  
这一题通过一个二维数组记录到对应点的最多方法。

上代码：
```cpp
int uniquePaths(int m, int n) {
    int dp[m][n];   //二维数组
    memset(dp,0,sizeof(dp));    //初始化（因为是memset针对字节，所以一般只能初始化为0）
    for(int i=0;i<m;i++){
        for(int j=0;j<n;j++){
            if(i == 0 || j==0){
                dp[i][j] = 1;   //边界值（初始化）
            }
            else{
                dp[i][j] = dp[i-1][j] + dp[i][j-1]; //状态转换方程
            }
        }
    }
    return dp[m-1][n-1];
}
```


## 406. 根据身高重建队列　
### 题目　　

假设有打乱顺序的一群人站成一个队列，数组 people 表示队列中一些人的属性（不一定按顺序）。每个 people[i] = [hi, ki] 表示第 i 个人的身高为 hi ，前面 正好 有 ki 个身高大于或等于 hi 的人。

请你重新构造并返回输入数组 people 所表示的队列。返回的队列应该格式化为数组 queue ，其中 queue[j] = [hj, kj] 是队列中第 j 个人的属性（queue[0] 是排在队列前面的人）。

示例：  
输入：people = [[7,0],[4,4],[7,1],[5,0],[6,1],[5,2]]  
输出：[[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]]


### 思路：
#### 方法一： 从高到低考虑
这个题一开始上手没有找到其中的窍门，感觉类似的题目的关键就是能够找到一个窍门，或者说一个隐藏的属性。  
这个题 ki 记录的是前面有多少身高大于等于 hi 的人数，言下的意思是前面如果放的是比他身高小的人，是没有任何影响的。那我们只要按照一定顺序填入队列就可以得到实际的位置。  
官方题解的原话是：
> 如果我们按照排完序后的顺序，依次将每个人放入队列中，那么当我们放入第 i 个人时：  
> * 第 0, ... , i-10, ... ,i−1 个人已经在队列中被安排了位置，并且他们无论站在哪里，对第 i 个人都没有任何影响，因为他们都比第 i 个人矮；  
> * 而第 i+1, ... ,i+1, ... ,n−1 个人还没有被放入队列中，但他们只要站在第 i 个人的前面，就会对第 i 个人产生影响，因为他们都比第 i 个人高。

所以我们要先找到大个的相对位置，把大个的放进队列里，这样其他人的位置不会影响到更大个的人的位置。同时，相同个子的人需要先放入前面人少的人，也就是得到了一个排序的方法：**先按第一列从大到小排序，第一列相同的按第二列从小到大排序**。
```cpp
//排序
sort(people.begin(), people.end() , [](const vector<int> & a, const vector<int> & b) {
    if(a[0] == b[0]) return a[1]<b[1];
    return a[0]>b[0];
});
```

那么，对示例`[[7,0],[4,4],[7,1],[5,0],[6,1],[5,2]]`来说，排序后的结果应该是：  
```
7,0
7,1
6,1
5,0
5,2
4,4
```
这也就是我们填入的顺序，每填入一个**当前最小个的人**，根据它前面的人数`ki`，把他放在第`ki`个位置，之后的全部往后移。最后全部填入，就是最终的顺序。  
这样频繁插入移动，在java中考虑使用 LinkedList 。

代码：
```cpp
vector<vector<int>> reconstructQueue(vector<vector<int>>& people) {
    //第零列从大到小
    //第一列从小到大哦
    int len = people.size();
    sort(people.begin(), people.end() , [](const vector<int> & a, const vector<int> & b) {
        if(a[0] == b[0]) return a[1]<b[1];
        return a[0]>b[0];
    });

    //排好序后开始放位置
    vector<vector<int>> list(len);
    for(int i = 0; i<len; i++) {
        int pos = people[i][1];
        for(int j = len-2; j>=pos; j--) {
            //往后窜
            list[j+1] = list[j];
        }
        list[pos] = people[i];
    }
    return list;
}
```

PS:  
后来发现往后窜的步骤可以通过 `insert`。
```cpp
for (const vector<int>& person: people) {
    ans.insert(ans.begin() + person[1], person);
}
```
#### 方法二： 从低到高考虑
同上，在排好序的队列中，个小的人是影响不到个高的人的 `ki` 的，所以我们可以按身高从低到高往队列中插入，前面只要留够 `ki` 个 “空座” 给之后个更高的人。对于相同身高的人，需要先排入 `ki` 大的人。  

> 这里需要注意的是排序方法和方法一刚好相反。  

对示例`[[7,0],[4,4],[7,1],[5,0],[6,1],[5,2]]`，排序后的结果应该是：  
```
4,4
5,2
5,0
6,1
7,1
7,0
```
有了这个顺序，便遵循 `留空 -> 插入` 的原则找到位置即可。  

<img src="https://img.foril.fun/%E6%A0%B9%E6%8D%AE%E8%BA%AB%E9%AB%98%E9%87%8D%E5%BB%BA%E9%98%9F%E5%88%97.webp" width=1000/>

代码：
```cpp
vector<vector<int>> reconstructQueue(vector<vector<int>>& people) {
    sort(people.begin(), people.end(), [](const vector<int>& u, const vector<int>& v) {
        return u[0] < v[0] || (u[0] == v[0] && u[1] > v[1]);
    });
    int n = people.size();
    vector<vector<int>> ans(n);
    for (const vector<int>& person: people) {
        int spaces = person[1] + 1;
        for (int i = 0; i < n; ++i) {
            if (ans[i].empty()) {
                --spaces;
                if (!spaces) {
                    ans[i] = person;
                    break;
                }
            }
        }
    }
    return ans;
}
```

## 96. 不同的二叉搜索树
### 题目　　

给你一个整数 n ，求恰由 n 个节点组成且节点值从 1 到 n 互不相同的 二叉搜索树 有多少种？返回满足题意的二叉搜索树的种数。

示例 1：  
输入：n = 3  
输出：5

<img src="https://assets.leetcode.com/uploads/2021/01/18/uniquebstn3.jpg" width=800>

### 思路：
如果了解过**卡塔兰数**的话，可以直接得到
```latex
C_{n+1} =  \frac{2(2n+1)}{n+2}C_n
```
> 令h(0)=1,h(1)=1，卡塔兰数满足递归式：
h(n)= h(0)*h(n-1) + h(1)*h(n-2) + ... + h(n-1)h(0) (其中n>=2),这是n阶递推关系;

以这个题作为背景的话，实际意义就是 n 个数对应分别以 1 到 n 作为根节点，也就是左孩子分别有 0 个到 n-1 个节点这n种情况的数量之和。  

## 664. 奇怪的打印机
### 题目　　

有台奇怪的打印机有以下两个特殊要求：

1. 打印机每次只能打印由 同一个字符 组成的序列。  
2. 每次可以在任意起始和结束位置打印新字符，并且会覆盖掉原来已有的字符。

给你一个字符串 s ，你的任务是计算这个打印机打印它需要的最少打印次数。

**示例 1：**  
输入：s = "aaabbb"  
输出：2  
解释：首先打印 "aaa" 然后打印 "bbb"。  
    
**示例 2：**  
输入：s = "aba"  
输出：2  
解释：首先打印 "aaa" 然后在第二个位置打印 "b" 覆盖掉原来的字符 'a'。  

### 思路：
之前在 [动态规划专题](https://www.foril.space/article/21) 中，提到过很多字符串最优问题都是使用动态规划解决的。  
这个题一上手，并没有找到解决的方法，主要问题是找不到状态转换方程，在查看题解后，发现题解将一个区间的最优问题转化为小区间的最优问题求和的最小值。后来从评论区明白这是一类叫 **“区间DP”** 的问题。  

#### 什么是区间DP？
区间dp就是在区间上进行动态规划，求解一段区间上的最优解。主要是通过 *合并小区间的最优解进而得出整个大区间上最优解* 的DP算法。  
既然让我求解在一个区间上的最优解，那么我把这个区间分割成一个个小区间，求解每个小区间的最优解，再合并小区间得到大区间即可。

朴素区间DP（$N^3$）转移方程：  
```cpp
dp[j][ends] = min(dp[j][ends],dp[j][i]+dp[i+1][ends]+weigth[i][ends]);
```
有了这些背景知识，就可以得到本题的基本思路。

动态规划的基本要素：  
* **保存**：`dp[i][j]`保存下标从`i`到`j`的最小打印次数
* **状态转移方程**： 
```latex
dp[i][j] = 
\begin{cases}
        dp[i][j-1] &,s[i]==s[j]\\
        min_{k=i}^{j-1} dp[i][k] + dp[k+1][j]&,s[i]!=s[j]
\end{cases}
```  
* **初始化**：`dp[i][i] = 1`
* **遍历顺序**：`i`从大到小；`j`从小到大
* **结果**：`dp[0][n-1]`

这就是做一道动态规划最重要的步骤了，这一题还有一个主要要理解的难点就是如何使用区间DP。  
上代码：  
```cpp
class Solution {
public:
    int strangePrinter(string s) {
        int n = s.length();
        int dp[n][n];
        //初始化
        for(int i = 0;i<n;i++){
            dp[i][i] = 1;
        }
        //状态转移方程
        for(int i = n-1;i>=0;i--){
            for(int j = i+1;j<n;j++){
                if(s[i] == s[j]){
                    dp[i][j] = dp[i][j-1];
                }else{
                    int minR = INT_MAX;
                    for(int k = i;k<j;k++){
                        minR = min(minR, dp[i][k] + dp[k+1][j]);
                    }
                    dp[i][j] = minR;
                }
            }
        }
        //结果
        return dp[0][n-1];
    }
};
```
## 39. 组合总和
### 题目　　

给定一个无重复元素的数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。

candidates 中的数字可以无限制重复被选取。

说明：

所有数字（包括 target）都是正整数。
解集不能包含重复的组合。 

### 思路：
这个题是在刷回溯专题时遇到的。接下来几个题都有很典型的回溯算法的结构。
```cpp
class Solution {
private:
    vector<vector<int>> res;    //存放结果
    void bt(vector<int>& v,int index, int already,int target, vector<int>& candidates){
        if(already>target){ //出口条件（不合格，舍弃）
            return;
        }
        if(already==target){//出口条件（是结果，加入结果数组）
            res.push_back(v);
            return;
        }else{
            //继续往下试探
            for(int i = index;i<candidates.size();i++){ //尝试不同的数字作为下一个候选

                v.push_back(candidates[i]); //第一部分，把当前数字加入结果往下试探

                bt(v, i, already+candidates[i], target, candidates);    //递归试探过程

                v.pop_back();   //试探结束后，为了使用下一个数字，要把当前数字所做的改变全部复原
            }
        }
    }
public:
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        vector<int> v;
        bt(v, 0, 0, target, candidates);    //进入
        return res;
    }
};
```

## 40. 组合总和 II

### 题目　　

给定一个数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。

candidates 中的每个数字在每个组合中只能使用一次。

说明：

所有数字（包括目标数）都是正整数。
解集不能包含重复的组合。 

**示例 1:**

输入: candidates = [10,1,2,7,6,1,5], target = 8,
所求解集为:

[  
  [1, 7],  
  [1, 2, 5],  
  [2, 6],  
  [1, 1, 6]  
]  

**示例 2:**

输入: candidates = [2,5,2,1,2], target = 5,
所求解集为:

[  
  [1,2,2],  
  [5]  
]  

### 思路：
这一题和上一题的不同在于有了**重复的数字**，上一个题中所有数字是排好了序的。这就会出现一部分结果相同但顺序不同的相同解重复的问题。  
比如：`[1, 7] 和 [7, 1]`。这一点比较麻烦，我个人的题解中，是通过先将所有数字排序，这样相同的数字会在一起，就不会出现`[1, 7] 和 [7, 1]`这种情况，再利用set来除重，就有了这一种虽然低效但是也可用的解法。

```cpp
class Solution {
private:
    set<vector<int>> ans;
    void bt(vector<int>& v, int index, int remain, vector<int>& candidates){
        if(remain==0){  //合格出口，加入结果
            ans.insert(v);
            return;
        }
        if(remain<0){   //不合格题解
            return;
        }
        for(int i = index+1;i<candidates.size();i++){
            v.push_back(candidates[i]); //加入当前数字，继续回溯

            bt(v, i, remain-candidates[i], candidates); //递归试探

            v.pop_back();   //试探结束后，为了继续使用其他数字，需要把影响复原，即再弹出当前数字。不用在意bt函数里进行了什么操作，只需要把当前自己加进去的数字删除即可
        }
    }
public:
    vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
        sort(candidates.begin(), candidates.end());
        vector<int> v;
        bt(v, -1, target, candidates);
        vector<vector<int>> ret;
        ret.assign(ans.begin(), ans.end());
        return ret;
    }
};
```

## 46. 全排列
### 题目　　

给定一个不含重复数字的数组 nums ，返回其 所有可能的全排列 。你可以 按任意顺序 返回答案。

示例 1：

输入：nums = [1,2,3]  
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]  

### 思路：
按照我个人的思路，也是典型的回溯算法几个重要特征。
```cpp
class Solution {
public:
    vector<vector<int>> res;
    void backTrack(vector<int>& unused, vector<int>& shape ){
        if(unused.size()==0){   //出口条件
            res.push_back(shape);
            return;
        }
        for(int i = 0;i<unused.size();i++){ 
            //加入当前数字需要做的变化
            int tmp = unused.at(i);
            unused.erase(unused.begin()+i);
            shape.push_back(tmp);

            backTrack(unused, shape);   //继续试探

            //撤销所有影响
            shape.pop_back();
            unused.insert(unused.begin()+i, tmp);
        }
    } 
    vector<vector<int>> permute(vector<int>& nums) {
        vector<int> shape;
        backTrack(nums, shape);
        return res;
    }
};
```


## 542. 01 矩阵
### 题目　　
给定一个由 0 和 1 组成的矩阵，找出每个元素到最近的 0 的距离。

两个相邻元素间的距离为 1 。


**示例 1：**

输入：  
[[0,0,0],  
 [0,1,0],  
 [0,0,0]]  

输出：  
[[0,0,0],  
 [0,1,0],  
 [0,0,0]]  

### 思路：
这个题一拿到手感觉像是动态规划的问题，但是每一个点的最小值可能是其上下左右四个值中最小值加一，我找不到合适的遍历方式，所以换了一个比较古怪的思路。
### 方法一： 
所求答案矩阵`ans`和输入矩阵`input`形状一样，记为`m*n`的矩阵，将`ans`的所有制初始化为`INT_MAX`，以此遍历`i, j`，到每一个格时更新他为 0 或者 上下左右四个方向上的最小值加一（比原来的位置上距离小），每一轮遍历如果有更新，则继续遍历，直到没有更新为止。  
这个方法实际提交能过，但是由于每一次有更新都可能需要多轮遍历来发散开，收敛相对很慢。所以提交结果比较差。

```cpp
class Solution {
public:
    
    vector<vector<int>> updateMatrix(vector<vector<int>>& mat) {
        int rows = mat.size();
        int cols = mat[0].size();
        vector<vector<int>> res(rows, vector<int>(cols,INT_MAX));
        int directions[][2] = {{-1,0},{1,0},{0,-1},{0,1}};
        while(1){
            //一直遍历直到没有更新
            int count = 0;
            for(int i = 0;i<rows;i++){
                for(int j = 0;j<cols;j++){
                    if(mat[i][j]==0 && res[i][j]!=0){
                        res[i][j] = 0;
                        count++;    //有更新
                    }
                    else{
                        for(int k = 0;k<4;k++){ //四个方向
                            int next_i = i+directions[k][0];
                            int next_j = j+directions[k][1];
                            if(next_i<0 || next_i==rows || next_j<0 || next_j==cols){
                                continue;   //碰壁，下一个方向
                            }
                            if(res[next_i][next_j]!=INT_MAX && res[next_i][next_j] + 1 < res[i][j]){
                                res[i][j] = res[next_i][next_j] + 1;    //更新最小值
                                count++;    //有更新
                            }
                        }
                    }
                }
            }
            if(!count) break;  //没有更新
        }
        return res;
    }
};
```

### 方法二：动态规划  
看了题解发现，虽然每一个格的的实际结果跟上下左右四个方向都可能有关系，但也可以通过动态规划解决这个问题，只需要分别考虑最近的 0 点在该点的**左上，左下，右上，右下**<u>四个方向上分别动态规划</u>，得到四个对应的`m*n`的矩阵，只需要每个点都取这四个矩阵中对应点的最小值，就可以得到最后的结果。
```
      |
  zs  |  ys
      |
------1------
      |
  zx  |  yx
      |
```
```cpp
class Solution {
public:
    vector<vector<int>> updateMatrix(vector<vector<int>>& matrix) {
        int m = matrix.size(), n = matrix[0].size();
        // 初始化动态规划的数组，所有的距离值都设置为一个很大的数
        vector<vector<int>> dist(m, vector<int>(n, INT_MAX / 2));
        // 如果 (i, j) 的元素为 0，那么距离为 0
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (matrix[i][j] == 0) {
                    dist[i][j] = 0;
                }
            }
        }
        // 只有 水平向左移动 和 竖直向上移动，注意动态规划的计算顺序
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (i - 1 >= 0) {
                    dist[i][j] = min(dist[i][j], dist[i - 1][j] + 1);
                }
                if (j - 1 >= 0) {
                    dist[i][j] = min(dist[i][j], dist[i][j - 1] + 1);
                }
            }
        }
        // 只有 水平向左移动 和 竖直向下移动，注意动态规划的计算顺序
        for (int i = m - 1; i >= 0; --i) {
            for (int j = 0; j < n; ++j) {
                if (i + 1 < m) {
                    dist[i][j] = min(dist[i][j], dist[i + 1][j] + 1);
                }
                if (j - 1 >= 0) {
                    dist[i][j] = min(dist[i][j], dist[i][j - 1] + 1);
                }
            }
        }
        // 只有 水平向右移动 和 竖直向上移动，注意动态规划的计算顺序
        for (int i = 0; i < m; ++i) {
            for (int j = n - 1; j >= 0; --j) {
                if (i - 1 >= 0) {
                    dist[i][j] = min(dist[i][j], dist[i - 1][j] + 1);
                }
                if (j + 1 < n) {
                    dist[i][j] = min(dist[i][j], dist[i][j + 1] + 1);
                }
            }
        }
        // 只有 水平向右移动 和 竖直向下移动，注意动态规划的计算顺序
        for (int i = m - 1; i >= 0; --i) {
            for (int j = n - 1; j >= 0; --j) {
                if (i + 1 < m) {
                    dist[i][j] = min(dist[i][j], dist[i + 1][j] + 1);
                }
                if (j + 1 < n) {
                    dist[i][j] = min(dist[i][j], dist[i][j + 1] + 1);
                }
            }
        }
        return dist;
    }
};

```

## 523. 连续的子数组和
### 题目　　
给你一个整数数组 nums 和一个整数 k ，编写一个函数来判断该数组是否含有同时满足下述条件的连续子数组：

* 子数组大小 至少为 2 ，且
* 子数组元素总和为 k 的倍数。

如果存在，返回 true ；否则，返回 false 。

如果存在一个整数 n ，令整数 x 符合 x = n * k ，则称 x 是 k 的一个倍数。

### 思路：
### 思路一：动态规划
第一次提交，觉得这是一个动态规划题目，dp[i][j]记录从i到j的和。又根据题目，只要出现是k的倍数便可以停止遍历，不需要之前的结果，所以可以用两个变量分别记录上次结果（余数）和这次的数（余数），若为0便可以停止遍历。
```cpp
class Solution {
public:
    bool checkSubarraySum(vector<int>& nums, int k) {
        int len = nums.size();
        int before, curr;
        //init
        for(int i = 0;i<len;i++){
            before = nums[i]%k;
            for(int j = i+1;j<len;j++ ){
                curr = (before + nums[j])%k;
                before = curr;
                if(curr == 0) return true;
            }
        }
        return false;
    }
};
```

但是最后两个用例足有500KB，使用动态规划思想，时间复杂度为 $O(n^2)$ ，无法解决超时问题。

### 思路二：前缀和 + 集合
看到这个题目的关键字 **连续的** 、**数组和** ，应该想到利用前缀和，只需要分别计算从下标 0 到下标 i 的数组前缀和，只需要 $O(n)$ 的空间`sum[n]`，就可以知道任意从 a 到 b 的数组之和为`sum[b] - sum[a-1]`。这便是前缀和的思想。  
值得注意的是一般从开头开始的序列需要减去值 0 ，我们可以通过开辟长度为`n+1`的数组或是初始化before变量为 0 来解决。
```
□□□□□□□□□
-
□□□□
=
    □□□□□
```
为了继续优化，我们考虑如果`sum[j] - sum[i]`即从`i+1`到`j`的数组和是`k`的倍数，则`sum[j]`和`sum[i]`的余数应该相同，可以使用一个**集合Set**来存放遇到过的所有余数，但要注意的是为了保证长度至少为2，我们应该在下一次遍历只后再放入上一次的余数结果。这样对于开头第一个数字，集合为空，对其他所有数字，看不到上一次的余数结果，就保证了长度至少为2。
```cpp
class Solution {
public:
    bool checkSubarraySum(vector<int>& nums, int k) {
        int len = nums.size();
        set<int> set;
        int before = 0, curr;
        for(int i = 0;i<len;i++){
            curr = (before + nums[i])%k;
            if(set.find(curr)!=set.end()) return true;
            set.insert(before);
            before = curr;
        }
        return false;
    }
};
```

## 416. 分割等和子集
### 题目　　
给你一个 只包含正整数 的 非空 数组 nums 。请你判断是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。

### 思路：0-1背包问题
容易想到的是，只需要判断是否能够从数组中选出一个子集使其和为原数组总和的一半即可。  
因此将这个题转换为 `0-1背包问题` ，每个数字可以拿或者不拿，需要刚好凑够总和的一半。  

### 最开始的判断
先进行以下判断，之后才可以进行动态规划求解：  
1. 总和 sum 是否为偶数，否则返回 false ，是偶数则令 target = sum/2
2. 其中最大值是否大于 target ，是则直接返回 false ，如果没有这一步判断，在下面的动态规划中会导致下标访问直接溢出  

### 动态规划
创建二维数组 `dp[n][target+1]` ，包含 n 行 target+1 列，其中 dp[i][j] 表示从数组的 [0,i] 下标范围内选取若干个正整数（可以是 0 个），是否存在一种选取方案使得被选取的正整数的和等于 j。初始时，dp 中的全部元素都是 false。  
**注意：** 需要 `target+1` 列来包括 `0~target`   

```cpp
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int sum = accumulate(nums.begin(), nums.end(), 0);  //求和

        //最开始的判断（防止溢出）
        if(sum%2) return false;
        int target = sum/2;
        int maxNum = *max_element(nums.begin(), nums.end());
        if(maxNum>target){
            return false;
        }

        int len = nums.size();
        vector<vector<bool>> dp(len,vector<bool>(target+1, false));
        //初始化
        for(int i =0;i<len;i++){
            dp[i][0] = true;
        }
        dp[0][nums[0]] = true;

        for(int i = 1;i<len;i++){
            int num = nums[i];
            for(int j = 1;j<=target;j++){
                //状态转换方程
                if(j - num <0){
                    dp[i][j] = dp[i-1][j];
                }else{
                    dp[i][j] = dp[i-1][j] ||  dp[i-1][j-num];
                }
            }
        }
        //所需结果
        return dp[len-1][target];
    }
};
```

## 525. 连续数组
### 题目　　
给定一个二进制数组 nums , 找到含有相同数量的 0 和 1 的最长连续子数组，并返回该子数组的长度。

### 思路：
看到关键词 **“最长”** 、**“连续子数组”**， 可以想到本地可以使用 **前缀和+哈希表** 来解决。具体前缀和原理可参考上面 523. 连续的子数组和 。

为了找到含有相同数量的 0 和 1 的最长连续子数组，可以使用一个变量 `count` 记录0和1的出现，见到1加一，见到的0减一。  
也就是说，如果一个子数组 `[i,j]` 中 0 和 1 数量相同，这个区间的 `count` 应该为零，也就是 `count[j]-count[i-1] = 0` ，在实际的实现中，不需要数组来记录所有的count[i]，只需要使用一个哈希表，记录所有之前出现过的count，如果新的count在哈希表里，只需要比对是否是新的最大长度，否则就将这个count加入哈希表中。
```cpp
class Solution {
public:
    int findMaxLength(vector<int>& nums) {
        int len = nums.size();
        unordered_map<int, int> hashMap;
        hashMap.insert({0,0});  //注意要加入初始值，如果使用数组保存，应该多声明一个，记录初始状态
        int count = 0;
        int max_num = 0;
        for(int i =1;i<=len;i++){
            //遍历每个数
            if(nums[i-1]==1) count++;
            else count--;
            auto it = hashMap.find(count);
            if(it!=hashMap.end()){
                //存在
                max_num = max(max_num,i - it->second);
            }else{
                //插入
                hashMap.insert({count, i});
            }
        }
        return max_num;
    }
};
```