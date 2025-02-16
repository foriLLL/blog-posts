---
description: "在实验时，我发现你有些合并冲突块实际上可以使用一些更细粒度的编辑脚本生成算法解决，也就是说其实在 single head 合并中双边的冲突其实针对的是不同的代码行，可以使用最朴素的采纳双边编辑的思想来解决，但是却被 Git 算作是冲突。所以想了解一下 histogram diff 算法的具体内容，以下是我的一些总结。"
time: 2023-09-25T14:50:11+08:00
heroImage: ""
tags: []
---

从 Git 2.33 版本开始，默认的 git-merge 策略已经变成了 `merge-ort`，而不是 `merge-recursive` 了，这个策略所采用的默认 diff 算法是 `histogram`（详见 [文档](https://git-scm.com/docs/merge-strategies/#Documentation/merge-strategies.txt-diff-algorithmpatienceminimalhistogrammyers)）。在实验时，我发现你有些合并冲突块实际上可以使用一些更细粒度的编辑脚本生成算法解决，也就是说其实在 single head 合并中双边的冲突其实针对的是不同的代码行，可以使用最朴素的采纳双边编辑的思想来解决，但是却被 Git 算作是冲突。所以想了解一下 histogram diff 算法的具体内容，以下是我的一些总结。

<img alt="20230925145030" src="https://img.foril.space/20230925145030.png" width=600px style="display: block; margin:10px auto"/>

## 什么是 Histogram Diff 算法

参考 jGit 中对 Histogram 的 [实现文档](https://archive.eclipse.org/jgit/docs/jgit-2.0.0.201206130900-r/apidocs/org/eclipse/jgit/diff/HistogramDiff.html)，Histogram 算法是 Bram Cohen 的 patience 差异算法的扩展形式。该实现是通过使用 [Bram Cohen 博客](https://bramcohen.livejournal.com/73318.html) 中概述 Patience 算法优势的 4 条规则衍生的，然后进一步扩展以支持低出现率的常见元素。

该算法的基本思想是为序列 A 的每个元素创建 **出现次数的直方图**。然后依次考虑序列 B 的每个元素。如果该元素也存在于序列 A 中，并且出现次数较低，则该位置被视为最长公共子序列 (LCS) 的候选位置。 B 扫描完成后，**选择出现次数最少的元素作为分割点**。该区域围绕 LCS 进行分割，并将算法递归地应用于 LCS 之前和之后的部分。

通过始终选择出现次数最少的 LCS 位置，每当两个序列之间存在唯一的公共元素时，该算法的行为就与 Bram Cohen 的 patience diff 完全相同。当不存在唯一元素时，将选择出现次数最少的元素。与简单地依靠标准 Myers 的 $O(ND)$ 算法产生的结果相比，Histogram 提供了更有可读性的 diff。

```js
// lcs函数用于寻找两个字符串序列的最长公共子序列
function lcs(as, bs, a0, a1, b0, b1) {
    // 从顶部和底部跳过相等的元素
    let hs = [], ts = [] // hs用于存储开头相同的元素，ts用于存储结尾相同的元素
    while (as[a0] == bs[b0] && a0 < a1 && b0 < b1) {
      hs.push(as[a0]); a0++; b0++
    }
    while (as[a1-1] == bs[b1-1] && a0 < a1 && b0 < b1) {
      ts.push(as[a1-1]); a1--; b1--
    }
    ts.reverse() // 反转ts数组以保持正确的顺序

    // 构建直方图，记录每个字符在两个字符串中的出现次数和位置
    let hist = {}
    for (let i = a0; i < a1; i++) {
      let rec = hist[as[i]]
      if (rec) { rec.ac++; rec.ai = i }
      else hist[as[i]] = { ac: 1, ai: i, bc: 0, bi: -1 }
    }
    for (let i = b0; i < b1; i++) {
      let rec = hist[bs[i]]
      if (rec) { rec.bc++; rec.bi = i }
      else hist[bs[i]] = { ac: 0, ai: -1, bc: 1, bi: i }
    }

    // 找到在两个字符串中都出现且出现次数最少的字符
    let cmp = Number.MAX_VALUE // 用于存储最小的出现次数
    let p = null // 存储这个字符
    for (let k in hist) {
      let rec = hist[k]
      if (rec.ac > 0 && rec.bc > 0 && rec.ac + rec.bc < cmp) {
        p = k; cmp = rec.ac + rec.bc
      }
    }

    // 如果没有这样的字符，则返回已经找到的相同子序列
    if (!p) return [...hs,...ts]

    let rec = hist[p]
    // 递归地寻找左边和右边的最长公共子序列，并将它们与当前找到的字符合并
    return [...hs, ...lcs(as,bs,a0,rec.ai,b0,rec.bi),
                p, ...lcs(as,bs,rec.ai+1,a1,rec.bi+1,b1), ...ts]
}

/*
Left: A,A,B,C,D,E,F,G
Right: A,A,X,Y,Z,D,E,F
Common: A,A,D,E,F
*/
```

## 核心思想

所有的 diff 算法的核心其实都是为了找到两个文件的最长公共子序列，有了找到的 LCS，
两个文件通过迭代直到找到所有的最长公共子序列，然后将这些结果合并起来就得到了最终的 diff 结果。

```js
function diff(as,bs) {
  let ds = lcs(as, bs, 0, as.length, 0, bs.length)
  let ai = 0
  let bi = 0
  for (let di = 0; di < ds.length; di++) {
    while (ai < as.length && as[ai] != ds[di])
      console.log("- " + as[ai++])
    while (bi < bs.length && bs[bi] != ds[di])
      console.log("+ " + bs[bi++])
    console.log("  " + ds[di]); ai++; bi++
  }
  while (ai < as.length)
    console.log("- " + as[ai++])
  while (bi < bs.length)
    console.log("+ " + bs[bi++])
}
/*
=== LEFT: ===
function foo() {
print("yo")
}
=== RIGHT: ===
// some comment
print("yo")
=== DIFF: ===
- function foo() {
+ // some comment
print("yo")
- }
*
```

histogram 的直译是直方图，实际上这个算法的核心思想以及与其他 diff 算法不同的地方就是通过在找到的公共元素中不唯一时，**选择的是出现次数最少的元素作为分割点**。之后与其他算法类似，将两个序列分割成两个子序列，递归寻找子序列的最长公共子序列，直到找到所有的最长公共子序列，然后将这些结果合并起来。

## 回到最初的例子

回到 Git 没能解决相邻行的编辑的问题，以默认的 ort merge 作为例子，说说 Git-merge 的源码是怎么生成冲突的。

跳过前半部分在命令行输入 `git merge`，一步一步通过内置命令定位仓库和文件取出对应两个版本 A 和 B 的文件和它们的 base 文件的琐碎部分。  
在 `xmerge.c` 中，首先会针对于 A 和 B 分别做两次 diff，得到的结果是两边行数分别对于 base 以及在 base 中变更的行的标识。

```c
xdl_do_diff(orig, mf1, xpp, &xe1)
xdl_do_diff(orig, mf2, xpp, &xe2)
```

接着，对于 histogram diff，方法会转移到 `xhistogram.c` 下的 `histogram_diff` 方法，在这里，变更后的版本会和 base 版本做比较，递归进行最长公共子序列（LCS）的查找，首先对 A 中每一行建立哈希（建立哈希的目的是在比较时有 $O(1)$ 的时间复杂度，直接比较哈希值即可，不需要逐字比较），建立好的哈希根据后几位分桶（检查时再对比详细哈希），接着对 B 中的行也建立哈希，从第一行开始匹配，对每一行，如果这一行在 A 中找到内容相同的行，**向上向下** 进行扩展找到最长连续相同的块，直到不相同或一方文件结束（*因为 B 中可能和 A 中很多行内容都相同，所以这里实际要对 A 中所有相同的哈希的位置都上下找一遍最大连续相同的块*），最后如果得到的连续相同块比原来找到的块都大，或是块一样大，但是这一行出现的次数更少（更有代表性）。

> 这里和上面的简单的 histogram diff 的不同之处在于，他不是通过先删除开头和结尾匹配的行来去除最大连续块，而是通过向上向下扩展找到最大连续块，连续块的长度比行出现次数少的优先级更高。

接着对找到的 LCS 上面的部分和下面的部分 **递归** 进行最长公共子序列的查找，直到找到所有的 LCS，然后将这些结果合并起来。

例如下面这个冲突得到的两个 diff 结果就是这样：

<img alt="20230925145030" src="https://img.foril.space/20230925145030.png" width=600px style="display: inline-block; margin:10px auto"/>

```
xe1 记录 A 与 base 的 diff 结果   xdf1 对应的都是 base 的结果
xe2 记录 B 与 base 的 diff 结果

xe1.xdf2.rchg    xe1.xdf1.rchg | xe2.xdf1.rchg    xe2.xdf2.rchg
    1               1          |     0               0
    0               0          |     1               1
                               |                     1
```

有了 diff 的结果，只需要从后往前遍历，找到最后相同行后连续的不同的行，作为一个块，将二者对应起来，就可以得到一个编辑脚本：

```c
// xdiff.c   xdl_build_script

int xdl_build_script(xdfenv_t *xe, xdchange_t **xscr) {		// 编辑脚本是一个xdchange_t结构的链表 编辑脚本的头指针   x_script 指针的指针
	xdchange_t *cscr = NULL, *xch;
	char *rchg1 = xe->xdf1.rchg, *rchg2 = xe->xdf2.rchg;
	long i1, i2, l1, l2;

	/*
	 * Trivial. Collects "groups" of changes and creates an edit script.
	 */
	for (i1 = xe->xdf1.nrec, i2 = xe->xdf2.nrec; i1 >= 0 || i2 >= 0; i1--, i2--)		// 从后往前, nrec 记录这个版本
		if (rchg1[i1 - 1] || rchg2[i2 - 1]) {		// 任一边有变化
			for (l1 = i1; rchg1[i1 - 1]; i1--);
			for (l2 = i2; rchg2[i2 - 1]; i2--);	// l1 l2 找到第一个相同的行的下一行  （收集所有连续的变更行）
			if (!(xch = xdl_add_change(cscr, i1, i2, l1 - i1, l2 - i2))) {		// 对于每一组变更，创建一个 xdchange_t 结构（通过xdl_add_change函数），加入链表头
				xdl_free_script(cscr);							// 这个结构包含了变更的起始行（i1，i2）以及涉及的行数（l1-i1, l2-i1）
				return -1;
			}
			cscr = xch;
		}

	*xscr = cscr;

	return 0;
}
```

接着 Git-merge 会双指针从后往前遍历两边的编辑脚本，如果两边的编辑脚本对应在 base 中的行有冲突，就会将这个块标记为冲突块，否则就是普通的编辑块直接接受。

问题就出在这个判断对应 base 行是否冲突的逻辑上，源码这里是

```c
if (xscr1->i1 + xscr1->chg1 < xscr2->i1)
```

就是说起始位置加上长度要小于结束位置才不算冲突，还是刚才那个例子，对应的编辑脚本是这样，左右两侧相等，于是认为是冲突块。

<img alt="编辑脚本示意" src="https://img.foril.space/编辑脚本示意.png" width=600px style="display: block; margin:10px auto"/>

也就是说 Git 倾向于把相邻行的编辑合并为一个大的冲突块，而不是直接应用相邻的编辑，个人认为这个原因是 Git 为了减少假阴性冲突的生成（自动消解但结果不正确的合并行为），会把同一个位置的编辑作为冲突块警示开发者，比如相邻块的编辑或是不确定顺序的在同一个位置的插入：

<img alt="相邻行编辑插入" src="https://img.foril.space/相邻行编辑插入.png" width=1000px style="display: block; margin:10px auto"/>

<img alt="同一位置的插入" src="https://img.foril.space/同一位置的插入.png" width=600px style="display: block; margin:10px auto"/>

## 总结

以上就是对于 Git-merge 合并时为什么没有直接接纳相邻行编辑这个现象的讨论，个人认为 Git 会在不确定拼接顺序或是不确定相邻行编辑采纳是否会造成编译冲突或是测试冲突的情况下，会把相邻的编辑划分为一个冲突块，交给开发人员去解决，这样做的目的是为了减少假阴性冲突的生成，但是这样做的代价就是会造成非常多的假阳性冲突，也就是说会把一些实际上可以直接接纳的编辑划分为冲突块，交给开发人员去解决，在实际解决冲突的过程中，如果能够细化编辑块中的编辑，将这种类型的假阳性冲突进行一个合并结果展示，那么就可以减少开发人员的工作量，一定程度上提高开发效率。

## 参考
* [tiarkrompf's note: The Histogram Diff Algorithm](https://tiarkrompf.github.io/notes/?/diff-algorithm/aside3)