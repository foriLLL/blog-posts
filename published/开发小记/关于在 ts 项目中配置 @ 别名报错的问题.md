---
description: "由于 tsc 并不会在 emitted code 中解决 import 的路径别名（这个问题已经在社区有很多讨论了，官方的意思是这就是期望的输出），所以我们需要额外加入一些配置来完成别名的配置，这里记录在 ts 项目中配置 \"@/*\" 别名报错的问题的三种解决方案。"
time: 2023-01-10T15:44:28+08:00
heroImage: "https://img.foril.fun/20230622201156.png"
tags: []
---
由于 tsc 并不会在 emitted code 中解决 import 的路径别名（这个问题已经在社区有很多 [讨论](https://github.com/microsoft/TypeScript/issues/10866) 了，官方的意思是这就是期望的输出），所以我们需要额外加入一些配置来完成别名的配置，这里记录在 ts 项目中配置 "@/*" 别名报错的问题的三种解决方案。

这个问题是我在开发 Koa 博客后端时遇到的，采用 TypeScript 开发。
在开发时想监听文件变化，自动重启服务，于是安装了 nodemon 和 ts-node，然后使用 ts-node 运行 ts 文件，具体配置如下：
```json
// nodemon.json
{
    "watch": [
        "src"
    ],
    "ext": "ts,js",
    "ignore": [],
    "exec": "ts-node -r tsconfig-paths/register ./src/application.ts"
}
```

在最初的开发中没有遇到问题，但是接下来我想给模块加入别名，于是对 ts 加入了配置：
```json
// tsconfig.json
{
    "compilerOptions": {
        "baseUrl": ".", /* Specify the base directory to resolve non-relative module names. */
        "paths": {
            "@/*": [
                "src/*"
            ] /* Specify a set of entries that re-map imports to additional lookup locations. */
        }, /* Specify a set of entries that re-map imports to additional lookup locations. */
    }
}
```
以及对 babel 加入了配置（这个问题跟 babel 关系不大，但这里都做一个记录）：
```json
// babel.config.json
{
    "plugins": [
        [
            "module-resolver",
            {
                "root": [
                    "./"
                ],
                "alias": {
                    "@": "./src"
                }
            }
        ]
    ],
    "presets": [
        [
            "@babel/preset-env",
            {
                "targets": {
                    "node": "current"
                },
                "useBuiltIns": "usage",
                "corejs": "3.6.5"
            }
        ],
        "@babel/preset-typescript"
    ]
}
```
接着在运行 ts-node 时，收到了报错：
```shell
> nodemon

[nodemon] 2.0.22
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): src/**/*
[nodemon] watching extensions: ts,js
[nodemon] starting `ts-node ./src/application.ts`
Error: Cannot find module '@/service/articleInfoService'
Require stack:
- /Users/foril/projects/foril_blog/blog_v2/koa_backend/src/routes/articleInfoRouter.ts
- /Users/foril/projects/foril_blog/blog_v2/koa_backend/src/application.ts
    at Function.Module._resolveFilename (node:internal/modules/cjs/loader:1047:15)
    at Function.Module._resolveFilename.sharedData.moduleResolveFilenameHook.installedValue [as _resolveFilename] (/Users/foril/projects/foril_blog/blog_v2/koa_backend/node_modules/.pnpm/registry.npmmirror.com+@cspotcode+source-map-support@0.8.1/node_modules/@cspotcode/source-map-support/source-map-support.js:811:30)
    at Function.Module._load (node:internal/modules/cjs/loader:893:27)
    at Module.require (node:internal/modules/cjs/loader:1113:19)
    at require (node:internal/modules/cjs/helpers:103:18)
    at Object.<anonymous> (/Users/foril/projects/foril_blog/blog_v2/koa_backend/src/routes/articleInfoRouter.ts:4:1)
    at Module._compile (node:internal/modules/cjs/loader:1226:14)
    at Module.m._compile (/Users/foril/projects/foril_blog/blog_v2/koa_backend/node_modules/.pnpm/registry.npmmirror.com+ts-node@10.9.1_@types+node@20.1.1_typescript@4.9.5/node_modules/ts-node/src/index.ts:1618:23)
    at Module._extensions..js (node:internal/modules/cjs/loader:1280:10)
    at Object.require.extensions.<computed> [as .ts] (/Users/foril/projects/foril_blog/blog_v2/koa_backend/node_modules/.pnpm/registry.npmmirror.com+ts-node@10.9.1_@types+node@20.1.1_typescript@4.9.5/node_modules/ts-node/src/index.ts:1621:12) {
  code: 'MODULE_NOT_FOUND',
  requireStack: [
    '/Users/foril/projects/foril_blog/blog_v2/koa_backend/src/routes/articleInfoRouter.ts',
    '/Users/foril/projects/foril_blog/blog_v2/koa_backend/src/application.ts'
  ]
}
[nodemon] app crashed - waiting for file changes before starting...
```

之后在搜索了一段时间后，发现了问题应该出在 TypeScript 和 Babel 的别名配置仅在编译时生效，而不会影响 Node.js 在运行时的模块解析。使用 ts-node 运行代码无法找到别名路径，所以报错。

这个问题可以使用 `tsconfig-paths` 来解决，这个包周下载量大概有 2400w，估计是比较常用的一个包了。
具体配置如下：
```json
{
    "watch": [
        "src"
    ],
    "ext": "ts,js",
    "ignore": [],
    "exec": "ts-node -r tsconfig-paths/register ./src/application.ts"
}
```

接着运行，问题完美解决。

> 附上 tsconfig-paths 的官方介绍翻译：  
> 
> 使用此包加载在 tsconfig.json 或 jsconfig.json 的 paths 部分指定的位置的模块。支持在运行时和通过 API 加载。  
默认情况下，TypeScript 模拟 Node.js 运行时的模块解析策略。但它还允许使用路径映射，这允许指定任意模块路径（不以 "/" 或 "." 开头的路径），并将它们映射到文件系统中的物理路径。**TypeScript 编译器可以从 tsconfig 解析这些路径，因此它将编译正常。但是，如果您尝试使用 node（或 ts-node）执行编译后的文件，它将仅在直到文件系统根目录的 node_modules 文件夹中查找，因此不会找到 tsconfig 中的路径所指定的模块。**  
如果您需要此包的 tsconfig-paths/register 模块，它将从 tsconfig.json 或 jsconfig.json 中读取路径并将 node 的模块加载调用转换为 node 可加载的物理文件路径。

***
## 补充
之后发现，使用 tsconfig-paths 之后，ts-node 确实可以在运行时找到别名路径，但是如果不使用 babel，而是直接使用 tsc 编译时，却找不到别名路径，可以生成 js 文件，但是运行时会报错。
```js
// 这里是生成的 .js 文件，可以看到别名路径并没有被替换
const file_1 = require("@/utils/file");
const config_1 = require("@/config");
```

那么如果我不想使用babel，而是直接使用 tsc 编译，应该怎么办呢？

### 方法一
最大胆的思路当然是自己写一个插件，检查 `tsconfig.json` 有没有配置别名，如果有的话，再检查 `baseUrl`，检查 `paths`，然后替换掉所有出现的别名路径，这样就可以了。

当然，也有现成的包帮我们解决了这个问题，比如 `tsc-alias`，他会在 `tsc` 生成 js 文件后检查生成的 js 文件，然后替换掉别名路径，这样就可以了。（目前 tsc-alias 还不支持运行时使用，也就是不能和 ts-node 搭配使用，详见 [这里](https://github.com/justkey007/tsc-alias/issues/81)）

安装
```sh
pnpm add -D tsc-alias
```
运行
```sh
tsc && ts-alias
```
如果只单独运行 tsc 是不会替换别名路径的，需要再运行一次 `ts-alias`。

那么现在我们的工作流程就变成了使用tsc编译时使用 `tsc && tsc-alias`，运行时使用 `ts-node -r tsconfig-paths/register ./src/application.ts`。需要用到两个第三方包，**生成的 js 文件可以直接使用 Node 运行的**。

### 方法二
另外一种解决方案是在 Node 中使用 `tsconfig-paths` 提供的注册模块，利用 
```sh
TS_NODE_BASEURL=./build_tsc node -r tsconfig-paths/register main.js
```
命令来给 Node 注册模块，这样就可以直接使用 `node` 来运行带有别名的js文件了。

同时由于 `tsconfig-paths` 只提供了 `baseUrl` 环境变量的修改，所以我们在 `tsconfig.json` 中的 `baseUrl` 也需要修改为能够被环境变量替换的值，比如在这里，我们将 `baseUrl` 修改为 `./src`，然后在运行时使用 `TS_NODE_BASEURL=./build_tsc` 来替换环境变量，就可以让 `tsconfig-paths` 在运行时找到 `@/*` 正确的路径了。

```json
// tsconfig.json
{
    "compilerOptions": {
        "baseUrl": "src",   // 这里进行了修改
        "paths": {
            "@/*": [
                "*"
            ]
        }, 
    }
}
```
<img alt="20230510121544" src="https://img.foril.fun/20230510121544.png" width=300px style="displat: block; margin:10px auto"/>

***这里需要特别注意这种方法生成的 js 文件中是包含别名信息的，不能直接使用 node 运行，必须注册 `tsconfig-paths` 模块，这和上面使用 `tsc-alias` 直接将 js 中的别名替换的方法不同，`tsc-alias` 生成的 js 文件可以直接使用 node 运行。***

在这里附上两种方法的区别：
```js
// 使用 tsc-alias 生成的 js 文件
const fs_1 = __importDefault(require("fs"));
const file_1 = require("../utils/file");
const config_1 = require("../config");
const path_1 = __importDefault(require("path"));
const gray_matter_1 = __importDefault(require("gray-matter"));
```

```js
// 使用 tsconfig-paths 运行时的 js 文件
const fs_1 = __importDefault(require("fs"));
const file_1 = require("@/utils/file");     // 区别在这里 
const config_1 = require("@/config");       // 区别在这里
const path_1 = __importDefault(require("path"));
const gray_matter_1 = __importDefault(require("gray-matter"));
```

> node -r命令是用于在Node.js程序中加载一个或多个指定的模块。这个-r选项允许你在程序开始运行之前注册一个或多个模块，这些模块将被预先加载到内存中，以便在程序执行期间可以使用它们。

### 方法三
由于编译时 tsconfig-paths 失效是因为 [找不到 ourDir](https://github.com/dividab/tsconfig-paths/issues/75)，如果不指定 ourDir 就可以正常使用 `node -r tsconfig-paths/register ./src/application.js`，但是这样就不能指定输出文件夹了。

所以一个 workaround 就是针对 node 可以单独写一个 register，然后在运行时使用 `node -r ./register.js ./src/application.js`，这样就可以了。当然 register.js 的位置你可以自己指定。

```js
// register.js
const tsConfig = require('./tsconfig.json');
const tsConfigPaths = require('tsconfig-paths');

const { baseUrl, outDir, paths } = tsConfig.compilerOptions;

const outDirPaths = Object.entries(paths).reduce(
  (outDirPaths, [k, v]) => Object.assign(
    outDirPaths,
    { [k]: v.map(path => path.replace(/^src\//, `${outDir}/`)) }
  ),
  {}
);

tsConfigPaths.register({ baseUrl, paths: outDirPaths });
```
<img alt="20230510144821" src="https://img.foril.fun/20230510144821.png" width=600px style="displat: block; margin:10px auto"/>

## 参考
* https://www.npmjs.com/package/tsconfig-paths
* https://stackoverflow.com/questions/59179787/tsc-doesnt-compile-alias-paths
* https://github.com/dividab/tsconfig-paths/issues/75
* https://github.com/microsoft/TypeScript/issues/10866