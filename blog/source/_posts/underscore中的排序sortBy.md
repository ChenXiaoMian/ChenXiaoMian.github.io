---
title: underscore中的排序sortBy
date: 2017-07-16 01:05:18
categories: js源码解读
tags: [underscore,sortBy]
---
优秀博客文章网络上有很多，一直以来都是读别人的文章。自己嘛，一来文笔不好，二来毫无头绪不知道想分享些什么，于是越是不开始就越不知道怎么写。看了很多鸡汤文，鼓励写博客的文章，终于鼓起勇气写写。目的呢，记录平时所看的书，所想的事，还有想说的话。这可能是今年来的第一篇博文吧，万事开头难，加油咯！

前端技术日新月异，更新的频率太快了，回头想想自己掌握的知识还是很薄弱，谈不上精通，更还有些许浮躁。一想至此，决定开始分析一些经典的js源码，这次先从underscore开始分析，结构性上jquery那么复杂，也能更好的学习函数式编程。

[underscore 1.8.2 源码地址](http://www.css88.com/doc/underscore1.8.2/docs/underscore.html)
[underscore 1.8.2 中文文档](http://www.css88.com/doc/underscore1.8.2/)

# Array.prototype.sort

ES5中，已对集合提供一个排序方法 [Array.prototype.sort](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)

``` bash
var students = [
  {name: 'wxj', age: 18},
  {name: 'john', age: 14},
  {name: 'bob', age: 23}
];
var sortedStudents = students.sort(function(left, right) {
  return right.age < left.age;
});
# => sortStudents: [
#  {name: 'john', age: 14},
#  {name: 'wxj', age: 18},
#  {name: 'bob', age: 23},
# ]
```

# _.sortBy
``` bash
_.sortBy(list, iteratee, [context]) 
```
返回一个排序后的list拷贝副本。如果传递iteratee参数，`iteratee将作为list中每个值的排序依据`。迭代器也可以是字符串的属性的名称进行排序的(比如 length)。[api戳这里](http://www.css88.com/doc/underscore1.8.2/#sortBy)

## 用法
``` bash
# iteratee将作为list中每个值的排序依据
_.sortBy([1, 2, 3, 4, 5, 6], function(num){ return Math.sin(num); });
# => [5, 4, 6, 3, 1, 2]
```
``` bash
# 根据属性的名称进行排序
var stooges = [{name: 'moe', age: 40}, {name: 'larry', age: 50}, {name: 'curly', age: 60}];
_.sortBy(stooges, 'name');
=> [{name: 'curly', age: 60}, {name: 'larry', age: 50}, {name: 'moe', age: 40}];
```

## 源码
``` bash
  _.sortBy = function(obj, iteratee, context) {
    # 内部函数，优化回调
    iteratee = cb(iteratee, context);
    # 先通过map生成新的对象集合,该对象提供了通过iteratee计算后的值, 方便排序
    # [{value:1,index:0,criteria: sin(1)}, ...]
    # 再排序.sort
    # 最后再通过pluck把值摘出来
    return _.pluck(_.map(obj, function(value, index, list) {
      return {
        value: value,
        index: index,
        criteria: iteratee(value, index, list)
      };
    }).sort(function(left, right) {
      var a = left.criteria;
      var b = right.criteria;
      if (a !== b) {
        if (a > b || a === void 0) return 1;
        if (a < b || b === void 0) return -1;
      }
      return left.index - right.index;
    }), 'value');
  };
```
# 开始分析

`_.sortBy`的工作流程：

## cb

`cb` 函数将根据不同情况来为我们的迭代创建一个迭代过程 `iteratee`。[迭代！迭代！迭代！](https://yoyoyohamapi.gitbooks.io/undersercore-analysis/content/base/迭代！迭代！迭代！.html)

## _.map

通过 [_.map](http://www.css88.com/doc/underscore1.8.2/#map) 生成一个新的集合，该集合的每个元素是一个对象，他具有三个属性：
* value:元素值
* index:索引位置
* criteria:排序准则，该准则将通过被优化的 iteratee 计算得到。underscore 看到了元素间的比较仍将落脚到“值比较”的本质。

假设我们现在有集合：
``` bash
var students = [
    {name: 'wxj', age: 18},
    {name: 'john', age: 14},
    {name: 'bob', age: 23}
];  
var sortedStudents = _.sortBy(students, 'age');  
# sortStudents: [
#  {name: 'john', age: 14},
#  {name: 'wxj', age: 18},
#  {name: 'bob', age: 23},
# ]
```

假设我们需要按照年龄进行排序，那么传入的 iteratee 为：
``` bash
var iteratee = function(value, key, index, elem) {
    return elem.age;
}
```
经过该过程，该集合将变为：
``` bash
var newStudents = [
    {
      value: {name: 'wxj', age: 18},
      index: 0,
      criteria: 18
    },
    {
      value: {name: 'john', age: 14},
      index: 0,
      criteria: 14
    },
    {
      value: {name: 'bob', age: 23},
      index: 0,
      criteria: 23
    }
];
```

利用 [Array.prototype.sort](#Array-prototype-sort) 以及我们确定的排序准则 criteria 对新生成的集合进行排序：

``` bash
var sortedStudents = newStudents.sort(function (left, right) {
  var a = left.criteria;
  var b = right.criteria;
  if (a !== b) {
      if (a > b || a === void 0) return 1;
      if (a < b || b === void 0) return -1;
  }
  return left.index - right.index;
});
# => sortedStudents: [
#    {
#      value: {name: 'john', age: 14},
#      index: 0,
#      criteria: 14
#    },
#    {
#      value: {name: 'wxj', age: 18},
#      index: 0,
#      criteria: 18
#    },
#    {
#      value: {name: 'bob', age: 23},
#      index: 0,
#      criteria: 23
#    }
#];
```

## _.pluck

再通过 [_.pluck](http://www.css88.com/doc/underscore1.8.2/#pluck) 取出 value 属性，过滤掉不需要的 index 及 criteria 属性。

``` bash
var ret = _.pluck(sortedStudents, 'value');
# => ret: [
#  {name: 'john', age: 14},
#  {name: 'wxj', age: 18},
#  {name: 'bob', age: 23},
#]
```
大功告成！感谢[@yoyoyohamapi](https://github.com/yoyoyohamapi)童鞋，文章是转载的，勿喷。搬运一番，也深入理解了一层。

# 参考
> * [更好用的排序](https://yoyoyohamapi.gitbooks.io/undersercore-analysis/content/collection/更好用的排序.html)
> * [迭代！迭代！迭代](https://yoyoyohamapi.gitbooks.io/undersercore-analysis/content/base/迭代！迭代！迭代！.html)