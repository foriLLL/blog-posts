
由于redux需要写很多繁琐的action和reducer

它通过透明的函数响应式编程(transparently applying functional reactive programming - TFRP)**使得状态管理变得简单和可扩展**

MobX 是状态管理库中侵入性最小的之一。这使得 MobX的方法不但简单，而且可扩展性也非常好


Mobx 是 flux 实现的后起之秀. 以更简单的时候和更少的概念, 让 flux 使用起来变得更简单.

相比 Redux 有mutation, action, dispatch 等概念. mobx则更加简洁, 更符合对store 增删改查的操作概念.

一个简单Demo理解mobx作用
```js
import React from "react";
import ReactDOM from "react-dom";
import { observable, action } from "mobx";
import { observer } from "mobx-react";

// 创建一个State对象
let appState = observable({ timer: 0 });

// 定义动作（更改State）
setInterval(
    action(() => {
        appState.timer += 1;
    }),
    1000
);

appState.resetTimer = action(() => {
    appState.timer = 0;
});

// 创建observer（监听observable的改变）
let App = observer(({ appState }) => {
    return (
        <div className="App">
            <h1>Time passed
: {appState.timer}</h1>
            <button onClick={appState.resetTimer}>reset timer</button>
        </div>
    );
});

const root = document.getElementById("root");
ReactDOM.render(<App appState={appState} />, root);
```

Observable state(可观察的状态)

Computed values(计算值)

Reactions(反应)  
使用autorun、reaction 和 when 函数即可简单的创建自定义 reactions，以满足你的具体场景。
* autorun 
* reaction
* when

```js
//存储数据
class TodoList {
    @observable todos = [];
    @computed get unfinishedTodoCount() {
        return this.todos.filter(todo => !todo.finished).length;
    }
}
----------------------
import React, {Component} from 'react';
import ReactDOM from 'react-dom';
import {observer} from 'mobx-react';

//展示列表
@observer
class TodoListView extends Component {
    render() {
        return <div>
            <ul>
                {this.props.todoList.todos.map(todo =>
                    <TodoView todo={todo} key={todo.id} />
                )}
            </ul>
            Tasks left: {this.props.todoList.unfinishedTodoCount}
        </div>
    }
}

//每一项TODO事项
const TodoView = observer(({todo}) =>
    <li>
        <input
            type="checkbox"
            checked={todo.finished}
            onClick={() => todo.finished = !todo.finished}
        />{todo.title}
    </li>
)

const store = new TodoList();
ReactDOM.render(<TodoListView todoList={store} />, document.getElementById('mount'));
```

yarn add mobx mobx-react
1. 初始化mobx容器仓库
```js
import {observable, action} from 'mobx'

class Store{
    @observable count=0
    @action.bound increment(){
        this.count++
    }
}
```
2. 在组件中使用mobx容器状态
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
3. 组件中发起action修改容器状态
```js
ReactDOM.render(<App store={new Store()} />, document.getElementById('root'))
```


## store间通信
```es6
//aStore
class aStore{
    @observable abc = []
    @observable tmp = 'sdaf'
    constructor(rootStore){
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
class roorStore{
    constructor(){
        this.aStore = new aStore(this)
        this.bStore = new bStore(this)
    }
}
```

## 将rootStore传递到所有组件
```js
//父组件
import { Provider } from 'mobx-react'
<Provider>
    <App />
<Provider />
```
```js
//子组件
import { observer, inject } from 'mobx-react'

@observer
@inject('productsStore')
class MyComponent{
    render(){
        console.log(this.props.productsStore)
        return(
            ...
        )
    }
}
```


首先是redux和mobx的对比：

redux将数据保存在单一的store中，mobx将数据保存在分散的多个store中
redux使用plain object保存数据，需要手动处理变化后的操作；mobx适用observable保存数据，数据变化后自动处理响应的操作
redux使用不可变状态，这意味着状态是只读的，不能直接去修改它，而是应该返回一个新的状态，同时使用纯函数；mobx中的状态是可变的，可以直接对其进行修改
mobx相对来说比较简单，在其中有很多的抽象，mobx更多的使用面向对象的编程思维；redux会比较复杂，因为其中的函数式编程思想掌握起来不是那么容易，同时需要借助一系列的中间件来处理异步和副作用
mobx中有更多的抽象和封装，调试会比较困难，同时结果也难以预测；而redux提供能够进行时间回溯的开发工具，同时其纯函数以及更少的抽象，让调试变得更加的容易
基于以上的对比，通常会有结论说：对于一些简单的，规模不大的应用来说，用mobx就足够了。但是是不是一些大的，复杂的应用就不能用mobx呢？我觉得不一定。其实如果能够合理的组织代码的结构，理清依赖关系，一些复杂的场景适用mobx也是可以的，不过mobx目前对react ssr的支持还没有redux那么完善，这也是急需改善的点。

相较于redux，mobx中存在着一些限制：

它不会分析你的数据中是否存在着循环依赖
它不会预先假定你的数据结构是plain object，class或是任何其他的数据类型

https://zhuanlan.zhihu.com/p/36294638

autorun, computed 内可观测的属性发生变化时自动执行（最开始也执行一次）  

## 计算属性computed
基于某些状态计算出的数据
```js
@computed get fn(){
    return ....
}
```
autorun默认会执行一次，内部可、观测的属性发生变化时自动执行

when
```js
//when两个参数，一个条件，一个操作
//只会执行一次，若默认符合，一开始就执行
when(
    ()=>{
        return store.count > 100
    },
    ()=>{
        cconsole.log("条件符合，进入")
    }
)
```

reaction
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