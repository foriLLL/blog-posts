---
description: "装饰器（Decorator）是一种与类相关的语法，用来注释或修改类和类方法。很多面向对象语言都有装饰器这种语法，本文记录在 JavaScript 中装饰器的使用。"
time: 2021-09-11
heroImage: ""
tags: []
---

装饰器（Decorator）是一种与类相关的语法，用来注释或修改类和类方法。很多面向对象语言都有装饰器这种语法，值得注意的是，js 并不是严格意义上的面向对象语言，称它是一种 “*基于对象的语言*” 应该更合适。在 es5 中，并没有太多对于装饰器的需求，但在 es6 中，随着 class 作为一种语法糖的出现，装饰器的便利性得到了体现，逐渐变得流行。

## 装饰器的本质

装饰器实质是上就是一段功能函数，放在类和类方法的定义前面，但不同于普通函数的是，它的**执行是在代码的编译阶段**，而普通函数的执行需要等到代码运行时，所以装饰器的实质是在**对编程语言进行编程**，属于元编程的一种思想。

```js
//使用示意
@decorator_function
class ClassName{
    ...
}
```

## 装饰器的配置

目前装饰器[提案](https://github.com/tc39/proposal-decorators)处在 Stage 2[2]，node 以及各浏览器均不支持装饰器的语法，所以我们需要使用 babel 插件 `@babel/plugin-proposal-decorators` 来转换代码。

### 配置webpack

如果使用 webpack 来打包项目，通过简单配置 webpack 就可以使项目支持装饰器语法。  

首先下载 webpack

```bash
yarn add -D webpack webpack-cli
```

下载babel依赖

```bash
yarn add -D babel-loader @babel/core @babel/preset-env @babel/plugin-proposal-decorators
```

创建`webpack.config.js`

```js
//webpack.config.js
var path = require('path')
module.exports = {
    mode: "development",
    entry: { main: "./index.js" },
    output: {
        path: path.resolve(__dirname, "./build"),
        filename: "build.js"
    }, module: {
        rules: [
            {
                test: /\.m?js$/,
                exclude: /(node_modules|bower_components)/,
                use: {
                    loader: 'babel-loader'
                }
            }
        ]
    }
}
```

配置babel支持装饰器语法

```json
//.babelrc
{
    "presets": [
        "@babel/preset-env"
    ],
    "plugins": [
        [
            "@babel/plugin-proposal-decorators",
            {
                "legacy": true
            }
        ]
    ]
}
```

在这里为了便于演示，简单修改package.json部分字段

```json
{
    "scripts": {
        "build": "webpack",
        "start": "node ./build/build.js"
    },
}
```

接下来就可以在代码中使用装饰器语法了！

> **PS**  
> 如果使用VSCode做编辑器，可能会出现编辑器对装饰器语法依然报错的情况，可以在项目根目录下创建`jsconfig.json`配置VSCode对语法的支持。
> 
> ```json
> //jsconfig.json
> {
>     "compilerOptions": {
>         "experimentalDecorators":true,
>     },
>     "exclude": [
>         "node_modules",
>         "dist/*"
>     ]
> } 
> ```

## 装饰器的常用写法

装饰器 `@` 后必然需要接一个函数，这个函数需要带一个参数，也就是被装饰的类。  

**需要注意的是`@`后的函数名带不带括号，带括号表示对函数的调用，调用的返回值不一定是函数**

### 基础用法

```js
@fn
class MyClass{
    message = 'hello'
}

function fn(target){
    //在类上加入静态属性
    target.foo = 'bar'
}

//注意是类的静态属性
console.log(MyClass.foo)    //bar
```

### 加入参数的装饰器

```js
// 注意decorator后有没有括号，@后一定是一个函数
@fn2(20)
class MyClass{
    message = 'hello'
}

function fn2(value){
    //返回一个函数
    return function(target){
        target.count = value
    }
}

console.log(MyClass.count)  //20
```

### 给类的实例加入属性
```js
@fn3
class MyClass{
    message = 'hello'
}

function fn3(target){
    //target.prototype
    target.prototype.foo = 'bar_ins'    //给类的实例加入属性
}

const c1 = new MyClass();
console.log(c1.foo) //bar_ins
```

### 修饰类成员

```js
class MyClass{
    @fn4 message = 'hello'
}

//修饰类成员，3个参数
function fn4(target, name, descriptor){
    console.log(target)             //目标类的.prototype
    console.log(name)               //被修饰的类成员的名称:message
    console.log(descriptor)         //被修饰的类成员的描述对象

    //descriptor下有一些对成员的属性描述
    descriptor.writable = false     //只读
    descriptor.enumerable = false   //不能被遍历 
}

const c1 = new MyClass();
c1.message = 123    //error，不允许对只读属性赋值
```

## 参考：

- [ES6—Decorator，装饰器简介](https://zhuanlan.zhihu.com/p/55086365)


## 注解

### [2] TC39 Stage Process (TC39 的 Stage 阶段进程)

* 阶段 0：Strawperson 稻草人——代表目前仅仅是一个想法。  
* 阶段1: Proposal 提案——当想法变成提案，就需要进入阶段 1，面向委员会讲解和介绍，你需要概述解决方案，并且提出一些潜在的困难。委员会可能会接受你的提案，但并不代表就要在浏览器中生效。它仅仅是委员会觉得这是一个值得讨论的议题且愿意继续讨论。  
* 阶段2: Draft 草案——进入这一阶段的讨论会更加严肃，需要讨论具体的语法和语义的细节。你需要提供具体的解决方案，如何在语言中实现它，就像一个具体的 API 的实现。
* 阶段3: Candidate 候选——这个阶段设计的工作已经结束，你需要接受来自具体实现者和用户们的反馈。这个阶段也会有不同的 JavaScript 引擎来实现你的新特性。  
* 阶段4: finished 结束——一旦这个特性被添加进至少两个 JavaScript 实现并且通过具体的测试，代表着可以被大家使用了，提案的标准和规范也会进入到主要的标准规范中，我们会制定测试去保证未来的实现都会包含这项特性，也会添加参考文档。