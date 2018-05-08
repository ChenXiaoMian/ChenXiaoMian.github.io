---
title: JS闭包学习笔记
date: 2018-04-23 23:12:14
categories: 读书笔记
tags: [javascript, 闭包]
---

# 闭包是什么

> 从技术的角度讲，所有的JavaScript函数都是`闭包`。————《javascript权威指南》

函数的执行依赖于变量作用域，而为了实现这种词法作用域，函数对象的内部状态还需要引用当前的作用域链，通过作用域相互关联起来，使得函数体内的变量都保存在函数作用域内，因此`闭包`的创建依赖于函数。

```js
function foo() {
    var a = 2;   // 局部变量
    function bar() {
        console.log( a );
    }
    return bar;
}
var baz = foo();
baz(); // 2
```

函数 `bar()` 的词法作用域能够访问 `foo()` 的内部作用域。然后我们将 `bar()` 函数本身当作一个值类型进行传递。`bar()`依然持有对该作用域的引用，而这个引用就叫作`闭包`。

**简单讲，`闭包`是指可以访问另一个函数作用域变量的函数，一般是定义在外层函数中的内层函数。**

# 闭包的作用

因为局部变量无法共享和长久的保存，而全局变量可能造成变量污染，所以我们希望有一种机制既可以长久的保存变量又不会造成全局污染，即 `让外部函数访问私有变量` 。

## 更多

* 管理私有变量和私有方法，将对变量（状态）的变化封装在安全的环境中

```js
var report = (function(){
    var imgs = [];
        return function( src ){
    var img = new Image();
    imgs.push( img );
        img.src = src;
    }
})();
```

* 将代码封装成一个闭包形式，等待时机成熟的时候再使用，比如实现柯里化和反柯里化

```js
var currying = function( fn ){
    var args = [];
    return function(){
        if ( arguments.length === 0 ){
            return fn.apply( this, args );
        }else{
            [].push.apply( args, arguments );
            return arguments.callee;
        }
    }
};
var cost = (function(){
    var money = 0;
    return function(){
        for ( var i = 0, l = arguments.length; i < l; i++ ){
            money += arguments[ i ];
        }
        return money;
    }
})();
var cost = currying( cost ); // 转化成 currying 函数
cost( 100 ); // 未真正求值
cost( 200 ); // 未真正求值
cost( 300 ); // 未真正求值
alert ( cost() ); // 求值并输出： 600
```

** 需要注意的，由于闭包内的部分资源无法自动释放，容易造成内存泄露（将变量设置为null）**

# 应用场景

## 循环和回调函数

```js
for(var i=1;i<=5;i++){
    (function(j){
        setTimeout(function timer(){
            console.log(j);
        }, j* 1000);
    })(i);
}
```

在循环的过程中每个迭代都需要一个闭包作用域。

在定时器、事件监听器、ajax请求、跨窗口通信、web workers或者异步任务中，只要使用了`回调函数`，实际上就是在使用`闭包`。

## 实现模块

大多数模块依赖加载器/管理器本质上都是将这种模块定义封装进一个友好的API。

```js
var MyModules = (function Manager(){
    var modules = {};

    function define(name, deps, impl){
        for(var i=0; i<deps.length; i++){
            deps[i] = modules[deps[i]];
        }
        modules[name] = impl.apply(impl, deps);  // `核心`
    }

    function get(name){
        return modules[name];
    }
    return {
        define: define,
        get: get
    };
})();

MyModules.define("bar", [], function(){
    function hello(who){
        return "Let me introduce:" + who;
    }
    return {
        hello: hello
    };
});

MyModules.define("foo", ["bar"], function(bar){
    var hungry = "hippo";

    function awesome(){
        console.log(bar.hello(hungry).toUpperCase());
    }
    return {
        awesome: awesome
    };
});

var bar = MyModules.get("bar");
var foo = MyModules.get("foo");

console.log(bar.hello("hippo"));
foo.awesome();
```

这段代码的核心是`modules[name] = impl.apply(impl, deps)`。为了模块的定义引入了包装函数（可以传入任何依赖），并且将返回值，也就是模块的API，储存在一个根据名字来管理的`模块列表`。

# 参考

[你不知道的JavaScript（上卷）](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20&%20object%20prototypes/README.md#you-dont-know-js-this--object-prototypes)
[javascript设计模式与开发实践](https://item.jd.com/11686375.html)
[JavaScript 里的闭包是什么？应用场景有哪些？](https://www.zhihu.com/question/19554716)