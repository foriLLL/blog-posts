---
description: "前一直没有注意过 Linux 切换身份的命令的不同，也没有注意过不同用户的环境变量问题。以为 su xxx 和 su - xxx 是同一个方法，直到有一次在运行 hadoop 时，发现找不到命令。"
time: 2021-05-08 14:00:32+08:00
---

之前一直没有注意过 Linux 切换身份的命令的不同，也没有注意过不同用户的环境变量问题。以为 `su xxx` 和 `su - xxx` 是同一个方法，直到有一次在运行 hadoop 时，发现找不到命令。明明记得在环境变量里加入过，但是 `echo` 也没打印出来，~~我一脸懵逼~~。后来在排除下发现是并没有在全局配置环境变量，而是配在了当时用户的环境变量下，后来也没有搞明白 `su` 和 `su - ` 不同用法，导致了问题的产生。  
所以在理解这两个命令的区别之前，应该首先搞明白另一个概念。  

## Linux环境变量  

Linux 是一个多用户多任务的操作系统，所以像我们在 Windows 中配置环境变量可以选择 *全局变量* 和 *用户变量* 一样，Linux 也有不同上下文的环境变量。  

### 环境变量的分类  

* 按照生命周期来分：  
1. **永久的**：需要用户修改相关的配置文件，变量永久生效。
2. **临时的**：用户利用export命令，在当前终端下声明环境变量，关闭Shell终端失效。

<br />

* 按照作用域来分：
1. **系统环境变量**：系统环境变量对该系统中所有用户都有效。
2. **用户环境变量**：顾名思义，这种类型的环境变量只对特定的用户有效。

### 设置环境变量  

* 在/etc/profile文件中添加变量 对所有用户生效（**永久的**）

* 在用户目录下的.bash_profile文件中增加变量 【对单一用户生效（**永久的**）】

* 直接运行export命令定义变量 【只对当前shell（BASH）有效（**临时的**）】

这里还有一篇讲解很详细的参考文章：[Linux环境下不同脚本文件配置的环境变量作用域范围的区别](https://blog.csdn.net/highfly591/article/details/42497007)  

<hr />

在解决了以上的问题后，来到我们的下一个话题：`su xxx`和`su - xxx`的区别。  

先说结论，su命令和su -命令最大的本质区别就是：
- 前者只是切换了root身份，<u>但Shell环境仍然是普通用户的Shell</u>；
- 而后者连用户和Shell环境一起切换成root身份了。只有切换了Shell环境才不会出现PATH环境变量错误。

我们查看su的帮助文档可以发现：

```bash
 -, -l, --login                  make the shell a login shell
```

**区别就是**  
login shell：此种方式登录时，shell会重新读取/etc/profile和~/.bash_profile来应用新的环境变量。

non-login shell：此时shell不会读取 `/etc/profile` 和 `~/.bash_profile`，而是读取 `~/.bashrc` 来应用新的环境变量。

到这里，我们就解决了我们的问题，区分了`su xxx`和`su - xxx`的区别。


## 参考

- [1] [简书：Linux环境变量总结](https://www.jianshu.com/p/ac2bc0ad3d74)

- [2] [CSDN：login shell和non-login shell](https://blog.csdn.net/zzzhktk/article/details/8221133)

- [2] [CSDN：su 和su -的区别](https://blog.csdn.net/nayanminxing/article/details/76424115)
