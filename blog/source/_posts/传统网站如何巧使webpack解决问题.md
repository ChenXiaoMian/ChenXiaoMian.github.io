---
title: 传统网站如何巧使webpack解决问题
date: 2018-04-20 10:23:32
categories: webpack
tags: [webpack, 打包]
line_number: false
---

# 前言

哈哈，感觉标题有点大了，有标题党的嫌疑，但实在想不出什么合适的名字。

昨天产品提出一个需求，就是修改现有移动端网站下载APP的提示样式。是的，这是个简单的需求，没什么值得好提的。但就是有个问题，由于现在网站还是以后端为主导，没做分离，很多层次结构比较混乱。而且一改是改三个网站，涉及到三个网站要更新`html，css，js，img`，额......感觉有点头大。

类似这样的需求：

{% asset_img 1.jpg 资讯界面加下载提示 %}

本来想硬着头皮做完一遍复制，替换就好。后来想到是不是可以用`webpack`将`html，css，js，img`打包成一个js文件呢，于是开始了尝试。

# 尝试

目录结构：
```bash
- build-test
    - example                       # 代码开发目录
        downApp.js                  # 逻辑代码
        downApp.min.js            # 打包代码
        index.css                 # 引用样式
        index.html                # html结构
        js.cookie.js              # 引用cookie
        km-downapp-close.png      # 关闭按钮图片
        km-downapp-logo.png       # logo图片
    + node_modules       #所使用的nodejs模块
    package.json         #项目配置
    package-lock.json
    webpack.config.js    #webpack配置
    README.md            #项目说明
```

1、新建目录 `mkdir build-test`，进入目录 `cd build-test`。

2、执行 `npm init --yes` 快速初始化项目。

3、安装webpack`npm install --save-dev webpack`，各种loader `npm install --save-dev css-loader style-loader url-loader`，js压缩插件 `npm install --save-dev uglifyjs-webpack-plugin`。

4、新建webpack配置

`webpack.config.js`

```js
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')

module.exports = {
    entry: {
        index : './example/downApp.js'
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                  { loader: "style-loader" },
                  { loader: "css-loader" },
                ]
            },
            // 将css中引用url路径的图片转成base64（字节数小于10000的图片）
            {
                test: /\.(png|jpg|gif|svg|eot|ttf|woff|woff2)$/,
                loader: 'url-loader',
                options: {
                  limit: 10000
                }
            }
        ]
    },
    plugins: [
        // 压缩js
        // new UglifyJsPlugin()
    ],
    output: {
        filename: './example/downApp.min.js'
    }
}
```

5、新建目录 `mkdir example`，进入目录 `cd example`

6、新建页面`index.html`，配置一些基本结构。

`index.html`

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>demo页面</title>

    <script type="text/javascript">
        document.documentElement.style.fontSize = document.documentElement.clientWidth / 7.50 + 'px';
    </script>
    <style type="text/css" media="screen">
    body{margin:0;}
    </style>
</head>
<body>
    <div id="km-header"></div>
    <script type="text/javascript" src="http://m.kmzyw.com.cn/resources/js/jquery.min.js"></script>
    <script src="downApp.min.js"></script>
</body>
</html>
```

7、主要逻辑 `downApp.js`

`downApp.js`

```js
import './index.css';
import Cookies from './js.cookie.js';
$(function(){
    function getRequest(){
      var u = location.search;
      var p = new Object();
      if (u.indexOf("?") != -1) {
          var str = u.substr(1);
          strs = str.split("&");
          for (var i = 0; i < strs.length; i++) {
               p[strs[i].split("=")[0]] = decodeURI(strs[i].split("=")[1]);
          }
      }
      return p;
    }
    var reqObj = getRequest();
    // 判断是否APP访问
    if(reqObj.from != 'app' && !reqObj.from){
        var downValues = Cookies("__closeapp");
        var downDom = '<div class="km-downapp-wrap"><a href="http://m.kmzyw.com.cn/resources/html/app-download.html"class="km-downapp-link"><div class="km-downapp-logo"></div><div class="km-downapp-text"><p class="font-large">康美中药城APP新版上线</p><p>随时随地，药材服务不离线</p></div><div class="km-downapp-button">立即下载</div></a><a href="javascript:void(0);"class="km-downapp-close"></a></div>';
        if (downValues != '1') {
            $("#km-header").before(downDom);
        }
    }
    $("body").on("click", ".km-downapp-close", function(){
        $(".km-downapp-wrap").remove();
        Cookies('__closeapp', '1',{ expires: 1});
    });
});
```

8、样式表 `index.css`

`index.css`

```css
.km-downapp-wrap{position:relative;width:100%;height:1rem;background:#e7666e;z-index:9999;-webkit-box-sizing:border-box;box-sizing:border-box;overflow:hidden;display:-webkit-box;display:-webkit-flex;display:-ms-flexbox;display:flex;-webkit-box-align:center;-webkit-align-items:center;-ms-flex-align:center;align-items:center;font-family: -apple-system, BlinkMacSystemFont, "PingFang SC","Helvetica Neue",STHeiti,"Microsoft Yahei",Tahoma,Simsun,sans-serif;}
.km-downapp-logo{margin-right: 0.24rem;width:0.7rem;height:0.7rem;background:url('./km-downapp-logo.png') no-repeat;background-size:100% 100%;}
.km-downapp-link{display:-webkit-box;display:-webkit-flex;display:-ms-flexbox;display:flex;text-decoration:none;margin-left: 0.3rem;-webkit-box-align:center;-webkit-align-items:center;-ms-flex-align:center;align-items:center;}
.km-downapp-link:hover,.km-downapp-link:active{text-decoration:none;}
.km-downapp-text>p{font-size:0.2rem;font-weight:normal;color:#f5cbcc;line-height: 0.28rem;margin:0;}
.km-downapp-text .font-large{font-size:0.32rem;color:#fff;line-height: 0.4rem;}
.km-downapp-button{display:block;width: 1.7rem;height: 0.6rem;background:#fff;-webkit-border-radius:0.3rem;border-radius:0.3rem;font-size:0.28rem;line-height:0.6rem;text-decoration:none;color:#ea4d58;text-align:center;-webkit-tap-highlight-color: rgba(255,255,255,.35);      -webkit-touch-callout: none;margin-left:0.36rem;}
.km-downapp-close{display:block;width:0.28rem;height:0.28rem;background:url('./km-downapp-close.png') no-repeat;background-size:100% 100%;margin-left: 0.2rem;}
```

9、跳回`/build-test`目录，执行 `webpack` 命令，生成 `downApp.min.js` 文件，测试结果，大功告成！（可以开启压缩JS插件减少文件大小）

打包结果

{% asset_img 2.jpg 打包结果 %}

# 总结

其实文章内容很简单，用的还是基本的webpack打包功能，关键是看你怎么用。如果是单页应用，打包成一个js会遇到白屏问题，为了避免这个还会把css文件单独分离出来。但对于传统项目年久失修的复杂结构来说的话，不失为一个好的解决方法。

谢谢您的品读，此处抛砖引玉，希望大家共同探讨学习。