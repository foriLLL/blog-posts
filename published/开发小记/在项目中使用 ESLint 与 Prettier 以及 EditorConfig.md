---
description: "最近在前端项目编写过程中为了统一代码风格以及实现编辑器中对代码风格的提示，尝试了linter和Prettier等工具，在应用过程中发现了很多没明白的配置，花了很长时间才有了一个大概理解。"
time: 2022-03-28 13:26:41+08:00
heroImage: ""
tags: []
---
最近在前端项目编写过程中为了统一代码风格以及实现编辑器中对代码风格的提示，尝试了 linter 和 Prettier 等工具，在应用过程中发现了很多没明白的配置，花了很长时间才有了一个大概理解。

## 他们都是什么？
### ESLint
<img src="https://d33wubrfki0l68.cloudfront.net/204482ca413433c80cd14fe369e2181dd97a2a40/092e2/assets/img/logo.svg" width=80px style="float:right" >

是一个开源的 JavaScript 的 linting 工具，使用 espree 将 JavaScript 代码解析成**抽象语法树(AST)**，然后通过AST来**静态分析**我们的代码，从而给予我们两种提示：  
1. 代码质量问题：使用方式有可能有问题(problematic patterns)
2. 代码风格问题：风格不符合一定规则 (doesn’t adhere to certain style guidelines)

许多大公司都有自己关于js项目的ESLint代码标准，认可度比较高的是[Airbnb的标准](https://www.npmjs.com/package/eslint-config-airbnb)，你可以通过`yarn add -D eslint-config-airbnb`，然后在`.eslintrc.json`加入：`"extends": ["airbnb"]`（可省略`eslint-config-`）来完成对于代码风格的配置。

### Prettier
<img src="https://prettier.io/icon.png" width=80px style="float:right" />

Prettier 的格式化代码的功能与 ESLint 的非常相似，但是它并**不检查代码质量问题**。它只是作为一个代码格式化工具（Code Formatter），是 js **特有的**格式化工具，里面很多配置项是 js 这门语言特有的规范。对于它原生支持的 JavaScript 而言，它做的非常好。然而，同时它也支持 JSX、Flow、TypeScript、HTML、JSON、CSS 等其他众多语言。

### EditorConfig
<img src="https://editorconfig.org/logo.png" width=80px style="float:right" />

EditorConfig 既不检测也不格式化你的代码。它仅仅在开发者团队内部使用的所有 IDE 和编辑器之间定义了一份标准的代码风格指南（**提供一个统一的设置和标准，而且通过各个主流编辑器和 IDE 可以自动同步设置，在不同的文件自动更改尾行配置或者是缩进编码等问题**）。比如，一个团队主要使用 Sublime Text 和 VSCode，EditorConfig 能够使得它们在单个文件内定义公共的缩进模式（空格或制表符）。  
EditorConfig 配置的是比较基础的东西，基本上你用编辑器本身能干的操作就可以用它来干。比如 Tab 变几个空格啊、换行符是 CR 还是 CRLF 啊、文件编码是不是 UTF-8 啊这种问题。  
而且它不只局限于格式化，名字也能看出来是“Config”（配置）而不是“Formatter”（格式化器）。你也可以用它来配置诸如让 IDE 忽略特定的编译警告错误之类的。

所以你会发现它跟编程语言本身没什么关系，各个语言的项目都能看到 `.editorconfig` 的身影，它更多地干的是当你用特定 IDE 时能配置的那些东西，好让那些不用这个 IDE 的、或者它 IDE 配置跟你不一样的开发者也能使用相同的编辑器方案。当然了它确实可以通过插件的形式去支持一些其他语言特有的格式化方案，不过并不常用。

## 为什么我要用三者，其中一个不行吗？
上文所说，ESLint 能够应用**代码质量规则**和**格式化规则**这两种规则，如果可以的话，它还能自动修复代码。另一方面，Prettier 只能检查代码中的格式错误，但是在这一方面上，它却比 ESLint 更专业。

因此，为了在代码质量和格式化方面达到最佳检测体验，你确实应该同时使用这两个工具。

至于有关 EditorConfig 和 Prettier 的第二个灵魂拷问，答案很简单，它们谁都替代不了对方。**EditorConfig 的作用是配置你的编辑器，以便你所编写的代码已经是格式良好的了，而 Prettier 要做的则是格式化你已经编写的代**。与 Prettier 相比，EditorConfig 可用于更多的语言和项目。

***
所以总结来说，虽然这三者的目的趋同，但是它们仍是**术业有专攻**的。

### 解决模式
所以我们想使用 *专事专办* 的方法。在这种情况下，ESLint 成为了我们的代码质量检测器，Prettier 充当代码格式化工具，而 EditorConfig 将为每个人提供正确的编辑器配置。

* 与编辑器相关的所有配置（结尾行、缩进风格、缩进大小等等）应该由 EditorConfig 来处理。
* 和代码格式相关的一切事物应该由 Prettier 处理。
* 剩下的（代码质量）则由 ESLint 负责。

## 如何友好使用三者结合
项目中我采用了next.js作为开发框架，项目中默认配置了ESLint，提供一个开箱即用的ESLint体验，默认下载`eslint`和`eslint-config-next`作为项目开发依赖，只要你在`.eslintrc.json`中配置extend一个rule set，就可以完成需要的规则集合的配置。  

> Prerequisite：
> * 本地或全局安装 Prettier 和 eslint，npm 安装了 Prettier 才能使用 `npm run lint` 命令来检查代码格式
> * 编辑器支持 EditorConfig 或安装插件

想要加入 Prettier， 就需要我们首先停用可能与 Prettier 冲突的所有 ESLint 规则（仅指代码格式规则）。幸运地是，`eslint-config-prettier` 包已经帮我们做了这件事。
```bash
yarn add -D eslint-config-prettier
```
接下来我们重写 `.eslintrc.json` 文件，把 prettier 添加到 extends 数组中，并移除我们全部已有的代码格式规则：
```json
{
  "extends": ["eslint:recommended", "prettier"],
  "env": {
    "es6": true,
    "node": true
  }
}
```
prettier 配置会覆盖先前 extends 数组中一些配置，禁掉所有 ESLint 的代码格式规则。有了这个配置，Prettier 和 ESLint 就可以相安无事地**独自运行**了。这意味着我们需要运行 `npx eslint main.js` 和 `npx prettier main.js --write` 这样的两个命令来检测与格式化文件，这样非常不方便，我们将通过添加 eslint-plugin-prettier 包来让 ESLint 集成 Prettier。
```bash
yarn add -D eslint-plugin-prettier
```
接着重写 `.eslintrc.json` 文件，在 plugins 数组中添加 prettier 插件，并设置新建的 prettier 规则为 error，从而达到 prettier 格式错误被当作 ESLint 错误的效果。
```json
{
  "extends": ["eslint:recommended", "prettier"],
  "env": {
    "es6": true,
    "node": true
  },
    "plugins": [
    "prettier"
  ],
  "rules": {
    "prettier/prettier": "error"
  }
}
```
接下来就可以通过`npx eslint main.js`得到输出了，输出会将prettier标记的格式错误通过ESLint输出，所有的问题一次性全部输出。

`eslint-plugin-prettier` 提供了这样一个配置 plugin:prettier/recommended，具体内容如下:
> 注意：使用 eslint-plugin-prettier 和 --fix 时开启 arrow-body-style & prefer-arrow-callback 可能会导致 issue，因此建议关闭这两条规则。
```json
{
  "extends": ["prettier"],
  "plugins": ["prettier"],
  "rules": {
    "prettier/prettier": "error",
    "arrow-body-style": "off", 
    "prefer-arrow-callback": "off"
  }
}
```

因此我们的 .eslintrc.js 可以直接简化为：
```json
{
  "extends": ["plugin:prettier/recommended"]
}
```
这样配置结束后，你就可以在 Prettier 的配置文件 `.prettierrc.json` 中配置你想要的代码格式化规则了：
```json
{
  "semi": true
}
```
这样我们每次写完代码格式化都可以将代码按照配置的规范进行优化，但每次都需要将设置编辑器中空格数量，尾行配置，编码等代码风格设置重新配置。

这时我们就需要引入 EditorConfig，让我们无论使用什么编辑器都可以拥有相同的编辑器配置（VS Code需要安装[插件](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig)，可以根据文件类型自动更新编辑器配置，这一点很有用）。  
这也意味着 Prettier 和 EditorConfig 共享一些配置项，我们需要**避免在两个单独的文件中重复这些配置项**，并让二者保持同步（比如：尾行配置）。  
***Prettier 的最新版本通过解析 .editorconfig 文件来确定要使用的配置选项解决了此问题。***

这些选项仅限于：
```sh
# .editorconfig
end_of_line
indent_style
indent_size/tab_width
max_line_length
```
这些配置选项将覆盖以下 Prettier 选项（**如果未在 `.prettierrc` 中定义它们**）：
```json
// .prettierrc
"endOfLine"
"useTabs"
"tabWidth"
"printWidth"
```
与 ESLint 和 Prettier 一样，如果要更改配置，规则就是检查它是否是 EditorConfig 或 Prettier 的相关配置，然后在适当的文件中进行更改。这 4 个选项之外的先前配置选项仅只写到 `.editorconfig` 中。

配置完成后，与代码风格相关的简单配置就可以直接在 `.editorconfig` 中配置，与 js 代码风格相关的一些配置（如行末分号等）可以在 `.prettierrc.json` 中修改，其他与代码质量息息相关的复杂问题配置可在 `.eslintrc` 中通过配置规则集合或手动覆盖规则。  

有了三个工具的结合，确实可以在很大程度上提高你的开发体验。

> 注意要将代码编辑器对于 .js 等文件的默认格式化工具设置为 Prettier，否则 Prettierrc 中的配置将不会生效。

接下来给出一个我自己实际在 Next.js 项目开发中的所用到的配置，以便之后直接复用：
```json
// .prettierrc
{
    "semi": false,
    "singleQuote": true,
    "trailingComma": "all",
    "arrowParens": "avoid",
    "printWidth": 80,
    "jsxSingleQuote": false,
    "jsxBracketSameLine": false
}
```
```json
// .eslintrc.json
{
    "extends": ["next/core-web-vitals", "plugin:prettier/recommended"]
}
```
```conf
# .editorconfig
root = true

[*]
charset = utf-8
indent_style = space
indent_size = 4
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.md]
max_line_length = off
trim_trailing_whitespace = false
```


## 参考

[为什么你要用ESLint，Prettier和EditorConfig](https://juejin.cn/post/6924249306039844872)

[无冲突设置ESLint，Prettier和EditorConfig](https://juejin.cn/post/6924546232945737742)

[ Prettier 和 EditorConfig](https://segmentfault.com/q/1010000040744932)

[搞懂 ESLint 和 Prettier](https://zhuanlan.zhihu.com/p/80574300)