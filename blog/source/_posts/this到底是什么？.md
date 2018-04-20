---
title: this到底是什么？
date: 2018-04-16
categories: javascript
tags: [javascript, this]
line_number: false
---

# 为什么要使用this

记得小学上英语课的时候，学着介绍自己的名字，每个人都是造一个同样的句型。

`普通函数`写法

```javascript
function myName(name){
    console.log('my name is ' + name);
}
myName('xiaoming'); // my name is xiaoming
myName('xiaohong'); // my name is xiaohong
```

使用`this`写法

```javascript
function myName(){
    console.log('my name is ' + this.name);
}
var a = { name: 'xiaoming' };
var b = { name: 'xiaohong' };
myName.call(a); // my name is xiaoming
myName.call(b); // my name is xiaohong
```

介绍自己的名字当然比较简单，那么我们来增加点难度。

`普通函数`写法

```javascript
function myName(name){
    return "my name is " + name;
}

function mySex(sex){
    return "I'm a " + sex;
}

function myAge(age){
    return "I'm " + age + " years old";
}

function introduce(name, sex, age){
    var greeting = myName(name) + '. ' + mySex(sex) + '. ' + myAge(age);
    console.log(greeting);
}
introduce('xiaoming','boy','9'); // my name is xiaoming. I'm a boy. I'm 9 years old
introduce('xiaohong','girl','7'); // my name is xiaohong. I'm a girl. I'm 7 years old
```

使用`this`写法

```javascript
function myName(){
    return "my name is " + this.name;
}

function mySex(){
    return "I'm a " + this.sex;
}

function myAge(){
    return "I'm " + this.age + " years old";
}

function introduce(){
    var greeting = myName.call(this) + '. ' + mySex.call(this) + '. ' + myAge.call(this);
    console.log(greeting);
}

var a = {
    name: 'xiaoming',
    sex: 'boy',
    age: '9'
};

var b = {
    name: 'xiaohong',
    sex: 'girl',
    age: '7'
};

introduce.call(a); // my name is xiaoming. I'm a boy. I'm 9 years old
introduce.call(b); // my name is xiaohong. I'm a girl. I'm 7 years old
```

如果不使用`this`，那么就需要给`myName()`函数显式的传入一个参数`name`来代替不同的上下文对象，单单一个参数还可以接受，可随着你的使用模式越来越复杂，显式传递上下文对象会让代码变得越来越混乱。所以，`this`提供了一种更优雅的方式来隐式“传递”一个对象的引用，可以将API设计的更加`简洁`易于`复用`。

# this到底是什么？

> `this`是运行时进行绑定的，是基于`函数的执行环境动态绑定`的，而非函数被声明时的环境。

当一个函数被调用时，会创建一个活动记录（也称为上下文）。这个记录会包含函数在哪里被调用（调用栈）、函数的调用方式、传入的参数 等信息。`this`就是这个记录的一个函数，会在函数执行的过程中用到。

# this的指向

* 默认绑定（作为普通函数调用，指向`全局对象`）
* 隐式绑定（作为对象的方法调用，指向`该调用对象`）
* 显式绑定（使用call和apply方法，指向`第一个参数对象`）
* new绑定（作为构造函数调用，指向`新创建的对象`）

## 默认绑定

当函数不作为对象的属性被调用时，也就是我们常说的普通函数方式，此时的`this`总是指向`全局对象`。

```javascript
window.name = 'globalName';

function getName(){
    var name = 'functionName';
    console.log(this.name);  // globalName
}

getName();
```

严格模式

```javascript
window.name = 'globalName';

function getName(){
    "use strict";
    var name = 'functionName';
    console.log(this.name);  // Uncaught TypeError: Cannot read property 'name' of undefined
}

getName();
```

如果使用`严格模式`，则不能将全局对象用于默认绑定。

## 隐式绑定

当函数作为对象的方法被调用时，`this`指向`该对象`。

```javascript
function foo(){
    console.log(this.a);
}

var obj = {
    a: 2,
    foo: foo
};

obj.foo(); // 2
```

对象属性引用链中只有`上一层`或者说是`最后一层`在调用位置中起作用。

```javascript
function foo(){
    console.log(this.a);
}

var obj2 = {
    a: 42,
    foo: foo
};

var obj1 = {
    a: 2,
    obj2: obj2
}

obj1.obj2.foo(); // 42
```

## 显式绑定

`Function`的原型定义了两个方法，它们是`Function.prototype.call`和`Function.prototype.apply`。它们的第一个参数，允许直接指定`this`的绑定对象。

```javascript
function foo(){
    console.log(this.a);
}

var obj = {
    a: 2
};

foo.call(obj); // 2
```

### call和apply的区别

`apply`接收两个参数，第一个参数指定了函数体内`this`对象的指向，第二个参数为一个带下标的集合，可以是`数组`或`类数组`，可以将这个集合中的元素作为参数传递。

```javascript
var func = function(a,b,c){
    alert([a,b,c]);  // 输出 [1,2,3]
};

func.apply(null, [1,2,3]);
```

`call`传入的参数数量不固定，但是跟`apply`相同的是，第一个参数也是代表了函数体内`this`对象的指向，之后的参数每个依次传入函数。

```javascript
var func = function(a,b,c){
    alert([a,b,c]);  // 输出 [1,2,3]
};

func.call(null,1,2,3);
```

### bind的用法

`bind`是把它的上下文绑定到`bind()`括号中的参数上，它`不会执行`，而只是返回一个改变了上下文的函数副本，而call和apply是`直接执行`函数。

```javascript
var button = document.getElementById("button"),
    text = document.getElementById("text");
button.onclick = function() {
    alert(this.id); // 弹出text
}.bind(text);
```

## new绑定

当用`new`调用函数时，会执行以下操作：

* 创建一个全新的对象。
* 新的对象会被执行[Prototype]链接。
* 新对象会绑定到函数调用的`this`。
* 如果函数没有返回其他对象，那么`new`中的函数调用会自动返回这个新对象。

```javascript
var myClass = function(){
    this.name = 'javascript';
};

var obj = new myClass();
console.log(obj.name); // javascript
```

# 判断this && 优先级

优先级

> new绑定 > 显示绑定 > 隐式绑定 > 默认绑定

按以下顺序判断：

1、函数是否在 `new` 中调用（new绑定）？如果是的话`this`绑定的是`新创建的对象`。

```javascript
var bar = new foo();  // bar
```

2、函数是否通过`call`、`apply`（显示绑定）或`bind`硬绑定？如果是的话`this`绑定的是`指定的对象`。

```javascript
var bar = foo.call(obj2);  // obj2
```

3、函数是否在某个上下文对象调用（隐式绑定）？如果是的话`this`绑定的是`那个上下文对象`。

```javascript
var bar = obj1.foo();  // obj1
```

4、如果都不是的话，使用默认绑定。如果在`严格模式`下，就绑定到`undefined`，否则绑定到`全局对象`。

```javascript
var bar = foo();
```

# 参考

[你不知道的JavaScript（上卷）](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20&%20object%20prototypes/README.md#you-dont-know-js-this--object-prototypes)
[javascript设计模式与开发实践](https://item.jd.com/11686375.html)

仅供个人学习记录~