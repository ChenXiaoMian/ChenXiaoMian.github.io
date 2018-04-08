---
title: vue-cli 3.0配置webpack目录别名alias
date: 2018-04-08 23:28:00
categories: vue
tags: [vue,vue-cli,webpack,alias]
---

最近用vue脚手架新建工程的时候，发现`vue-cli`提供的是`3.0.0-beta.6`版本，安装完成之后也找不到`config`、`build`等目录，不懂要从哪里入手配置别名`alias`

看了下官方文档，简化成使用`vue.config.js`来配置项目，一路找到了`webpack`这一项，发现它可以使用了[webpack-chain](https://github.com/mozilla-neutrino/webpack-chain)链式API的调用方式，简化了对webpack配置的修改。

# 安装

```javascript
npm install -g @vue/cli
# or
yarn global add @vue/cli

vue create my-project
```

# 启动

```javascript
"scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build",
    "lint": "vue-cli-service lint"
}
```

# 配置

在根目录新建`vue.config.js`

```javascript
const path = require('path');
function resolve (dir) {
    return path.join(__dirname, dir)
}
module.exports = {
    lintOnSave: true,
    chainWebpack: (config)=>{
        // alias
        config.resolve.alias
            .set('@$', resolve('src'))
            .set('assets',resolve('src/assets'))
            .set('components',resolve('src/components'))
            .set('layout',resolve('src/layout'))
            .set('base',resolve('src/base'))
            .set('static',resolve('src/static'))
    }
}
```

# 参考

[vue-cli configuration](https://github.com/vuejs/vue-cli/blob/dev/docs/README.md)
[webpack-chain](https://github.com/mozilla-neutrino/webpack-chain)

谢谢您的品读，此处抛砖引玉，希望大家共同探讨学习。