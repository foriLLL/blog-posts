在工作中总是容易弄混 `reset` 和 `checkout` 各种用法的区别，《Pro Git》第 7.7 节对 `reset` 命令做了很详细的讲解，在这里做一个简单的学习记录。

## Git 中的三棵树
要更好地理解 `reset` 命令，需要先对如何通过管理“三棵树”来实现版本控制有所了解。

|树|用途 |
|--|----|
|HEAD|上一次提交的快照，下一次提交的父结点|
|Index|预期的下一次提交的快照|
|Working Directory| 沙盒|

简单来说就是 HEAD 指向提交历史中的最后一次提交，Index 可以理解为暂存区，`git add` 后会把工作区下的文件同步到暂存区，`git commit` 后会把暂存区的内容作为新的提交放入提交历史中，同时 HEAD 指向最新的提交。

![理解三棵树](https://git-scm.com/book/en/v2/images/reset-start.png)

## reset 如何操纵三棵树
### Step 1: 移动 HEAD
`reset` 做的第一件事是移动 HEAD 的指向。 这与改变 HEAD 自身不同（`checkout` 所做的）；`reset` 移动 HEAD 指向的分支。  
如果使用 `git reset --soft` 命令，那么操作将停止在这一步。
![](https://git-scm.com/book/en/v2/images/reset-soft.png)

### Step 2: 更新 Index
第二步中，git 会将 HEAD 指向的内容同步到 Index，这样的操作等价于取消了暂存，使我们回到了 `add` 和 `merge` 之前。  
如果使用 `git reset --mixed` 命令，那么操作将停止在这一步。这也是 `reset` 的默认效果。
![](https://git-scm.com/book/en/v2/images/reset-mixed.png)

### Step 3: 更新工作目录
第三步就是让工作目录看起来像暂存区，使用 `git reset --hard` 命令才会进行这一步操作。需要注意 `--hard` 会让你丢失工作区的所有修改，确保在之前做好记录或 `stash` 你的工作。

## 通过路径来 reset
如果给 `reset` 命令制定了一个文件路径或文件集，如 `git reset file.txt` （其实是 `git reset --mixed HEAD file.txt` 的简写形式），`reset` 将会跳过 Step 1，并且将它的作用范围限定为指定的文件或文件集合。因为 HEAD 只是一个指针，你无法让它同时指向两个提交中各自的一部分。 不过 Index 和工作目录可以部分更新，所以重置会继续进行第 2、3 步。所以**本质上只是将 file.txt 从 HEAD 复制到索引中**。相当于 **取消暂存文件** 的实际效果。

![](https://git-scm.com/book/en/v2/images/reset-path1.png)
当然这个命令也可以用于让暂存区和任意提交同步，如 `git reset eb43bf file.txt`。

## checkout 和 reset 之间的区别
和 `reset` 一样，`checkout` 也操纵三棵树，不过它有一点不同，这取决于你是否传给该命令一个文件路径。
### 不带路径
运行 `git checkout [branch]` 与运行 `git reset --hard [branch]` 非常相似，它会更新所有三棵树使其看起来像 `[branch]`，不过有两点重要的区别。
1. 首先不同于 `reset --hard`，`checkout` 对工作目录是安全的，它会通过检查来确保不会将已更改的文件弄丢。 其实它还更聪明一些。它会在工作目录中先试着简单合并一下，这样所有 还未修改过的 文件都会被更新。 而 `reset --hard` 则会不做检查就全面地替换所有东西。
2. 第二个重要的区别是 `checkout` 如何更新 HEAD。 `reset` 会移动 HEAD 分支的指向，而 `checkout` 只会移动 HEAD 自身来指向另一个分支。
   
![](https://git-scm.com/book/en/v2/images/reset-checkout.png)

### 带路径
运行 `checkout` 的另一种方式就是指定一个文件路径，这会像 `reset` 一样不会移动 HEAD。 它就像 `git reset [branch] file` 那样用该次提交中的那个文件来更新索引，但是它也会覆盖工作目录中对应的文件。 它就像是 `git reset --hard [branch] file`（如果 `reset` 允许你这样运行的话）， 这样对工作目录并不安全，它也不会移动 HEAD。

此外，同 `git reset` 和 `git add` 一样，`checkout` 也接受一个 --patch 选项，允许你根据选择一块一块地恢复文件内容。

## 参考
* https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E9%87%8D%E7%BD%AE%E6%8F%AD%E5%AF%86#_git_reset