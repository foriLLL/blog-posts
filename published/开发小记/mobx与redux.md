---
description: "Redux 和 MobX 是当下较为流行的两个状态管理库，在复杂前端项目的状态管理中发挥了很大的作用，那么我们应该怎么使用它们，又该如何选择呢？本文记录一下学习过程中的笔记。"
time: 2021-09-19
heroImage: ""
tags: []
---
Redux 和 MobX 是当下较为流行的两个状态管理库，在复杂前端项目的状态管理中发挥了很大的作用，那么我们应该怎么使用它们，又该如何选择呢？本文记录一下学习过程中的笔记。

## 什么是状态管理  

当我们在使用 React 进行开发时，每个组件都有自己的状态可以管理，在简单的应用中，我们只需要组件管理好自己的状态，同时向下传递状态；而随着项目规模变大变复杂，在一些场景下，可能会出现一些比较棘手的组件间状态通信难题：
* 多个组件间的状态共享
* 组件触发其他组件的状态更新

> React 只是 DOM 的一个抽象层，**并不是 Web 应用的完整解决方案**。有两个方面，它没涉及：  
> 1. 代码结构
> 2. 组件之间的通信

按照 React 的设计思想，给出的建议是将公共状态提升，再需要的地方一层一层传递下来，但这样会使得代码臃肿而混乱，维护性极差；Redux 和 MobX 这两个状态管理库，本质任务都是**解决状态管理混乱问题**，更好地管理状态，解决项目中常见到的一些组件间的状态通信问题。  

> 如果你只是想避免层层传递一些属性，组件组合（component composition）有时候是一个比 context 更好的解决方案。  

## Redux

再介绍 Redux 之前，不得不先介绍 Flux。  

Flux 架构的概念由 Facebook 在 2014 年提出。不严谨地讲，它和 MVC 架构应该算同一个级别的东西。采用 Flux 组织你的前端应用开发，可以使你的数据组织更加清晰简单，更便于维护。  

Flux 的最大特点，就是数据的"单向流动"。  

![Flux 的数据单向流动](https://img0.baidu.com/it/u=3576204246,1143892932&fm=26&fmt=auto)

它将一个应用分成四个部分。
* View： 视图层
* Action（动作）：视图层发出的消息（比如 mouseClick）  
* Dispatcher（派发器）：用来接收 Actions、执行回调函数（对应 Redux 的 Reducer）  
* Store（数据层）：用来存放应用的状态，一旦发生变动，就提醒 Views 要更新页面

每个 Action 都是一个**对象**，包含一个 actionType 属性（说明动作的类型）和一些其他属性（用来传递数据）。Dispatcher 只能有一个，而且是全局的。  

概括来说，整个数据流动过程包括用户与 View 交互，触发 Action，这时 Dispatcher 监听到 Action，根据 Action 的类型（ActionType）执行不同的回调函数，生成新的 Store，Store 在发生变动后，向 View 发出事件，实现数据和视图的同步。

Redux 就是对 *Flux* 与 *函数式编程* 的统一，与 React 结合，成为一时热门的前端解决方案。其特点是可预测（源于其纯函数的使用）、易调试（配合 Redux DevTools），且能够与任何 UI 层搭配使用，使你的应用数据中心化。  

## Redux的API使用
对于 Redux 来说：
1. Web 应用是一个状态机，视图与状态是一一对应的。
2. 所有的状态，保存在一个对象里面。

其基本组成与 Flux 对应，包括：
### Store
Store就是整个应用保存数据的地方，**整个应用只能有一个 Store**。使用 Redux，可以使用 `createStore` 这个函数生成 Store。

```js
import { createStore } from 'redux';
const store = createStore(reducer);
```

`createStore`这个函数 **接受一个 Reducer**，生成 Store。  
（Reducer 传入第一个参数默认值即是 Store 初始值，后面会提到。）

下面是 createStore 的一个简单实现
```js
const createStore = (reducer) => {
  //本质是闭包
  let state;
  let listeners = [];

  const getState = () => state;

  const dispatch = (action) => {
    state = reducer(state, action);
    listeners.forEach(listener => listener());
  };

  const subscribe = (listener) => {
    listeners.push(listener);
    return () => {
      listeners = listeners.filter(l => l !== listener);
    }
  };

  dispatch({});

  return { getState, dispatch, subscribe };
};
```
此外，Store 有三个重要的方法：
* `store.getState()`  
  Store 对象包含所有数据。如果想得到某个时间点的数据，就要对 Store 生成快照。这种时间点的数据集合，就叫做 **State**。当前时刻的  State，可以通过 `store.getState()` 拿到。  
* `store.dispatch()`  
  是 **View 发出 Action 的唯一方法**，触发 Reducer 的自动执行，会在稍后的 Action 中提到。
* `store.subscribe()`
  Store 允许使用 store.subscribe 方法设置监听函数，一旦 State 发生变化，就自动执行这个函数（比如更新视图的操作）；store.subscribe方法返回一个函数，调用这个函数就可以解除监听。
  ```js
  let unsubscribe = store.subscribe(() =>
    console.log(store.getState())
  );
  unsubscribe();
  ```

### Action
Action 的本质其实就是一个对象，其中 `type` 属性是必选的，也可以自定义其他属性用来传递任意想要的数据。 

```js
//Action
const action = {
  type: 'ADD_TODO',
  payload: 'Learn Redux'
};
```

当我们想要让 Action 携带不同的数据的时候（比如上面例子中的`payload`），就可以使用一个函数接收参数，返回一个携带了信息的Action，这样的函数就是 `Action Creator`。

```js
const ADD_TODO = '添加 TODO';

//Action Creator
function addTodo(text) {
  return {
    type: ADD_TODO,
    payload: text   //将作为参数的text置入对象
  }
}
//返回对象作为Action
const action = addTodo('Learn Redux');
```

当视图层有所动作，需要触发 Action 时，需要使用 `store.dispatch()` 方法（**唯一方法**），将 Action 作为参数传入；`store.dispatch()` 会触发 Reducer 的自动执行，得到更新后的 State。

```js
import { createStore } from 'redux';
const store = createStore(fn);

store.dispatch({
  type: 'ADD_TODO',
  payload: 'Learn Redux'
});
```

结合刚才提到的 `Action Creator` 你可以将代码写作：

```js
store.dispatch(addTodo('Learn Redux'));
```

### Reducer

Reducer 可能是整个数据流动环节中最为重要的一环，负责根据 Action 的不同类型，执行不同的操作，更新 State。  

**Reducer 的实质是一个函数。他接受当前 State 和一个 Action 作为参数，返回一个新的 State。**  当没有最初始的 State 时，也就是第一次调用 Reducer 时，State 采用传入的默认值，也就是 State 的初始值。

```js
const defaultState = 0; //State初始值，以default的形式传给Reducer第一个参数
const reducer = (state = defaultState, action) => {
  switch (action.type) {
    case 'ADD':
      return state + action.payload;
    default: 
      return state;
  }
};
```

store.dispatch 方法会触发 Reducer 的自动执行。为此，Store 需要知道 Reducer 函数，做法就是在生成 Store 的时候，将 Reducer 传入 createStore 方法。（串起来了）

> 为什么这个函数叫做 Reducer 呢？因为它可以作为数组的 reduce 方法的参数。（参数是 state, action 且返回新的 state）请看下面的例子，一系列 Action 对象按照顺序作为一个数组。
> ```js
> const actions = [
>   { type: 'ADD', payload: 0 },
>   { type: 'ADD', payload: 1 },
>   { type: 'ADD', payload: 2 }
> ];
> 
> const total = actions.reduce(reducer, 0); // 3
> ```
> 上面代码中，0首先作为初始值，传入reducer，相当于`reducer(0, { type: 'ADD', payload: 0 })`，返回新的state=0+0=0，返回的值0作为当前值，继续调用reducer，相当于`reducer(0, { type: 'ADD', payload: 1 })`，得到新的state=0+1=1，以此类推。

上面说到 Reducer 的本质是一个函数，这里需要注意的是，Reducer 是一个**纯函数**，这就意味着，面对同样的输入，Reducer 必然能得到同样的输出。  
纯函数是函数式编程的概念，必须遵守以下一些约束：
* 不得改写参数
* 不能调用系统 I/O 的 API
* 不能调用 Date.now() 或者 Math.random() 等不纯的方法，因为每次会得到不一样的结果

也正因为此，Reducer 函数里面 **不能改变 State，必须返回一个全新的对象**，请参考下面的写法。

```js
// State 是一个对象
function reducer(state, action) {
  return Object.assign({}, state, { thingToChange });   //用thingToChange覆盖原state中的key
  // 或者
  return { ...state, ...newState }; //同样是覆盖，返回的是全新的对象
}

// State 是一个数组
function reducer(state, action) {
  return [...state, newItem];   //返回的是全新的数组
}
```

### Reducer的拆分

整个应用只能有一个 Store，这样会导致对于 Reducer 的管理非常冗杂，我们可以把 Reducer 函数拆分。不同的函数负责处理不同属性，最终把它们合并成一个大的 Reducer 即可。

```js
const chatReducer = (state = defaultState, action = {}) => {
  const { type, payload } = action;
  switch (type) {
    case ADD_CHAT:
      return Object.assign({}, state, {
        chatLog: state.chatLog.concat(payload)
      });
    case CHANGE_STATUS:
      return Object.assign({}, state, {
        statusMessage: payload
      });
    case CHANGE_USERNAME:
      return Object.assign({}, state, {
        userName: payload
      });
    default: return state;
  }
};

             |
             | 
             | 改为
             |
            \|/

const chatReducer = (state = defaultState, action = {}) => {
  return {
    chatLog: chatLog(state.chatLog, action),    //每个小的Reducer都返回一个小的state
    statusMessage: statusMessage(state.statusMessage, action),
    userName: userName(state.userName, action)
  }
};
```

Redux 提供了一个 combineReducers 方法，用于 Reducer 的拆分。你只要定义各个子 Reducer 函数，然后用这个方法，将它们合成一个大的 Reducer。

```js
import { combineReducers } from 'redux';

const chatReducer = combineReducers({
  chatLog,
  statusMessage,
  userName
})

export default todoApp;
```

下面是combineReducer的简单实现。

```js
const combineReducers = reducers => {
  //返回的是一个大Reducer
  return (state = {}, action) => {
    //大Reducer返回的是所有的小state累加  
    return Object.keys(reducers).reduce(
      (nextState, key) => {
        nextState[key] = reducers[key](state[key], action);
        return nextState;
      },    //accumulator
      {}    //初始值
    );
  };
};
```

### Redux工作流程

附上 Redux 的工作流程：

![Redux 工作流程](https://img.foril.fun/bg2016091802.jpg)

## MobX

MobX 也是 flux 架构的一种实现，但是它凭借更简单的 api 和更快的速度，得到了很多人的追捧。MobX 是状态管理库中侵入性最小的之一。这使得  MobX 的方法不但简单，而且可扩展性也非常好。React 和 MobX 是一对强力组合。React 通过提供机制把应用状态转换为可渲染组件树并对其进行渲染。而MobX提供机制来存储和更新应用状态供 React 使用。

> Anything that can be derived from the application state, should be derived. Automatically.  
> 任何起源于应用状态的数据应该自动获取。

MobX 的 api 非常间接也很符合直觉，如果项目支持装饰器语法，编写会更加简单高效，具体如何配置可参考 [在 JavaScript 中使用装饰器](https://www.foril.space/article/36) 。  

### MobX 的使用

### 核心概念

#### @observable

通过使用 @observable 装饰器（ES.Next）来给你的类属性添加注解，可以使被注解的属性作为一个 **可观察的状态**，可被外界观测到该状态的变化

#### @observer

通过 @observer 修饰你的组件，可以使组件观测到内部使用的可观测状态（observable）的变化，及时更新。

#### @action

通过 @action 可将你的函数变为一个 **动作**，可在其中对状态进行修改，在严格模式下，只有动作才可以修改状态。  
`action.bound` 会绑定 this。   
`runInAction` 函数：

```js
import { runInAction } from 'mobx'
runInAction(()=>{
    store.count=10; //直接调用修改store
})
```

#### @computed

基于某些状态计算出的数据提炼为一个方法。**所依赖的可观测状态没有变化时，多次调用只会执行一次，计算结果会被缓存。（计算属性的好处之一）**

```js
import { computed } from 'mobx'
class Store{
    @observable count = 0;
    @computed get doubleCount(){
        return this.count * 2;
    }
} 
```

***

### 监视数据

MobX 主要提供了三种方式监视数据：

#### autorun

在 autorun 内部使用到的可观测状态发生改变时，autorun 会自动执行；同时 autorun 默认会在初始时执行一次。

```js
autorun(()=>{
        console.log(store.count)    //依赖了store.count，发生变化时会执行（初始也会执行）
    }
)
```

#### when

`when(predicate: () => boolean, effect?: () => void, options?)`  
when 观察并运行给定的 predicate，直到返回 true。 一旦返回 true，给定的 effect 就会被执行，然后 autorunner(自动运行程序) 会被清理。 该函数返回一个清理器以提前取消自动运行程序。

```js
//when两个参数，一个条件，一个操作
//只会执行一次，若默认符合，一开始就执行
when(
    ()=>{
        return store.count > 100
    },
    ()=>{
        cconsole.log("条 件符合，进入")
    }
)
```

#### reaction

autorun 的变种，对于如何追踪 observable 赋予了更细粒度的控制。 它接收两个函数参数，第一个（**数据函数**）是用来追踪并返回数据，同时作为第二个函数（**效果函数**）的输入。不同于 autorun 的是当创建时效果函数不会直接运行，只有在数据表达式首次返回一个新值后才会运行。在执行效果函数时访问的任何 observable 都不会被追踪。

```js
//不同于autorun和when，只有当被观测的数据被改变的时候才会执行（没有autorun的第一次执行）
reaction(
    ()=>{
        //业务操作，返回数据给下一个函数使用
        return store.count
    },
    (countData, reaction)=>{    
        console.log(countData)
        reaction.dispose()  //停止当前reaction对数据的监听
    }
)
```

***

接下来简单记录 MobX 的具体使用。

### 安装

```bash
yarn add mobx
# React 绑定库:
yarn add mobx-react
```

### 初始化MobX容器仓库

```js
import {observable, action} from 'mobx'

class Store{
    @observable count=0
    @action.bound increment(){
        this.count++
    }
}
```

### 在组件中使用MobX容器状态

```js
ReactDOM.render(<App store={new Store()} />, document.getElementById('root'))   //也可以使用Provider
```

### 组件中发起action修改容器状态

```js
import {observer} from 'mobx-react'

@observer
class App extends React.Component{
    render(){
        const {store} = this.props
        return(
            <div>
                <p>{store.count}</p>
                <p>
                    <button onClick={store.increment}>增加</button>
                </p>
            </div>
        )
    }
}
```

### store间通信

```js
//aStore
class aStore{
    @observable abc = []
    @observable tmp = 'sdaf'
    constructor(rootStore){
        //传入根store使能够找到其他store
        this.rootStore = rootStore
    }
}

//bStore
class bStore{
    @observable asdf = []
    @observable ddd = 'sdaf'
    constructor(rootStore){
        this.rootStore = rootStore
    }
}

//index
class rootStore{
    constructor(){
        //引入各个小的store
        this.aStore = new aStore(this)
        this.bStore = new bStore(this)
    }
}
```

### 将rootStore传递到所有组件

```js
//父组件
import { Provider } from 'mobx-react'
import RootStore from './stores'
<Provider {...RootStore}>
    <App />
<Provider />

//子组件
import { observer, inject } from 'mobx-react'

@observer
@inject('StoreName')    //把容器中的数据成员映射进来
class MyComponent{  
    render(){
        console.log(this.props.StoreName)
        return(
            ...
        ) 
    } 
}
```

## MobX还是Redux

MobX 和 Redux 都是应用状态管理库，都适用于 React，Angular，VueJs 等框架或库，而不局限于某一特定 UI 库，在项目中应该如何选择呢？  
从上面的笔记中也可以看出来，MobX 的使用比 Redux 要方便许多，Redux 需要写很多繁琐的 action 和 reducer，而 MobX 是状态管理库中侵入性最小的之一。这使得 MobX 的方法不但简单，而且可扩展性也非常好。  
同时 Redux 对 ts 的支持比较困难，而 MobX 则完美支持 ts。

所以，MobX 可能更适合在简单的、数据流不太复杂的中小型项目中，但这并不表示其不能支撑大型项目，关键在于大型项目通常需要特别注意可拓展性，可维护性，相比而言，规范的 Redux 更有优势，而 MobX 更自由，需要我们自己制定一些规则来确保项目后期拓展，维护难易程度；

## 参考

* [你需要 MobX 还是 Redux？](https://www.cnblogs.com/zhouyangla/p/10165650.html)
* [React 官方文档 Context](https://react.docschina.org/docs/context.html)
* [阮一峰 React 技术栈系列教程](http://www.ruanyifeng.com/blog/2016/09/)
* [React 系列教程之 MobX](https://www.bilibili.com/video/BV1tL4y1h7ND?p=3&spm_id_from=pageDriver)