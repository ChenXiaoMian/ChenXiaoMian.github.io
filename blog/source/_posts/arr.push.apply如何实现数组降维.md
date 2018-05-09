---
title: 'arr.push.apply如何实现数组降维'
date: 2018-05-09 16:44:11
categories: javascript
tags: [javascript, apply, push]
---

# 前言

这个问题是出自学习《javascript ES6 函数式编程入门经典》第五章 5-7 concatAll 函数时的疑惑。

有这样的一个对象数组：

```js
let apressBooks = [
  {
    name : "beginners",
    bookDetails : [
      {
        "id": 111,
        "title": "C# 6.0"
      },
      {
        "id": 222,
        "title": "Efficient Learning Machines"
      }
    ]
  },
  {
      name : "pro",
      bookDetails : [
      {
        "id": 333,
        "title": "Pro AngularJS"
      },
      {
        "id": 444,
        "title": "Pro ASP.NET"
      }
    ]
  }
];
```

假设我们想要获取所有`bookDetails`的数据，我们先使用`map`函数。

`map`函数

```js
const map = (array,fn) => {
  let results = []
  for(const value of array)
      results.push(fn(value))
  return results;
}
```

使用`map`函数

```js
map(apressBooks, (book)=>{
    return book.bookDetails
})
```

返回结果：

```js
[
    [
      {
        "id": 111,
        "title": "C# 6.0"
      },
      {
        "id": 222,
        "title": "Efficient Learning Machines"
      }
    ],
    [
      {
        "id": 333,
        "title": "Pro AngularJS"
      },
      {
        "id": 444,
        "title": "Pro ASP.NET"
      }
    ]
]
```

如你所见，`map`函数返回的数据包含了数组中的数组。因为 `bookDetails` 本身就是一个数组，那么我们还需要将所有嵌套数组连接到一个数组中（降维操作）！这时候要使用 `concatAll` 函数。

`concatAll` 函数

```js
const concatAll = (array, fn) => {
    let results = [];
    for(const value of array){
        results.push.array(results, value);  // 为什么不是results.push(value);
    }
    return results;
}
```

`concatAll` 的主要目的是`将嵌套数组转换为非嵌套的单一数据`。

使用 `concatAll` 函数

```js
concatAll(
    map(apressBooks, (book)=>{
        return book.bookDetails
    })
)
```

返回结果（正是我们想要的结果）：

```js
[
  {
    "id": 111,
    "title": "C# 6.0"
  },
  {
    "id": 222,
    "title": "Efficient Learning Machines"
  },
  {
    "id": 333,
    "title": "Pro AngularJS"
  },
  {
    "id": 444,
    "title": "Pro ASP.NET"
  }
]
```

# 疑惑

`concatAll` 函数中的 `results.push.array(results, value)` 为什么要这么调用，不能直接 `results.push(value)` ？

我们打印一下`value`是什么值。

```js
[
  {
    "id": 111,
    "title": "C# 6.0"
  },
  {
    "id": 222,
    "title": "Efficient Learning Machines"
  }
]
[
  {
    "id": 333,
    "title": "Pro AngularJS"
  },
  {
    "id": 444,
    "title": "Pro ASP.NET"
  }
]
```

是两个数组，那直接 `results.push(value)` 调用的话，

```js
const concatAll = (array, fn) => {
    let results = [];
    for(const value of array){
        results.push(value);
    }
    return results;
}
```

可想而知，结果自然还是不对的。也就是 `map` 函数返回的结果，看上。

# 答案

`call`和`apply`的区别

> `Function.prototype.call` 和 `Function.prototype.apply` 都是非常常用的方法。它们的作用一模一样，区别仅在于传入参数形式的不同。

apply 接受两个参数，第一个参数指定了函数体内 this 对象的指向，第二个参数为一个带下标的集合，** 这个集合可以为数组，也可以为类数组 **， apply 方法把这个集合中的元素作为参数传递给被调用的函数。

call 传入的参数数量不固定，跟 apply 相同的是，第一个参数也是代表函数体内的 this 指向，从第二个参数开始往后，每个参数被依次传入函数。

```js
var func1 = function(arg1, arg2) {};
```

就可以通过 `func1.call(this, arg1, arg2)` 或者 `func1.apply(this, [arg1, arg2])` 来调用。

所以，** `apply`第二个参数放的是数组形式 ** ，`results.push.array(results, value)` 这里的`value`就解释为 `[arg1, arg2]`，自然而然可以 `push` 到 `map`函数返回的每个实参，从而实现将嵌套数组转换为非嵌套的单一数据。

# 结语

学习每一个知识的时候，如果不去深究其原因，学到的只是表面的皮毛。

还有，函数式编程真的很优雅及强大~

# 参考

[javascript设计模式与开发实践](https://item.jd.com/11686375.html)
[javascript ES6 函数式编程入门经典](https://item.jd.com/12257861.html)
[关于红宝石书里的一个问题.push.apply()?](https://www.zhihu.com/question/60828411)
[如何理解和熟练运用js中的call及apply？](https://www.zhihu.com/question/20289071/answer/14745394)