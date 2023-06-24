---
description: "Git Rerere 可以记录冲突解决的过程，当遇到相同的冲突时，自动应用之前记录的解决方法，而不需要手动再次解决。这样可以大大简化开发过程，提高开发效率。本文记录其基本用法。"
time: 2023-04-12
---

在解决一些科研问题是偶然接触到 `git-rerere` （Reuse Recorded Resolution）这个工具，就像它的名字所说，它能够帮助我们在合并遇到冲突时，通过复用之前的 resolution，来解决一些之前遇到过的 **一模一样** 的合并冲突。其实这是一个非常常见的场景，在使用相对较长的主题分支的工作流中，开发人员有时需要一遍又一遍解决相同的冲突。

Git Rerere 可以记录冲突解决的过程，当遇到相同的冲突时，自动应用之前记录的解决方法，而不需要手动再次解决。这样可以大大简化开发过程，提高开发效率。

## Git Rerere 的基本用法

### 启用 Git Rerere  

在使用 Git Rerere 之前，需要在 Git 中启用这个功能：

```sh
git config [--global] rerere.enabled true
```

### 具体使用

这里我们以一个例子来展示 Git Rerere 的使用方法。假设我们现在有两个分支，分别是 `main` 和 `b1`，两个分支上都有一个文件 `hello.txt`，在 `main` 中他的内容是：

```
hello 世界
```

而在 `b1` 中他的内容是：

```
你好 world
```

现在我们需要将 `b1` 合并到 `main` 中，这时候 Git 会提示我们有冲突，已经记录了一个 preimage，我们可以通过 `git rerere status` 来查看：

<img alt="20230331100935" src="https://img.foril.fun/20230331100935.png" width=300px style="displat: block; margin:10px auto"/>

这里我们可以看到，Git 已经记录了一个 preimage，我们可以通过 `git rerere diff` 来查看这个 preimage 的具体内容：

<img alt="20230331101000" src="https://img.foril.fun/20230331101000.png" width=300px style="displat: block; margin:10px auto"/>

我们可以看到，这个 preimage 的内容就是我们的冲突文件（如果不想记录这个文件的冲突，可以通过 `git rerere clear` 来清除这个 preimage）。

接下来我们手动解决这个冲突，将 `hello.txt` 的内容修改为：

```
你好 世界
```

然后我们再次执行 `git rerere diff`，可以看到，Git 已经记录了一个 resolution：

<img alt="20230331101248" src="https://img.foril.fun/20230331101248.png" width=300px style="displat: block; margin:10px auto"/>

在这个 resolution 中，Git Rerere 记录了解决冲突的方案，也就是删除了冲突的部分，然后将 `hello.txt` 的内容修改为：`你好 世界`。接着我们就可以提交这个修改了。提交后就可以复用这个 resolution。

### Living a peaceful life with Git Rerere

之后，当我们再次遇到相同的冲突时，Git Rerere 就会自动应用之前记录的解决方法，而不需要手动再次解决。这样可以大大简化开发过程，提高开发效率。

<img alt="20230331102023" src="https://img.foril.fun/20230331102023.png" width=300px style="displat: block; margin:10px auto"/>