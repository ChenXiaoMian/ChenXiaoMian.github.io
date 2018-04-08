---
title: vue-cli 3.0配置alias
date: 2018-04-08 23:28:00
categories: vue
tags: [vue,vue-cli,alias]
---

最近用vue脚手架新建工程的时候，发现`vue-cli`提供的是`3.0.0-beta.6`版本，安装完成之后也找不到`config`、`build`等目录。

看了下官方文档，发现配置都通过`vue.config.js`来设置，一路找到了`webpack`的配置项，它使用了[webpack-chain](https://github.com/mozilla-neutrino/webpack-chain)链接API生成，简化webpack配置的修改。

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