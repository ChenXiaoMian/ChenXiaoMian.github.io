---
title: JS高阶函数学习
date: 2018-05-08 09:03:34
categories: 读书笔记
tags: [javascript, 高阶函数]
---

# 高阶函数

> 高阶函数是`接受函数作为参数`并且/或者`返回函数作为输出`的函数。

## 函数作为参数传递

最经典的莫过于ajax异步请求的回调函数。

```js
var getUserInfo = function( userId, callback ){
    $.ajax( 'http://xxx.com/getUserInfo?' + userId, function( data ){
        if ( typeof callback === 'function' ){
            callback( data );
        }
    });
}
getUserInfo( 13157, function( data ){
    alert( data.userName );
});
```

当我们想在 ajax请求返回之后做一些事情，但又并不知道请求返回的确切时间时，最常见的方案就是把callback 函数当作参数传入发起 ajax 请求的方法中，待请求完成之后执行 callback 函数。

## 函数作为返回值输出

> ** 让函数继续返回一个可执行的函数，意味着运算过程是可延续的，** 也更能体现函数式编程的巧妙。

判断数据类型的例子

```js
var isType = function( type ){
    return function( obj ){
        return Object.prototype.toString.call( obj ) === '[object '+ type +']';
    }
};
var isString = isType( 'String' );
var isArray = isType( 'Array' );
var isNumber = isType( 'Number' );
console.log( isArray( [ 1, 2, 3 ] ) ); // 输出： true
```

## 综合实现

单例模式

```js
var getSingle = function ( fn ) {
    var ret;
    return function () {
        return ret || ( ret = fn.apply( this, arguments ) );
    };
};
```

既把函数当作参数传递，又让函数执行后返回了另外一个函数。

```js
var getScript = getSingle(function(){
    return document.createElement( 'script' );
});
var script1 = getScript();
var script2 = getScript();
alert ( script1 === script2 ); // 输出： true
```

# 高阶函数运用

## 实现抽象

> 高阶函数就是定义抽象，** 抽象让我们专注于预定的目标而无须关心底层的系统概念。**

如何实现 `forEach` 函数？

```js
const forEach = (arrry, fn) => {
    for(let i = 0; arrry.length; i++){
        fn(arrry[i])
    }
}
```

一般使用API，forEach的用户不需要理解 `forEach` 函数是如何遍历的，如此问题就被抽象出来了。

因此，`forEach` 函数这样调用它：

```js
forEach([1,2,3],(data)=>{
    // data 被作为参数从forEach函数传到当前的函数
})
```

forEach本质上遍历了数组，那如何遍历一个对象呢？

```js
const forEachObject = (obj, fn) => {
    for(var property in obj){
        if(obj.hasOwnProperty(property)){
            // 以key和value作为参数调用fn
            fn(property, obj[property])
        }
    }
}
```

`forEachObject` 接受第一个参数为js对象，第二个参数是一个函数fn，并分别以`key`和`value`作为参数调用`fn`。

运行结果：

```js
let object = {a:1,b:2}
forEachObject(object, (k,v) => console.log(k + ":" + v)
// a:1
// b:2
```

## AOP(面向切面编程)

`AOP（面向切面编程）` 的主要作用是把一些跟核心业务逻辑模块无关的功能抽离出来，这些
跟业务逻辑无关的功能通常包括日志统计、安全控制、异常处理等。

这样做的好处首先是可以保持业务逻辑模块的纯净和高内聚性，其次是可以很方便地复用日志统计等功能模块。

```js
Function.prototype.before = function(beforefn){
    var _self = this;
    return function(){
        beforefn.apply(this, arguments);
        return _self.apply(this, arguments);
    }
};
Function.prototype.after = function(afterfn){
    var _self = this;
    return function(){
        var ret = _self.apply(this, arguments);
        afterfn.apply(this, arguments);
        return ret;
    }
};
var func = function(){
    console.log(2);
};
func = func.before(function(){
    console.log(1);
}).after(function(){
    console.log(3);
});
func();
```

这种使用 AOP 的方式来给函数添加职责，也是 JavaScript 语言中一种非常特别和巧妙的装饰
者模式实现。

## 函数节流

`throttle` 函数的原理是，将即将被执行的函数用setTimeout延迟一段时间执行。如果该次延迟执行还没有完成，则忽略接下来调用该函数的请求。
`throttle` 函数接受2个参数，第一个参数为需要被延迟执行的函数，第二个参数为延迟执行的时间。

```js
var throttle = function(fn, interval){
    var _self = fn,     // 保存需要被延迟执行的函数引用
        timer,          // 定时器
        firstTime = true;   // 是否是第一次调用
    return function(){
        var args = arguments,
            _me = this;
        if(firstTime){      // 如果是第一次调用，不需要延迟执行
            _self.apply(_me, args);
            return firstTime = false;
        }
        if(timer){          // 如果定时器还在，说明前一次延迟执行还没有完成
            return false;
        }
        timer = setTimeout(function(){   // 延迟一段时间执行
            clearTimeout(timer);
            timer = null;
            _self.apply(_me, args);
        }, interval || 500);
    }
};

window.onresize = throttle(function(){
    console.log(1);
}, 500);
```

## 分时函数

有时候我们需要往页面大量添加DOM节点，但是要一次性往页面中创建成百上千个节点，短时间内显然会让浏览器吃不消，造成卡顿甚至假死。

解决方案之一就是使用 `timeChunk` 函数，`timeChunk` 函数让创建节点的工作分批进行，比如把1秒钟创建1000个节点，改为每隔200毫秒创建8个节点。

`timeChunk` 函数接受3个参数，第1个参数是创建节点时需要用到的数据，第2个参数是封装了创建节点逻辑的函数，第3个参数表示每一批创建的节点数量。

```js
var timeChunk = function(ary, fn, count){
    var obj,
        t;
    var len = ary.length;
    var start = function(){
        for(var i = 0; i < Math.min(count || 1, ary.length); i++){
            var obj = ary.shift();
            fn(obj);
        }
    };
    return function(){
        t = setInterval(function(){
            if(ary.length === 0){     // 如果全部节点都已经被创建好
                return clearInterval(t);
            }
            start();
        }, 200);  // 分批执行的时间间隔，也可以用参数的形式传入
    };
};
```

测试

```js
var ary = [];
for(var i = 1; i <= 1000; i++){
    ary.push(i);
}

var renderFriendList = timeChunk(ary, function(n){
    var div = document.createElement('div');
    div.innerHTML = n;
    document.body.appendChild(div);
}, 8);

renderFriendList();
```

# 结语

高阶函数的实现得益于JS的闭包机制，让开发者在实际项目中只需要关注业务逻辑和容易变化的部分，而无须关心底层的概念，避免了多余的代码。

# 参考

[javascript设计模式与开发实践](https://item.jd.com/11686375.html)
[javascript ES6 函数式编程入门经典](https://item.jd.com/12257861.html)