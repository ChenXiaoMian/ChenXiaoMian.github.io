---
title: redux基本概念
date: 2019-11-11
categories: redux
tags: [redux]
---

## 基本概念和API

### 1、Store

`Store` 保存数据、`createStore` 用来生成Store。

```js
import { createStore } from 'redux';
const store = createStore(fn);
```

### 2、State

通过 `store.getState()` 拿到。

```js
import { createStore } from 'redux';
const store = createStore(fn);

const state = store.getState();
```

### 3、Action

`Action` 就是 View 发出的通知，表示 `State` 应该要发生变化了。
`Action` 是一个对象，`type` 是必须的。

```js
const action = {
  type: 'ADD_TODO',
  payload: 'Learn Redux'
}
```

### 4、Action Creator

定义一个函数来生成 `Action`，这个函数就叫 `Action Creator`。

```js
const ADD_TODO = '添加 TODO';
function addTodo(text) {
  return {
	type: ADD_TODO,
	text
  }
}
const action = addTodo('Learn Redux');
```

### 5、store.dispatch()

`store.dispatch()` 是 View 发出 Action 的唯一方法。

```js
import { createStore } from 'redux';
const store = createStore(fn);

store.dispatch({
  type: 'ADD_TODO',
  payload: 'Learn Redux'
})
```
```
store.dispatch(addTodo('Learn Redux'));
```

### 6、Reducer

Store 收到 Action 以后，必须给出一个新的 State，这样 View 才会发生变化。这种 State 的计算过程就叫做 `Reducer`。
Reducer 是一个函数，它接受 Action 和当前 State 作为参数，返回一个新的 State。

```js
const reducer = function(state, action) {
  // ...
  return new_state;
}
```
```js
const defaultState = 0;
const reducer = (state = defaultState, action) => {
  switch (action.type) {
    case 'ADD':
	  return state + action.payload
	default:
	  return state;
  }
}

const state = reducer(1, {
  type: 'ADD',
  payload: 2
})
```

### 7、纯函数

Reducer 函数最重要的特征是，它是一个纯函数。也就是说，只要是同样的输入，必定得到同样的输出。

由于 Reducer 是纯函数，就可以保证同样的State，必定得到同样的 View。
但也正因为这一点，Reducer 函数里面不能改变 State，必须返回一个全新的对象，请参考下面的写法。

```js
// State 是一个对象
function reducer(state, action){
  return Object.assign({}, state, { thingToChange });
  // 或者
  return { ...state, ...newState };
}

// State 是一个数组
function reducer(state, action) {
  return [...state, newItem];
}
```

### 8、store.subscribe()

Store 允许使用 `store.subscribe()` 方法设置监听函数，一旦 State 发生变化，就自动执行这个函数。

```js
import { createStore } from 'redux';
const store = createStore(reducer);

store.subscribe(listener);
```

显然，只要把 View 的更新函数（对于 React 项目，就是组件的render方法或setState方法）放入listen，就会实现 View 的自动渲染。

store.subscribe方法返回一个函数，调用这个函数就可以解除监听。

```js
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
);

unsubscribe();
```

### 9、Store 的实现

下面是 `createStore` 方法的一个简单实现，可以了解一下 Store 是怎么生成的。

```js
const createStore = (reducer) => {
  let state;
  let listeners = [];
  
  const getState = () => state;
  
  const dispatch = (action) => {
    state = reducer(state, action);
	  listener.forEach(listener => listener());
  }
  
  const subscribe = (listener) => {
	listeners.push(listener);
	return () => {
	  listeners = listeners.filter(l => l !== listener);
	}
  }
  
  dispatch({});
  
  return { getState, dispatch, subscribe };
}
```

### 10、combineReducers方法

```js
import { combineReducers } from 'redux';

const chatReducer = combineReducers({
  chatLog,
  statusMessage,
  userName
})

export default todoApp;
```

```js
const reducer = combineReducers({
  a: doSomethingWithA,
  b: processB,
  c: c
})

// 等同于
function reducer(state = {}, action) {
  return {
    a: doSomethingWithA(state.a, action),
    b: processB(state.b, action),
    c: c(state.c, action)
  }
}
```

### 11、中间件

想来想去，只有发送 Action 的这个步骤，即 `store.dispatch()` 方法，可以添加功能。
举例来说，要添加日志功能，把 Action 和 State 打印出来，可以对store.dispatch进行如下改造。

```js
let next = store.dispatch;

store.dispatch = function dispatchAndLog(action) {
  console.log('dispatching', action);
  next(action);
  console.log('next state', store.getState());
}
```

**如何使用中间件redux-logger?**

```js
import { applyMiddleware, createStore } from 'redux';
import createLogger from 'redux-logger';
const logger = createLogger();

const store = createStore(
  reducer,
  applyMiddleware(logger)
);
```

（1）createStore方法可以接受整个应用的初始状态作为参数，那样的话，applyMiddleware就是第三个参数了。

```js
const store = createStore(
  reducer,
  initial_state,
  applyMiddleware(logger)
);
```

（2）中间件的次序有讲究。

```js
const store = createStore(
  reducer,
  applyMiddleware(thunk, promise, logger)
);
```
上面代码中，`applyMiddleware` 方法的三个参数，就是三个中间件。有的中间件有次序要求，使用前要查一下文档。
比如，logger就一定要放在最后，否则输出结果会不正确。

## redux使用

redux-thunk 处理异步逻辑的 redux 中间件
immutable 进行持久性数据结构处理的库（使用原生支持深度更新）

Store 就是把它们联系到一起的对象。Store 有以下职责：

维持应用的 state；
提供 getState() 方法获取 state；
提供 dispatch(action) 方法更新 state；
通过 subscribe(listener) 注册监听器;
通过 subscribe(listener) 返回的函数注销监听器。

## redux-thunk 原理

实际上redux-thunk只是一个中间件，它的代码非常简单
```js
const thunk = store => next => action =>
   typeof action === 'function'
     ? action(store.dispatch, store.getState)
     : next(action)
```

