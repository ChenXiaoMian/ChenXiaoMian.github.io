---
title: 《javascript设计模式与开发实践》观后感
date: 2017-11-26 22:21:26
categories: 读书笔记
tags: [javascript,设计模式]
---

好记忆不如烂笔头，这句话是很有道理的。看过的东西，当时有感悟就应该记录下来，回忆和复习是更浪费时间的一件事。《javascript设计模式与开发实践》--曾探著。这本书写的真好，我看了很久（内容很多），之后隔了很久才想起来要记录下来。

先说一说什么是设计模式吧。通俗的讲：设计模式是在某种场合下对某个问题的一种解决方案，其实也就是给面向对象软件开发中的一些好的设计取个名字。作用呢？就是在熟悉这些模式，对某些模式的理解的情况下，形成条件反射，在解决问题的时候能很快的找到解决方案。

本书分为三部分，分别为基础知识，设计模式、设计原则和编程技巧，那我就挑些重点记录下来，供以后参考。

# 基础知识

javascript没有提供传统面向对象语言中的类式继承，而是通过原型委托的方式来实现对象与对象之间的继承。主要有几个特性，分别是动态类型，多态性，封装，原型继承。

## 动态类型

在javascript中，我们都知道对一个变量赋值时，可以不需要考虑它的类型，从而可以把更多精力放在业务逻辑上，编写的代码数量更少，看起来更加简洁。但是，动态类型语言对变量类型的宽容给实际编码带来很大灵活的同时，也会让程序在某些情况下变得难以理解，在程序运行期间有可能发生于跟类型相关的报错，需要编写一些检测类型的代码，可谓是有利也有弊。

## 多态性

多态的实际含义是：同一操作作用于不同的对象上面，可以产生不同的解释和不同的执行结果。换句话说，就是给不同的对象发送同一个消息的时候，这些对象会根据这个消息分别给出不同的反馈。

举个栗子

``` javascript
var makeSound = function(animal){
    if(animal instanceof Duck){
        console.log('嘎嘎嘎');
    }else if(animal instanceof Chicken){
        console.log('咯咯咯');
    }
};
var Duck = function(){};
var Chicken = function(){};
makeSound(new Duck());      //嘎嘎嘎
makeSound(new Chicken());   //咯咯咯
```

但是以上的做法如果增加一个狗叫声的话，就必须改动makeSound函数，才能实现“多态性”。这样做并没有把“做什么”和“谁去做”分离开。应该把不变的部分隔离出来，可变的部分封装起来。归根结底就是要消除他们之间的耦合关系。

``` javascript
var makeSound = function(animal){
    animal.sound();
};
var Duck = function(){};
Duck.prototype.sound = function(){
    console.log('嘎嘎嘎');
}
var Chicken = function(){};
Chicken.prototype.sound = function(){
    console.log('咯咯咯');
}
makeSound(new Duck());      //嘎嘎嘎
makeSound(new Chicken());   //咯咯咯
var Dog = function(){};
Dog.prototype.sound = function(){
    console.log('汪汪汪');
}
makeSound(new Dog());       //汪汪汪
```

`多态最根本的作用就是通过把过程化的条件分支语句转化为对象的多态性，从而消除这些条件分支语句。`

假设我们要编一个地图应用，有两家可选的API提供商选择接入我们的应该。目前我们选择的是谷歌地图，谷歌地图API提供了show方法，负责在页面上展示整个地图。
``` javascript
var googleMap = {
    show: function(){
        console.log('开始渲染谷歌地图');
    }
};
var renderMap = function(){
    googleMap.show();
};
renderMap();            //输出：开始渲染谷歌地图
```

后来因为某些原因，要求把谷歌地图换成百度地图，我们使用条件分支来让renderMap函数同时支持两家地图API。

``` javascript
var googleMap = {
    show: function(){
        console.log('开始渲染谷歌地图');
    }
};
var baiduMap = {
    show: function(){
        console.log('开始渲染百度地图');
    }
};
var renderMap = function(type){
    if(type === 'google'){
        googleMap.show();
    }else if(type === 'baidu'){
        baiduMap.show();
    }
};
renderMap('google');            //输出：开始渲染谷歌地图
renderMap('baidu');             //输出：开始渲染百度地图
```

如果在增加一个搜搜地图呢，无疑必须改动renderMap函数，继续堆砌条件分支语句。

我们还是可以把程序中相同的部分抽象出来，那就是显示地图show方法。

``` javascript
var renderMap = function(map){
    if(map.show instanceof Function){
        map.show();
    }
};
renderMap(googleMap);            //输出：开始渲染谷歌地图
renderMap(baiduMap);             //输出：开始渲染百度地图
var sosoMap = {
    show: function(){
        console.log('开始渲染搜搜地图');
    }
}
renderMap(sosoMap);              //输出：开始渲染搜搜地图
```

这段代码中，分别调用它们的show方法，会产生各自不同的执行结果。即使增加搜搜地图，renderMap函数任然不需要做任何改变。这一就把“做什么”和“怎么去做”分开了。

## 封装

封装的目的就是将信息隐藏。在许多语言中，封装数据都是由语法解析，提供了private、public、protected等关键字来提供不同的访问权限。但是在javascript中并没有提供这些关键字的支持，而是通过函数创建作用域来模拟出public和private这两种封装性。

封装不仅仅是隐藏数据，还包括隐藏实现细节，设计细节以及隐藏对象的类型等。

## 原型继承

如果我们想创建一个对象，一种方法就是先指定它的类型，然后通过类来创建这个对象。原型模式选择另一种方式，我们不再关心对象的具体类型，而是找到一个对象，然后通过克隆来创建一个一模一样的对象。

``` javascript
var Plane = function(){
    this.blood = 100;
    this.attackLevel = 1;
    this.defenseLevel = 1;
};
var plane = new Plane();
plane.blood = 500;
plane.attackLevel = 10;
plane.defenseLevel = 7;
var clonePlane = Object.create(plane);
console.log(clonePlane.blood);              //输出500
console.log(clonePlane.attackLevel);        //输出10
console.log(clonePlane.defenseLevel);       //输出7
```

在不支持Object.create方法的浏览器中，可以使用以下代码：

``` javascript
Object.create = Object.create || function(obj){
    var F = function(){};
    F.prototype = obj;
    return new F();
}
```
javascript的原型继承遵守的基本规则

* 所有的数据都是对象。
``` javascript
var obj1 = new Object();
var obj2 = {};
console.log(Object.getPrototypeOf(obj1)===Object.prototype);  //输出：true  
console.log(Object.getPrototypeOf(obj2)===Object.prototype);  //输出：true
```
* 要得到一个对象，不是通过实例化，而是找到一个对象作为原型并克隆它。
``` javascript
function Person(name){
    this.name = name;
};
Person.prototype.getName = function(){
    return this.name;
};
var a = new Person('sven');
console.log(a.name);        //输出：sven
console.log(a.getName());   //输出：sven
console.log(Object.getPrototypeOf(a)===Person.prototype);   //输出：true
```
* 对象会记住它的原型(`__proto__`)。
``` javascript
var a = new Object();
console.log(a.__proto__===Object.prototype);  //输出：true  
```
* 如果对象无法响应某个请求，会把这个请求委托给它自己的原型。
``` javascript
var obj = {name: 'sven'};
var A = function(){};
A.prototype = obj;
var a = new A();
console.log(a.name);    //输出：sven
```
