---
title: router路由原理
date: 2019-11-03
categories: javascript
tags: [router,路由原理]
---

## 1、页面路由
```js
window.location.href = "http://www.baidu.com";
history.back();
```

## 2、hash路由
```js
window.location.href = "#test";
window.location.href = "#test2";
```

通过onhashchange事件来获取hash值

```js
window.onhashchange = function(){
	console.log('current hash:', window.location.hash)
}
```

## 3、h5 路由

// 手动推进一个历史记录，三个参数，并不会立即加载这个URL，但可能会在稍后某些情况下加载这个URL

pushState() 需要三个参数: 一个状态对象, 一个标题 (目前被忽略), 和 (可选的) 一个URL
```js
history.pushState('test','Title','#test');
```

注意 pushState() 绝对不会触发 hashchange 事件，即使新的URL与旧的URL仅哈希不同也是如此。

history.replaceState() 的使用与 history.pushState() 非常相似，区别在于  replaceState()  是修改了当前的历史记录项而不是新建一个。 注意这并不会阻止其在全局浏览器历史记录中创建一个新的历史记录项。
```js
window.onpopstate = function(e) {
	console.log('h5 router change', e.state);
	console.log(window.location.href);
	console.log(window.location.pathname);
	console.log(window.location.hash);
	console.log(window.location.search);
}	
```
history 退栈的时候触发的事件。