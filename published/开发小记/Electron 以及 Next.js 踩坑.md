---
description: "最近毕设准备采用 Electron 将 Web 项目迁移到本地应用，Next.js 相比 CRA 封装了很多开箱即用功能，但在学习过程中还是踩了一些坑，在这里做一个记录。"
time: 2022-04-12
heroImage: "https://img.foril.space/20230622181312.png"
tags: []
---

最近毕设准备采用 Electron 将 Web 项目迁移到本地应用，Next.js 相比 CRA 封装了很多开箱即用功能，但在学习过程中还是踩了一些坑，以下做一个记录。

当我想利用 `next export` 生成静态文件并用 Electron 直接 `loadFile()` 来显示页面时，发现生成的页面没有任何样式和 js 脚本，通过 Chrome 开发者工具检查发现生成的资源路径全部都是 `/_next/...` 这种形式。

<img src="https://img.foril.space/next_export_bug.png" width=600px style="margin:10px auto"/>
<img src="https://img.foril.space/next_export_bug2.png" width=600px style="margin:10px auto"/>

在网上搜索解决方案后发现这个问题早在2019年就有在社区[讨论](https://github.com/vercel/next.js/issues/8158)过，主要出现在不使用服务器而是直接单独使用 html。

根据官网介绍，`next export` 允许你将 Next.js 应用程序导出到静态 HTML，这样可以独立运行，而**不需要 Node.js 服务器**。  
`next export` 构建了一个 HTML 版本的应用程序。在 `next build` 过程中，`getStaticProps` 和 `getStaticPaths` 将为页面目录中的每个页面生成一个 HTML 文件(动态路径会更多)。然后，`next export`将把已经导出的文件 **拷贝** 到正确的目录中。

也许由于是面向服务器的原因，所有的静态文件资源路径都是`/_next/...` 的形式，导致的问题就是生成的静态文件用于 Electron 时所有资源请求失败。

## 思考

Next.js 官方并没有给出网上的解决方案，大概是通过 Next.js 等服务器没有这样的根路径问题，在企图与 Electron 配合 `loadFile` 使用时就会出现<u>请求地址并不是对于操作系统而言的相对路径而是绝对路径</u>。这一点我们也可以通过使用 `serve` 来验证。

```bash
# yarn global add serve
serve ./out/
```

启动 serve 服务，这时候打开对应端口，我们可以发现请求地址变为了类似 `http://localhost:3000/_next/static/chunks/webpack-cb7634a8b6194820.js` 这样的地址，对于这样的地址，服务器就可以正确匹配到静态资源（而Electron 无法通过本地资源在根目录下找到匹配），则可以正确加载资源。

所以接下来如果想要使用 Electron 和 Next.js 结合，你可以：

1. 手动将生成的所有 `html` 文件中的资源路径修改，加入 `.`（不推荐）
2. 在 `next.config.js` 中加入
   ```js
   module.exports = {
     assetPrefix: ".",
   };
   ```
   可以在生成的资源路径中自动加入前缀`.`
3. 继续使用 Next.js 服务器，转而在 `Electron` 中使用 `loadURL` 解决问题。

## 注意
同时需要注意的一点是 `electron export` 后不能使用 `API Routes`，这一点在[官方文档](https://nextjs.org/docs/api-routes/introduction#caveats)中也有提及。

## 参考
* https://simonallen.coderbridge.io/2021/07/15/nextjs-export-static/
* https://github.com/vercel/next.js/issues/8158
* https://nextjs.org/docs/advanced-features/static-html-export
* https://nextjs.org/docs/api-routes/introduction