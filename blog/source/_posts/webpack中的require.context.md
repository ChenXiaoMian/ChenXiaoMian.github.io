---
title: webpack中的require.context
date: 2020-03-13
categories: webpack
tags: [webpack]
---

webpack 文档： [带表达式的 require 语句](https://webpack.docschina.org/guides/dependency-management/)

### 起因

```js
const req = require.context('./svg', false, /\.svg$/)
const requireAll = requireContext => requireContext.keys().map(requireContext)
requireAll(req);
```

如果你的 require参数含有表达式(expressions)，会创建一个上下文(context)，因为在编译时(compile time)并不清楚具体是哪一个模块被导入

```js
require("./template/" + name + ".ejs");
```

webpack 解析 require() 的调用，提取出来如下这些信息：

```js
Directory: ./template
Regular expression: /^.*\.ejs$/
```

则会返回template目录下的所有后缀为.ejs模块的引用，包含子目录。

### 参考

* [require.context](https://juejin.im/post/5ab8bcdb6fb9a028b77acdbd#heading-4)

