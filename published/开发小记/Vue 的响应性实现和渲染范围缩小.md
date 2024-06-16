---
description: 本文从在项目中遇到的减少渲染开销的问题出发，简单记录了 Vue 响应性的实现思路，以及如何利用 Vue 的响应性来实现缩小渲染范围。
time: 2024-06-15T19:33:39+08:00
tags: 
heroImage: 
---

## TL; DR

### React 与 Vue 响应性思路的不同

- Vue：使用 ***依赖追踪*** 和 ***数据劫持*** 实现响应性，数据变更时自动触发更新。
- React：依赖 ***组件状态（state）*** 和 ***不可变数据结构***，状态变化时通过 **比较引用** 进行更新。
- 如果更新的组件在树中的位置非常高，渲染更新后的组件内部所有嵌套组件的默认行为将不会获得最佳性能。

## 背景

本文更像是讲一个最近遇到的故事，并记录从中发现的 Vue 和 React 对于响应性实现的不同思路以及相关思考。

故事的起因是在尝试对公司项目代码进行重构，一方面发现了很多可以优化提高性能的点，另一方面想对结构方面的一些东西做些修改，以便之后可以加入一些新 feature，也算是铺路了。

在第一次重构的 PR 中，我将组件中计算属性原来的语法做了一些改动（可以理解为从右边改到了左边），主要就是在原来的对象上加入了几个属性。修改后没有用原来的对象，而是通过展开语法，新建了一个对象。

<img alt="code example" src="https://img.foril.fun/20240615194005.png" style="display: block; margin:10px auto"/>

这样做在 React 中十分常见，因为 React 是依赖于 **引用** 比较（`Object.is`）来检测状态变化（不可变性原则，每一个 state 应该作为一个快照，不应该被更新）。当你直接修改现有的数组或对象的内容，无法检测到数组和对象的变化，因为它仅仅是比较引用。**如果引用相同，即使数组或对象发生了变化，React认为状态没有发生变化而不会重新去渲染组件。**

用下面这个 React 例子说明这个问题：

```js
'use client'
import { useState } from "react";
export default function App() {
  const [person, setPerson] = useState({
    name: "previous name",
    age: 22,
  });
  function changeName() {
    console.log('changeName')
    // 1.
    // 使用第一种代码，每次点击后都给 setPerson 传一个新对象，会打印 changeName，并触发重新渲染
    // 重新渲染过程中会打印 rerender，名称变为 new name
    
    setPerson({
      ...person,
      name: "new name",
    });
    
    // 2.
    // 使用第二种代码，点击后会打印 changeName，但 setPerson 发现是原来的对象，不会触发重渲染
    // 因此不会打印 rerender，名称不变化
    
    // person.name = "new name";
    // setPerson(person);
  }
  console.log('rerender')
  return (
    <div>
      <button onClick={changeName}>click to change name</button>
      <div>{person.name}</div>
    </div>
  );
}
```

在上面的例子中，如果我们没有使用新的对象，而是在原来的对象上做了修改然后传入 `setPerson`，React 发现对象还是原来的引用，则不会继续接下来的任务。

而使用展开语法生成新的对象，由于每次调用 `setPerson` 传入的都是一个不同的对象，所以每次点击都会触发重新渲染，第一次点击，React 比较虚拟 DOM 和之前的虚拟 DOM 不一致，会更新实际 DOM 节点。  
而之后的点击则会经历以下流程：

1. 调用 `changeName` 函数。
2. `setPerson` 接收到一个新的对象（尽管 `name` 值是一样的）。
3. React 认为状态发生了变化，因此会触发重新渲染。
4. 在重新渲染过程中，React 会比较新的虚拟 DOM 树和之前的虚拟 DOM 树。
5. 由于 `person.name` 的值没有变化，React 在比较虚拟 DOM 树时会发现对应的文本节点没有变化，从而不会更新实际的 DOM 节点。

因此，虽然每次点击按钮都会触发重新渲染，但由于 `name` 的值没有变化，React 不会更新实际的 DOM 节点。

> 这个流程也简单说明了 React 这种以虚拟 DOM 减少开销的方法，这种方法在 Vue 中也被借鉴，我们简单梳理一下流程：
> 1. 状态变更触发：用户交互或事件处理函数调用 setState（如 setPerson）。
> 2. 重新渲染：React 重新调用组件函数，生成新的虚拟 DOM 树。
> 3. 虚拟 DOM 比较：React 比较新的虚拟 DOM 树和旧的虚拟 DOM 树，找出差异（使用 diff 算法）。
> 4. DOM 更新：React 仅更新实际 DOM 中有变化的部分（根据虚拟 DOM 比较结果）。 
 
> 虚拟 DOM 树其实就是一个纯 JavaScript 的对象。

```js
const vnode = {
  type: 'h1',
  props: {
    id: 'hello'
  },
  children: [
    /* 更多 vnode */
  ]
}
```

所以在 React 中，我们一般不会在原来的对象上修改，状态被认为是只读的，你应该替换它而不是改变现有对象。更改局部变量不会触发渲染。使用展开语法生成新的对象是一个好的编码习惯。

由于对 Vue 没有很熟悉，我想当然地认为改成左边的代码没有什么不同，而且也应该这么写。

然后我就发现，怎么重构了以后，渲染开销一下变得很大？甚至是原来的若干倍，并且由于这个渲染开销的变化是在一个大的重构里引入的，我在定位这个原因的过程中大概花了三四天。最后发现这个问题的根本其实是由于 Vue 对于响应式能力的实现思路与 React 有所区别，所以上面的代码写法会导致渲染开销增大，并且从这一点出发，也进一步发现了很多项目中可以优化的地方。

> 事实证明一个提交确实最好不要做太多改动，不然定位 bug 很痛苦


## Vue 的响应性实现

### 什么是响应性

*Vue 最标志性的功能就是其低侵入性的响应式系统。响应性是一种可以使我们声明式地处理变化的编程范式。*

用人话说就是一个地方的数据 A 加入依赖了其他数据 B 和 C，当 B 或 C 变化时，A 也会随之变化。

Excel 中的公式就是一个很好的例子，`C3 = C1 + C2`，当 C1 或 C2 变化时，C3 也会随之变化。

<img alt="响应性的理解" src="https://img.foril.fun/响应性的理解.gif" width=200px style="display: block; margin:10px auto"/>

而在 JavaScript 中，如果有下面这样的内容，B 或 C 修改时，A 是不会发生响应式修改的。

```js
let B = 1;
let C = 2;
let A = B + C;
```

为了实现 A 的更新，我们将对 A 的修改包装在一个函数里：

```js
function update() {
  A = B + c;
}
```

这里的 `update` 我们称作一个 *副作用（effect）*，`B` 和 `C` 就是这个 effect 的依赖。
Vue 中的响应式也就可以形式化为：追踪 effect 的依赖，effect 作为依赖的订阅者，依赖发生变化时调用 effect（修改状态）。

所以我们需要一种机制，能够监听当 `B` 或 `C` 这些依赖发生变化时，调用这个 effect。

这里就可以用到观察者模式或者发布订阅模式。

当一个变量被读取时进行追踪。例如我们执行了表达式 `B + C` 的计算，则 `B` 和 `C` 都被读取到了。
如果一个变量在当前运行的 effect 中被读取了，就将该副作用设为此变量的一个订阅者。例如由于 `B` 和 `C` 在 `update()` 执行时被访问到了，则 `update()` 需要在第一次调用之后成为 `B` 和 `C` 的订阅者。
探测一个变量的变化。例如当我们给 `B` 赋了一个新的值后，应该通知其所有订阅了的副作用重新执行。

那么接下来要考虑的问题就变成了，什么机制可以探测到变量的变化呢？  

在 JavaScript 中，没有办法直接追踪对于 non-object 的局部变量的变化，但是我们可以追踪对象属性的读写。  
在 JavaScript 中有两种劫持 property 访问的方式：

- [getter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get#description) / [setter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set#description)
- [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)

Vue 正是利用了这两种方式来实现响应式的。

- Vue 2 使用 getter / setters 是出于支持旧版本浏览器的限制。
- Vue 3 中使用了 Proxy 来创建响应式对象，仅将 getter / setter 用于 ref。

下面的伪代码将会说明它们是如何工作的：

```js
function reactive(obj) {
  return new Proxy(obj, {
    get(target, key) {
      track(target, key)
      return target[key]
    },
    set(target, key, value) {
      target[key] = value
      trigger(target, key)
    }
  })
}

function ref(value) {
  const refObject = {
    get value() {
      track(refObject, 'value')
      return value
    },
    set value(newValue) {
      value = newValue
      trigger(refObject, 'value')
    }
  }
  return refObject
}
```

```js
// 这会在一个副作用就要运行之前被设置
let activeEffect

function track(target, key) {
  if (activeEffect) {
    const effects = getSubscribersForProperty(target, key)
    // Map<target, Map<key, Set<effects>>>    
    effects.add(activeEffect)
  }
}
```

```js
function trigger(target, key) {
  const effects = getSubscribersForProperty(target, key)
  effects.forEach((effect) => effect())
}
```

有了上面的思路，接下来我们引进一个 `whenDepsChange` 的函数，我们想让 `update` 能响应式执行，只需要首先保证 `B` 和 `C` 是响应式变量，外面包一个 `effect` 函数，在执行 `update` 时，外层的 `activeEffect` 设置为这个 `effect`，这样 `update` 内部的响应式变量（依赖）被 get 时就可以将这个 effect 加入订阅集合。

```js
function whenDepsChange(update) {
  const effect = () => {
    activeEffect = effect
    update()
    activeEffect = null
  }
  effect()
}
```

简单地说，Vue 的响应性实现就是劫持了数据的 getter 和 setter，每当 *计算属性* 或 *watch* 或 *template* 中用到了 `someReactiveState.someKey`，就会触发一个 `track` ，会注册这个 `target` 对象和对应的 `key` 被追踪，而当前的 template 或 effect 或 计算属性 会被作为一个 `activeEffect` 被加入一个订阅列表，每当对应的 `target.key` 被 set 时（trigger），就会触发对应的订阅列表中的所有 effect 一个一个被执行。


## 如何利用 Vue 的响应性来实现缩小渲染范围

我们用另一个 demo 展示这个问题：

```html
<template>
  <div>
    <button @click="changeFirst()">change First</button>
    <!-- <div v-for="p in people" :key="p.id">{{ p.name }} {{ p.id }}</div> -->
    <ListItem v-for="p in people" :key="p.id" :item="p" />
  </div>
</template>

<script>
import ListItem from './components/Item.vue'
const init = [];
for (let i = 0; i < 100000; i++) {
    init.push({
        name: 'previous name',
        id: i,
        age: 22
    })
}

export default {
  name: 'App',
  components: {
    ListItem,
  },
  data() {
    return {
      people: init,
    }
  },
  methods: {
    changeFirst() {
      this.people[0].name = 'tada'
    }
  },
  updated() {
    console.log(`App updated`)
  }
}
</script>
```

上面这个例子很能说明问题，如果使用 `div`，`p.name` 在 App 中被 `track`，调用 `changeFirst` 时，会触发 App 的 render，打印出 `App updated`，而触发 App 的渲染会导致一个复杂的 VDOM 构造、VDOM 比较以及子组件的递归比较等，而如果使用 `Item` 组件，只传入 `p`，那么在 App 中，只有 `people` 里的对象被 `track`，而不是 `p.name` 这个属性，那么我们更新 `p.name` 就不会触发 App 的渲染，而在第一个 `Item` 组件内部调用了 `item.name`，那么 `p.name` （这里的 `p` 只是下标为 0 的那一个对象）的 set 会触发第一个 `Item` 的渲染。相比之下，这样做只触发了一个 `Item` 组件的更新，渲染的开销极小。

从 Devtools 调试中我们也可以看到使用 `div` 时 `renderEffect` 的实例是 `App`，而使用 `ListItem` 时 `renderEffect` 的实例是 `ListItem`。

<img alt="渲染范围 App" src="https://img.foril.fun/渲染范围 App.PNG" width=600px style="display: block; margin:10px auto"/>

<img alt="渲染范围 ListItem" src="https://img.foril.fun/渲染范围 ListItem.PNG" width=600px style="display: block; margin:10px auto"/>

回到最开始遇到的渲染开销的问题，想要在每个 item 上加一些属性，如果是使用原来的对象，只会触发 `ListItem` 的重渲染，而构建新对象，会触发上层组件的重渲染，Vue 的虚拟 DOM 系统将对整个组件树进行重新渲染。在这个过程中，Vue 需要比较大量新旧虚拟 DOM 树，找出差异，并决定如何最有效地更新实际的 DOM。

因此使用展开语法构建新对象的写法会造成更多的内存分配（每个对象都创建了一个新实例）和更复杂的虚拟 DOM 操作。这种方法在处理大型数组或对象时应谨慎使用，因为这样可能会显著影响应用程序的性能。

> 即使一棵树的某个部分从未改变，还是会在每次重渲染时创建新的 vnode，带来了大量不必要的内存压力。  
> 这也是虚拟 DOM 最受诟病的地方之一：这种有点暴力的更新过程通过牺牲效率来换取声明式的写法和最终的正确性。


## *有关另一个在项目中的 bug

有了上面的了解，接下来我们再看一个实际代码中的例子，这段代码的目的是一个项目有缩略图时渲染缩略图，没有的时候先渲染一个 Icon：

<img alt="20240616001917" src="https://img.foril.fun/20240616001917.png" width=600px style="display: block; margin:10px auto"/>

<img alt="20240616001926" src="https://img.foril.fun/20240616001926.png" width=600px style="display: block; margin:10px auto"/>

但当使用 `item.hasOwnProperty('thumbnail')` 时，Vue 的响应式系统注册的 get 的 `target` 和 `key` 是 `item.hasOwnProperty`，所以只有当 `item.hasOwnProperty` 被 set 时才会触发组件的重渲染。
另一方面，使用 `item.thumbnail === undefined` 时，实际上是在访问 `item` 对象的 `thumbnail` 属性，这会创建一个 getter，使得 thumbnail 成为一个响应式依赖。因此，当 thumbnail 属性的值更新时，Vue 会检测到这个变化，并触发组件的重渲染。
所以使用 `item.thumbnail === undefined` 可以正确地触发重渲染，而使用 `!item.hasOwnProperty('thumbnail')` 则不会。

---

### FYI

在 Vue 3.2.46 `以后，hasOwnProperty` 也被追踪，所以上面这个 `hasOwnProperty` 的代码在新版本 Vue 中也可以正常运行。详见 Vue repo 中的[这个提交](https://github.com/vuejs/core/commit/588bd44f036b79d7dee5d23661aa7244f70e6beb)。
