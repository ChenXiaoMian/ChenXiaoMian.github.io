---
title: webpack3.0前端工程化
date: 2018-02-26
categories: webpack
tags: [webpack,前端,工程化]
---

## 核心概念

* Entry 入口
* Output 输出
* Loaders
* Plugins 插件

### 入口Entry

webpack.config.js
```javascript
module.exports = {
    entry: 'index.js'
}
```
```javascript
module.exports = {
    entry: ['index.js','vendors.js']
}
```
```javascript
module.exports = {
    entry: {
        index: ['index.js','app.js'],
        vendors: 'vendors.js',
    }
}
```

### 输出Output

打包成的文件(bundle)，一个或多个，可自定义规则

webpack.config.js
```javascript
module.exports = {
    entry: 'index.js',
    output: {
        filename: 'index.min.js'
    }
}
```
```javascript
const path = require('path')
module.exports = {
    entry: {
        index: 'index.js'
        vendors: 'vendors.js',
    },
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].min.[hash].js',
        // cdn
        publicPath: "http://cdn.example.com/assets/[hash]/"
    }
}
```

### 解析Resolve

设置模块如何被解析

webpack.config.js
```javascript
module.exports = {
    resolve: {
        // 设置别名
        alias: [
            xyz$: path.resolve(__dirname, 'path/to/file.js')
        ],
        // 自动解析确定的扩展
        extensions: ['.js','.vue']
    }
}
```
结果
```javascript
import Test1 from 'xyz';           // 精确匹配，所以 path/to/file.js 被解析和导入
import Test2 from 'xyz/file.js';   // 精确匹配，触发普通解析
import File from '../path/to/file' // 能够使用户在引入模块时不带扩展
```

### Loaders

处理文件，转换为模块

webpack.config.js
```javascript
module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/,
                use: 'css-loader'
            }
        ]
    }
}
```

常用loader

* 编译相关 babel-loader、ts-loader
* 样式相关 style-loader、css-loader、less-loader、postcss-loader
* 文件相关 file-loader、url-loader


### Plugins

参与打包整个过程、打包优化和压缩、配置编译时的变量

webpack.config.js
```javascript
const webpack = require('webpack');
module.exports = {
    plugins: [
        new webpack.optimize.UglifyJsPlugin()
    ]
}
```

常用plugins

* 优化相关 CommonsChunkPlugin(提取公共代码块)、UglifyjsWebpackPlugin(混淆压缩代码)
* 功能相关 ExtractTextPlugin(单独提取CSS文件)、HtmlWebpackPlugin(生成html)、HotModuleReplacementPlugin(热更新)、CopyWebpackPlugin(引用打包好的插件)

### 名词

* `Chunk` 代码块
* `Bundle` 捆，束，打包
* `Module` 模块

## 使用

* webpack命令
* webpack配置
* 第三方脚手架

### Babel-loader 使用

> * webpack 3.x | babel-loader 8.x | babel 7.x

```
npm install babel-loader@8.0.0-beta.0 @babel/core @babel/preset-env webpack
```

> * webpack 3.x babel-loader 7.x | babel 6.x

```
npm install babel-loader babel-core babel-preset-env webpack
```

```javascript
module: {
  rules: [
    {
      test: /\.js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env']
        }
      }
    }
  ]
}
```

### Babel Presets

* targets
* targets.browsers
* targets.browsers: “last 2 versions”  // 主流浏览器最后两个版本
* targets.browsers: “> 1%”             // 主流浏览器占有率大于1%

`browsers`设置，参考[browserslist](https://github.com/ai/browserslist)

数据参考[Can I use](https://caniuse.com/)

> * babel默认只转换语法，而不转换新的API，如需使用新的API，还需要使用对应的`转换插件`或者`polyfill`

### babel-polyfill

> * `babel-polyfill` 全局兼容ES6新特性，污染全局。全局垫片，为应用准备

```
npm install babel-polyfill --save
```

使用直接import

```javascript
import "babel-polyfill"
```

### babel-runtime & babel-plugin-transform-runtime

> * `babel-runtime` & `babel-plugin-transform-runtime` 无全局污染，依赖统一按需引入，无重复引入，无多余引入。局部垫片，为开发框架准备

```
npm install babel-plugin-transform-runtime --save-dev
npm install babel-runtime --save
```

使用`转换插件`在`.babelrc`文件设置

```json
{
  "presets": [
    ["env", {
      "modules": false,
      "targets": {
        "browsers": ["> 1%", "last 2 versions", "not ie <= 8"]
      }
    }]
  ],
  "plugins": ["transform-runtime"]
}

```

### [提取公用代码](https://doc.webpack-china.org/plugins/commons-chunk-plugin/)

* 减少代码冗余
* 提高加载速度

#### 场景

* 单页应用
* 单页应用 + 第三方依赖
* 多页应用 + 第三方依赖 + webpack生成代码

### [代码分割](https://doc.webpack-china.org/guides/code-splitting/)

* 分离业务代码和第三方依赖
* 分离业务代码和业务公共代码和第三方依赖
* 分离首次加载和访问后加载的代码

### [style-loader](https://doc.webpack-china.org/loaders/style-loader/)

`style-loader/url` 也可以添加一个URL `<link href="path/to/file.css" rel="stylesheet">`
```javascript
{
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          { loader: "style-loader/url" },
          { loader: "file-loader" }
        ]
      }
    ]
  }
}
```

`style-loader/useable` 是否调用这段 `<style></style>`

```javascript
{
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          { loader: "style-loader" },
          { loader: "css-loader" },
        ],
      },
      {
        test: /\.useable\.css$/,
        use: [
          {
            loader: "style-loader/useable"
          },
          { loader: "css-loader" },
        ],
      },
    ],
  },
}
```

component.js
```javascript
import style from './file.css'

style.use(); // = style.ref();
style.unuse(); // = style.unref();
```

样式不会添加在 `import/require()` 上，而是在调用 `use/ref` 时添加。如果 `unuse/unref` 与 `use/ref` 一样频繁地被调用，那么样式将从页面中删除。

### [css-loader](https://doc.webpack-china.org/loaders/css-loader/)

### [postcss-loader](https://doc.webpack-china.org/loaders/postcss-loader/)

* autoprefixer 添加前缀
* cssnano 压缩
* cssnext 使用新语法(css variable, custom selectors, calc())

#### 安装

```
npm install postcss postcss-loader autoprefixer cssnano postcss-cssnext --save-dev
```

#### 使用

webpack.config.js

```javascript
{
  test: /\.css$/,
  use: [
    'style-loader',
    'css-loader',
    {
      loader: 'postcss-loader',
      options: {
        ident: 'postcss',
        plugins: [
          require('postcss-cssnext')(),
          ...,
        ]
      }
    }
  ]
}
```

### Tree Shaking

* JS Tree Shaking
* CSS Tree Shaking

#### 场景

* 常规优化
* 引入第三方库的某一功能

[UglifyjsWebpackPlugin](https://doc.webpack-china.org/plugins/uglifyjs-webpack-plugin/)

[Purify Css](https://github.com/purifycss/purifycss)

[purifycss-webpack](https://github.com/webpack-contrib/purifycss-webpack)

### 文件处理

```
npm install file-loader url-loader img-loader postcss-sprite --save-dev
```

webpack.config.js

```javascript
{
  test: /\.(png|jpg|jpeg|gif)$/,
  use: [{
      loader: 'url-loader',
      options: {
        limit: 8192,
        name: './img/[hash].[ext]'
      }
  },{
      loader: 'file-loader',
      options: {
        outputPath: 'dist/',
        publicPath: '',
        useRelativePath: true
      }
  }]
}
```

#### 图片压缩[img-loader](https://github.com/thetalecrafter/img-loader)

webpack.config.js

```javascript
{
  test: /\.(png|jpg|jpeg|gif)$/,
  use: [{
      loader: 'url-loader',
      options: {
        limit: 8192,
        name: './img/[hash].[ext]'
      }
  },{
      loader: 'file-loader',
      options: {
        outputPath: 'dist/',
        publicPath: '',
        useRelativePath: true
      }
  },{
      loader: 'img-loader',
      options: {
        pngquant: {
          quality: 80
        }
      }
  }]
}
```

#### 雪碧图[postcss-sprites](https://github.com/2createStudio/postcss-sprites)

webpack.config.js

```javascript
{
  test: /\.css$/,
  use: [
    'style-loader',
    'css-loader',
    {
      loader: 'postcss-loader',
      options: {
        ident: 'postcss',
        plugins: [
          require('postcss-sprites')({
            spritePath: 'dist/assets/imgs/sprites',
            retina: true  // @2x结尾
          }),
          ...,
        ]
      }
    }
  ]
}
```

#### 处理第三方库的方式

* webpack.providePlugin
* [imports-loader](https://github.com/webpack-contrib/imports-loader)
* window

```
npm install imports-loader
```

webpack.config.js

```javascript
{
  test: path.resolve(__dirname, 'src/app.js'),
  use: [{
      loader: 'imports-loader',
      options: {
        $: 'jquery'
      }
  }]
}
```


## 参考

> * [webpack概念](https://doc.webpack-china.org/concepts/)