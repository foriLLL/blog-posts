---
description: "数位DP（Digit Dynamic Programming，数位动态规划）是一种动态规划算法的应用，用于解决与数字相关的问题，特别是涉及数字位数的计数、组合、计算等问题。"
time: 2022-11-28
---

## 题目特征

数位 DP：用来解决一类特定问题，这种问题比较好辨认，一般具有这几个特征：
* 要求统计满足一定条件的数的数量（即，最终目的为计数）；
* 这些条件经过转化后可以使用「数位」的思想去理解和判断；
* 输入会提供一个数字区间（有时也只提供上界）来作为统计的限制；
* 上界很大（比如 $10^8$），暴力枚举验证会超时。

## 基本原理

考虑人类计数的方式，最朴素的计数就是从小到大开始依次加一。但我们发现对于位数比较多的数，这样的过程中有许多重复的部分。例如，从 7000 数到 7999、从 8000 数到 8999、和从 9000 数到 9999 的过程非常相似，它们都是后三位从 000 变到 999，不一样的地方只有千位这一位，所以我们可以把这些过程归并起来，将这些过程中产生的计数答案也都存在一个通用的数组里。此数组根据题目具体要求设置状态，用递推或 DP 的方式进行状态转移。

数位 DP 中通常会利用常规 **计数问题技巧**，比如把一个区间内的答案拆成两部分相减（即 $ans_{\lbrack l, r \rbrack} = ans_{\lbrack 0, r \rbrack} - ans_{\lbrack 0, l-1 \rbrack}$）。

那么有了通用答案数组，接下来就是统计答案。统计答案可以选择记忆化搜索，也可以选择循环迭代递推。为了不重不漏地统计所有不超过上限的答案，要从高到低枚举每一位，再考虑每一位都可以填哪些数字，最后利用通用答案数组统计答案。

针对许多需要排除重复结果的题目，通常可以引入 mask 来记录已经使用过的数字，下面提供一个带注释的模板。

## 例题

### [2376. 统计特殊整数](https://leetcode.cn/problems/count-special-integers/)

```java
class Solution {
    int[][] memo;   // memo[i][mask]记录当前选择顺位为i，已选状态为mask时，构造第i位及后面位的合法方案数
    char[] s;

    public int countSpecialNumbers(int n) {
        /*
        参考灵神の数位DP记忆化DFS模板：
        注意这题与LC1012是一样的，不过这题更直接求每一位都不相同数字
        dfs(i, mask, isLimit, hasNum) 代表从左到右选到第i个数字时(i从0开始)，前面数字已选状态为mask时的合法方案数
        各个参数的含义如下:
        i:当前选择的数字位次，从0开始
        mask:前面已择数字的状态，是一个10位的二进制数，如:0000000010就代表前面已经选了1
        isLimit:boolean类型，代表当前位选择是否被前面位的选择限制了；
            如最大的数n=1234，前面选了12都是贴着最大的选择，那么 isLimit=true，选第3位的时候会被限制在0~3，否则超过n；否则是0~9，isLimit=false
        hasNum:表示前面是否已经选择了数字，若选择了就为true(识别直接构造低位的情况)
        时间复杂度:O(1024*M*10) 空间复杂度:O(1024*M)
        记忆化DFS的时间复杂度=状态数*每一次枚举的情况数
        **记忆化本质就是减少前面已选状态一致的情况，将1eM的时间复杂度压缩至1<<M，效率非常高**
         */
        s = String.valueOf(n).toCharArray();    // 转化为字符数组形式
        int m = s.length;
        memo = new int[m][1 << 10];     // i∈[0,m-1]，mask为一个10位二进制数
        // 初始化memo为-1代表该顺位下该已选状态还没进行计算
        for (int i = 0; i < m; i++) {
            Arrays.fill(memo[i], -1);
        }
        // 注意一开始最高位是有限制的，isLimit=true
        return dfs(0, 0, true, false);
    }

    // dfs(i, mask, isLimit, hasNum) 代表从左到右选第i个数字时，前面已选状态为mask时的合法方案数
    private int dfs(int i, int mask, boolean isLimit, boolean hasNum) {
        // base case
        // i越过最后一位，此时前面选了就算一个，没选的就不算，因为不选后面也没得选了
        if (i == s.length) return hasNum ? 1 : 0;
        // 已经计算过该状态，并且该状态是有效的，直接返回该状态
        // 这一步是降低时间复杂度的关键，使得记忆化dfs的时间复杂度控制得很低
        // !isLimit表示没有被限制的才可以直接得出结果，否则还要根据后面的数字进行计算子问题计算
        if (!isLimit && hasNum && memo[i][mask] != -1) return memo[i][mask];
        int res = 0;    // 结果
        // 本位可以取0(可直接构造低位数)的情况，此时要加上构造低位数0xxx的方案数
        // 将是否选了数字作为分类条件是为了避免出现00010这样有多个0的就不能统计了
        if (!hasNum) res = dfs(i + 1, mask, false, false);
        // 构造与当前顺位相同位数的数字就要枚举可选的数字进行DFS
        // 枚举的起点要视hasNum而定，如果前面选择了数字，那么现在可以选0；否则只能从1开始
        // 枚举得终点视isLimit而定，若被限制了只能到s[i]，否则可以到9
        for (int k = hasNum ? 0 : 1, end = isLimit ? s[i] - '0' : 9; k <= end; k++) {
            // 如果该数字k还没有被选中，那猫就可以选该位数字
            if (((mask >> k) & 1) == 0) {
                // 方案数遵循加法原理
                // i:进行下一位的DFS，因此为i+1
                // mask:由于该位选中了k，mask掩膜传下去就要更新，已选状态加上k
                // isLimit:当且仅当前面的被限制了且该位被限制
                // hasNum:该位选了必定为true
                res += dfs(i + 1, mask | (1 << k), isLimit && k == end, true);
            }
        }
        if (!isLimit && hasNum) memo[i][mask] = res;    // 如果前面没有限制，表明后面都是同质的，可以记录进memo中
        return res;
    }
}
```

### [233.数字1的个数](https://leetcode.cn/problems/number-of-digit-one/)

```py
class Solution:
    def countDigitOne(self, n: int) -> int:
        s = str(n)
        
        # 这里没有 mask，所以可以不需要 isNum，全为 0 也是一种状态，对答案没有影响
        @cache
        def f(i: int, cnt1: int, is_limit: bool) -> int:
            if i == len(s): return cnt1
            res = 0
            upper = int(s[i]) if is_limit else 9
            for d in range(upper + 1):  # 枚举要填入的数字 d
                res += f(i + 1, cnt1 + (d == 1), is_limit and d == up)
            return res
        return f(0, 0, True)
```

### [2719.统计整数数目](https://leetcode.cn/problems/count-of-integers/)

这个题目的核心约束是两个范围，我们可以用类似前缀和的概念，解决第一个约束：用满足上界以内的数量减去满足下界以内的数量。  
对于第二个约束，在 DP 的过程中，我们可以用一个变量记录当前的数位和，然后在 DP 的过程中，用这个变量来判断是否满足第二个约束。
对于 isNum 的约束，因为全为 0 也是一种状态，所以可以不需要 mask 来记录已经使用过的数字。

```py
class Solution:
    def count(self, num1: str, num2: str, min_sum: int, max_sum: int) -> int:
        def calc(high: str) -> int:
            @cache
            def dfs(i: int, s: int, is_limit: bool) -> int:
                if s > max_sum:  # 非法
                    return 0
                if i == len(high):
                    return s >= min_sum
                res = 0
                up = int(high[i]) if is_limit else 9
                for d in range(up + 1):  # 枚举当前数位填 d
                    res += dfs(i + 1, s + d, is_limit and d == up)
                return res
            return dfs(0, 0, True)

        is_num1_good = min_sum <= sum(map(int, num1)) <= max_sum
        return (calc(num2) - calc(num1) + is_num1_good) % 1_000_000_007
```

## 参考

* https://oi-wiki.org/dp/number/
* https://leetcode.cn/problems/count-special-integers/solution/shu-wei-dp-mo-ban-by-endlesscheng-xtgx/
* https://leetcode.cn/problems/number-of-digit-one/solutions/1750339/by-endlesscheng-h9ua/