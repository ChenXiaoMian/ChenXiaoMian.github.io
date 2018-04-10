---
title: node.js常用知识
date: 2018-04-03
categories: node
tags: [node,path,回调]
---

## `process.cwd()`与`__dirname`的区别

`process.cwd()` 是当前执行node命令时候的文件夹地址 ——工作目录，保证了文件在不同的目录下执行时，路径始终不变。
`__dirname` 是被执行的js 文件的地址 ——文件所在目录

## `path.join()`和`path.resolve()`的区别

`path.join()` 方法使用平台特定的分隔符把全部给定的 `path` 片段连接到一起，并规范化生成的路径。

```javascript
path.join('/foo', 'bar', 'baz/asdf', 'quux', '..');
// 返回: '/foo/bar/baz/asdf'
```

`path.resolve()` 方法会把一个路径或路径片段的序列解析为一个绝对路径。其处理方式类似于对这些路径逐一进行cd操作

```javascript
path.resolve('/foo/bar', './baz')
// 输出结果为
'/foo/bar/baz'
path.resolve('/foo/bar', '/tmp/file/')
// 输出结果为
'/tmp/file'
```
```javascript
path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif')
// 当前的工作路径是 /home/itbilu/node，则输出结果为
'/home/itbilu/node/wwwroot/static_files/gif/image.gif'
```

### 对比

```javascript
const path = require('path');
let myPath = path.join(__dirname,'/img/so');
let myPath2 = path.join(__dirname,'./img/so');
let myPath3 = path.resolve(__dirname,'/img/so');
let myPath4 = path.resolve(__dirname,'./img/so');
console.log(__dirname);           //D:\myProgram\test
console.log(myPath);     //D:\myProgram\test\img\so
console.log(myPath2);   //D:\myProgram\test\img\so
console.log(myPath3);   //D:\img\so<br>
console.log(myPath4);   //D:\myProgram\test\img\so
```

## 全局变量

```javascript
global.xxx = 123;
```

## process进程(常用)

* `process.argv` 属性返回一个数组，包含了启动Node.js进程时的命令行参数
* `process.argv0` 属性，保存Node.js启动时传入的argv[0]参数值的一份只读副本。
* `process.execArgv` 属性返回当Node.js进程被启动时，Node.js特定的命令行选项
* `process.execPath` 属性，返回启动Node.js进程的可执行文件所在的绝对路径

```javascript
const {argv, argv0, execArgv, execPath} = process;
argv.forEach(item => {
    console.log(item);
});
console.log("argv0:"+argv0);
console.log("execArgv:"+execArgv);
console.log("execPath:"+execPath);
```

执行`node --harmony process.js --version`输出：

```bash
C:\Program Files\nodejs\node.exe
E:\study\nodejs\process.js
--version
argv0:C:/Program Files/nodejs/node.exe
execArgv:--harmony
execPath:C:\Program Files\nodejs\node.exe
```

## 事件队列

setTimeout()、setInterval()、setImmediate()、process.nextTick()

nextTick.js

```javascript
setImmediate(() => {
    console.log('setImmediate延迟执行');
});
setTimeout(() => {
    console.log('timeout');
}, 0);
process.nextTick(() => {
    console.log('nextTick延迟执行');
});
console.log('正常执行');
```

执行`node nextTick.js`输出：

```bash
正常执行
nextTick延迟执行
timeout
setImmediate延迟执行
```

结果看出，`process.nextTick()`中的回调函数执行的优先级要高于setImmediate()。原因在于事件循环对观察者的检查是有先后顺序的，`process.nextTick()`属于idle观察者，`setImmediate()`属于check观察者。在每一个轮循环检查中，idle观察者先于I/O观察者，I/O观察者先于check观察者。

## 解决地狱回调的方法

1、promise

```javascript
const fs = require('fs');
const promisify = require('util').promisify;
const read = promisify(fs.readFile);
read('./promisify.js').then(data => {
    console.log(data.toString());
}).catch(ex => {
    console.log(ex);
});
```

2、async

```javascript
const fs = require('fs');
const promisify = require('util').promisify;
const read = promisify(fs.readFile);
async function test(){
    try{
        const content = await read('./promisify.js');
        console.log(content.toString());
    }catch(ex){
        console.log(ex);
    }
}
```

## 参考

[Node.js 中文网](http://nodejs.cn/api/)

本文将会持续更新~