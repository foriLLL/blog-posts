

Redux 出现，将 Flux 与函数式编程结合一起，很短时间内就成为了最热门的前端架构。  
Flux 是一种架构思想，专门解决软件的结构问题。它跟MVC 架构是同一类东西，但是更加简单和清晰  

Flux 的最大特点，就是数据的"单向流动"。
![](https://www.ruanyifeng.com/blogimg/asset/2016/bg2016011503.png)

Flux将一个应用分成四个部分。
View： 视图层
Action（动作）：视图层发出的消息（比如mouseClick）
Dispatcher（派发器）：用来接收Actions、执行回调函数
Store（数据层）：用来存放应用的状态，一旦发生变动，就提醒Views要更新页面

每个Action都是一个对象，包含一个actionType属性（说明动作的类型）和一些其他属性（用来传递数据）。  
Dispatcher 只能有一个，而且是全局的。


https://www.ruanyifeng.com/blog/2016/01/flux.html

什么时候需要redux？
某个组件的状态，需要共享
某个状态需要在任何地方都可以拿到
一个组件需要改变全局状态
一个组件需要改变另一个组件的状态

reducer接收两个参数(state = initialLoginStatus, action)
在函数内部switch接收到的action的类型，返回新的state

Store 就是保存数据的地方，你可以把它看成一个容器。整个应用只能有一个 Store。
createStore()需要传入Reducer
```js
//createStore的一个简单实现
const createStore = (reducer) => {
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
Store对象包含所有数据。如果想得到某个时点的数据，就要对 Store 生成快照。这种时点的数据集合，就叫做 State。
store.getState()
store.dispatch()
store.subscribe()
当前时刻的 State，可以通过store.getState()拿到。
Redux 规定， 一个 State 对应一个 View。只要 State 相同，View 就相同。你知道 State，就知道 View 是什么样，反之亦然。



View 要发送多少种消息，就会有多少种 Action。如果都手写，会很麻烦。可以定义一个函数来生成 Action，这个函数就叫 Action Creator。

**store.dispatch()是 View 发出 Action 的唯一方法。**store.dispatch接受一个 Action 对象作为参数，将它发送出去。

store.dispatch方法会触发 Reducer 的自动执行。

**store.dispatch发出action，reducer根据action的type自动更新为新的state**

createStore()需要传入Reducer
createStore接受 Reducer 作为参数，生成一个新的 Store。以后每当store.dispatch发送过来一个新的 Action，就会自动调用 Reducer，得到新的 State。

![](https://www.ruanyifeng.com/blogimg/asset/2016/bg2016091802.jpg)

首先，用户发出 Action。  
然后，Store 自动调用 Reducer，并且传入两个参数：当前 State 和收到的 Action。 Reducer 会返回新的 State 。  
State 一旦有变化，Store 就会调用监听函数。  
listener可以通过store.getState()得到当前状态。如果使用的是 React，这时可以触发重新渲染 View。

```js
const Counter = ({ value, onIncrement, onDecrement }) => (
  <div>
  <h1>{value}</h1>
  <button onClick={onIncrement}>+</button>
  <button onClick={onDecrement}>-</button>
  </div>
);

const reducer = (state = 0, action) => {
  switch (action.type) {
    case 'INCREMENT': return state + 1;
    case 'DECREMENT': return state - 1;
    default: return state;
  }
};

const store = createStore(reducer);

const render = () => {
  ReactDOM.render(
    <Counter
      value={store.getState()}
      onIncrement={() => store.dispatch({type: 'INCREMENT'})}
      onDecrement={() => store.dispatch({type: 'DECREMENT'})}
    />,
    document.getElementById('root')
  );
};

render();
store.subscribe(render);
```