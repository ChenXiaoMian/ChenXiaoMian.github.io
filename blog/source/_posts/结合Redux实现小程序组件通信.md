---
title: 结合Redux实现小程序组件通信
date: 2020-06-24
categories: 微信小程序
tags: [微信小程序, Redux, Promise]
---

## 前言

我想，一提到小程序内部数据共享，大家肯定会想到 `globalData`。但是 `globalData` 是依托于全局 `app` 对象，而全局变量的影响大家是心知肚明的，不一定哪个新手给你搞坏了也是说不定的。所以也就导致了状态变化的不可跟踪。我们都知道 `Redux` 是一个前端状态管理的容器，对于构建大型应用，对里面进行数据、状态的管理非常方便，而如何去实现一个小程序版本的`Redux`的难点是我们如何实现一个类似于`react-redux`的东西将`Redux`结合到小程序里面来。

在`React`类库的项目中，当`State`发生变化时，通过`props`接收参数的组件也会同时发生变化，最终触发`render`函数并完成视图更新。在小程序项目中，基于小程序的内部实现机制，通过`setData`改变`data`对象中的属性，同样能达到更新视图的目的。

如果读过`Redux`源码或读过相关文章就会发现，`Redux`实现`State`状态管理及小程序自带实现视图更新的功能，可以简单概况如下：
* 订阅————监听状态，保存对应的回调。
* 发布————状态变化，执行回调。
* 同步视图————同步数据到视图。

## 如何实现

项目目录：

- miniprogram
	- reducers           
		- index.js         (reducers 集合)
		- userInfo.js      (用户信息)
		- redux.min.js     (redux 库)
	- pages/       			 (...)
	- utils/              
		- connect.js       (将state状态机制融合到小程序里)
		- shallowEqual.js  (Redux库中的工具函数)
		- toPromise.js     (封装wx自带的接口，使其支持Promise)
	app.js 						   (小程序入口)
	app.json             (小程序配置)
	app.wxss						 (全局样式)

**Step1**

实现小程序组件通信的思路是这样的，首先，通过 `Redux` 实现发布订阅模式，从官方网站获取 `redux.min.js` 文件，放到项目对应的目录下。然后创建对应的 `Reducers` 文件 `userInfo.js`。

```js
const INITAL_STATE = {
  data: null
}
const User = (state = INITAL_STATE, action) => {
  switch(action.type) {
    case "MODIFY_USER": 
      return {
        state,
        ...action.data
      };
    default:
      return state;
  }
}

export default User;
```

**Step2**

创建 `reducers/index.js` 文件，应用 `Redux` 的 `combineReducers()` 辅助函数，用来整合所有 `Reducers`。

```js
import { createStore, combineReducers } from './redux.min.js';
import userInfo from './userInfo';

export default createStore(combineReducers({
  userInfo
}));
```

**Step3**

通过 `Object.defineProperty` (Vue的设计思路) 方式更新视图，同时以“原型继承”的方式对小程序的生命周期进行包装。当`State`发生变化时，如果状态值不一样，就同步执行`setData`函数以更新视图。

对小程序的生命周期进行包装，是实现小程序通信的核心步骤，`connect.js`的主要功能是在继承配置项后，针对不同的生命周期加入对应的订阅、发布和销毁机制。当小程序展示页面时，会判断是否存在销毁记录，如果没有就重新监听`State`。当小程序关闭或隐藏时，销毁之前的监听事件。

`connect.js` 代码

```js
import shallowEqual from './shallowEqual';
let __Store = getApp().Store;

export default (mapStateToData = () => {}) => {
  let config = {
    __Store,
    __dispatch: __Store.__dispatch,
    __destory: null,
    __observer() {
      if (super.__observer) {
        super.__observer();
        return;
      }
      const state = __Store.getState();
      const newData = mapStateToData(state);
      const oldData = mapStateToData(this.data || {});
      if (shallowEqual(oldData, newData)) {
        return;
      }
      this.setData(newData);
    },
    onLoad() {
      super.onLoad();
      this.__destory = this.__Store.subscribe(this.__observer);
      this.__observer();
    },
    onUnload() {
      super.onUnload();
      this.__destory && this.__destory() & delete this.__destory;
    },
    onShow() {
      super.onShow();
      if (!this.__destory) {
        this.__destory = this.__Store.subscribe(this.__observer);
        this.__observer();
      }
      let state = getApp().Store.getState();
      if (this.route != 'pages/index/index' && (
        !state.userInfo ||
        !state.userInfo.data ||
        !state.userInfo.data.name)) {
          wx.reLaunch({
            url: '/pages/index/index',
          });
          wx.showToast({
            title: '请您重新登录',
            icon: 'none',
            mask: true,
            duration: 2000
          });
      }
    },
    onHide() {
      super.onHide();
      this.__destory && this.__destory() & delete this.__destory;
    }
  }
  return (options = {}) => {
    Object.setPrototypeOf(config, {
      __observer: null,
      onLoad() {  },
      onUnload() {  },
      onShow() {  },
      onHide() {  },
      ...options
    });
    return config;
  }
}
```

`shallowEqual.js` 代码
```js
const hasOwn = Object.prototype.hasOwnProperty;

function is(x, y){
  if (x === y) {
    return x !== 0 || y !== 0 || 1 / x === 1 / y;
  } else {
    return x !== x && y !== y;
  }
}

export default function shallowEqual(objA, objB) {
  if (is(objA, objB)) return true;

  if (typeof objA !== 'object' || objA === null || 
      typeof objB !== 'object' || objB === null) {
    return false;
  }
  const keysA = Object.keys(objA);
  const keysB = Object.keys(objB);

  if (keysA.length !== keysB.length) return false;
  for (let i = 0; i < keysA.length; i++) {
    if (!hasOwn.call(objB, keysA[i]) || 
        !is(objA[keysA[i]], objB[keysA[i]])) {
      return false;
    }
  }
  return true;
}
```

**Step4**

在小程序入口文件`app.js`文件，需要先把整合好的 `Store` 作为 `App`启动的参数。

`app.js` 代码

```js
import Store from './reducers/index';

//app.js
App({
  Store,
  onLaunch: function () {
		// ...
	},
	// 获取用户信息
  getUserInfo() {
		// ...
	}
});
```

**Step5**

在需要用到状态机制的文件中引入`connect.js`文件，对其生命周期进行包装。

```js
import connect from '../../utils/connect';

const mapStateToProps = (state) => {
  return {
    userInfo: state.userInfo
  }
}

Page(connect(mapStateToProps)({
	data: {},
	onLoad: {}
}))
```

此时的 `this` 对象中，已经内置了`__Store`、`__dispatch`、`__destory` 和 `__observe` 属性。

* __Store: 状态管理中心。
* __dispatch: 用来触发 `Reducer` 函数。
* __destory: 销毁者，取消监听事件。
* __observe: 观察者，在 `State` 状态变化时执行对应的回调函数。

其用法也比较简单。如果页面通过 `connect.js` 进行包装，只需用 `this.__dispatch` 即可触发对应的 `Reducer` 函数。如果页面或组件没有通过 `connect.js` 进行包装，可以调用小程序实例对象的扩展属性 `getApp().Store.dispatch` 来触发。

## 支持 Promise

微信小程序框架提供的 JavaScript API 基本上都是异步的，如 `wx.login()`、`wx.getUserInfo()`、`wx.getLocation()`等。这些接口提供的回调处理方式比较传统：在参数中传入 `success/fail/complete` 回调函数。为了在项目中更方便的编写代码，避免“回调地狱”问题，`toPromise.js` 代码主要利用ES5的特性 `Object.defindProperty`，以及小程序对原生Promise的支持，重新定义了小程序的部分接口。

```js
let wxKeys = ['request', 'login', 'uploadFile', 'chooseImage', 'checkSession', 'getSetting', 'openSetting', 'showModal', 'getUserInfo', 'scanCode'];

Promise.prototype.finally = function(callback) {
  let P = this.constructor;
  return this.then(
    value => P.resolve(callback()).then(() => value),
    reason => P.resolve(callback()).then(() => {
      throw reason;
    })
  )
}

wxKeys.forEach(key => {
  const wxFnKey = wx[key];
  if (wxFnKey && typeof wxFnKey === 'function') {
    Object.defineProperty(wx, key, {
      get() {
        return (option = {}) => {
          return new Promise((resolve, reject) => {
            option.success = res => {
              resolve(res);
            };
            option.fail = res => {
              reject(res);
            };
            wxFnKey(option);
          });
        }
      }
    })
  }
});
```

## 参考

* [koa与node.js开发实战](https://item.jd.com/12471619.html)