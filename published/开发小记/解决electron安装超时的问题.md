在运行网上一些开源electron项目以及学习electron的过程中，发现无法正常下载electron，在网上搜索一番后找到了解决方案，以下加以记录。

在安装electron的时候，一直卡在node install.js最终超时，使用npm和yarn都有相同的问题，我遇到的问题是通过在 `.npmrc` 配置文件加入`electron_mirror`解决。

在安装过程中，electron 模块会通过 electron-download 为您的平台下载 Electron 的预编译二进制文件。查阅文档后发现可以配置国内镜像。

![electron-download](https://img.foril.fun/electron-download.webp)

Linux在`~/.npmrc`中修改  
Windows在`C:\Users\{username}\.npmrc`中修改
```
registry=https://registry.npm.taobao.org/
electron_mirror="https://npm.taobao.org/mirrors/electron/"
```

## 参考
[解决安装electron卡在node install.js 不动问题](https://www.jianshu.com/p/28a0305ac187)