---
description: "记录一次使用两种方式 squash commit 的过程。"
time: 2023-10-19T14:49:47+08:00
heroImage: "https://img.foril.fun/squash logo.png"
tags: []
---

squash 功能可以将多个 commit 合并成一个，这样可以使得 commit history 更加清晰。今天在简单学习使用 `git rebase -i` 的过程中，我就使用了这个功能，以下是我记录的一些过程。

## 方法一：使用 git rebase -i 来 squash commit

之前只是知道 `rebase` 有 interactive 的功能，但是一直没有用过，今天在合并分支时，想要将一些 commit 合并成一个，于是就想到了这个功能，于是就尝试了一下。

以下内容是一个 mock 出来的 Git 仓库内容。

<img alt="mock 仓库 git graph" src="https://img.foril.fun/mock 仓库 git graph.png" width=200px style="display: block; margin:10px auto"/>

接下来我想把 `add 45` 和 `add 67` 合并成一个 commit，于是我就在 branch a 上执行了 `git rebase -i HEAD~2`，然后就进入了一个交互式的编辑界面（vim）。

```sh
pick 9f5ca8a add 45
pick dd6aeb5 add 67

# Rebase a8624c6..dd6aeb5 onto a8624c6 (2 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
#                    commit's log message, unless -C is used, in which case
#                    keep only this commit's message; -c is same as -C but
#                    opens the editor
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
#         create a merge commit using the original merge commit's
#         message (or the oneline, if no original merge commit was
#         specified); use -c <commit> to reword the commit message
# u, update-ref <ref> = track a placeholder for the <ref> to be updated
#                       to this position in the new commits. The <ref> is
-- INSERT --
```

这里有两个 commit，我想要合并，所以我将第二个 commit 的 `pick` 改成了 `squash`，然后保存退出。

```sh
pick 9f5ca8a add 45
squash dd6aeb5 add 67
```

> 如果你将一行或连续多行的 `pick` 改成了 `squash`，那么这些 commit 就会被合并成一个 commit，和第一个 `squash` 上面的 commit 合并，然后会进入一个编辑界面，让你编辑 commit message。

接着编辑新的 commit message：

```sh
add 4567        # 在这里编辑 commit message

# This is a combination of 2 commits.
# This is the 1st commit message:

add 45

# This is the commit message #2:

add 67

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Thu Oct 19 14:57:03 2023 +0800
# 
# interactive rebase in progress; onto a8624c6
# Last commands done (2 commands done):
#    pick 9f5ca8a add 45
#    squash dd6aeb5 add 67
# No commands remaining.
# You are currently rebasing branch 'a' on 'a8624c6'.
#
# Changes to be committed:
#       modified:   1.txt
```

<img alt="after squash" src="https://img.foril.fun/after squash.png" width=250px style="display: block; margin:10px auto"/>

至此，我就完成了合并 commit 的操作。接下来，使用 `git rebase main` 即可将分支 a 上的 commit 合并到 main 分支上。

<img alt="after rebase" src="https://img.foril.fun/after rebase.png" width=250px style="display: block; margin:10px auto"/>

这样一整套流程的好处是，可以将一些不必要的 commit 合并成一个，从而使得 commit history 更加清晰。但代价就是会将之前在 a 上详细的提交过程合并成一个 commit，这样就会丢失一些细节。

## 方法二：使用 git merge --squash 来 squash commit

在上面这个例子中，我们在合并 a 时可以直接使用 `git merge --squash a` 来将 a 分支上的 commit 合并到 main 分支上，这样就不会丢失之前的提交历史了，但这样做的缺点是很难将新的提交和 a 中具体的 commit 对应起来，因为这些 commit 已经被合并成一个了。

<img alt="merge squash" src="https://img.foril.fun/merge squash.png" width=300px style="display: block; margin:10px auto"/>

## 参考

[git-tower: how to squash commits in Git](https://www.git-tower.com/learn/git/faq/git-squash)