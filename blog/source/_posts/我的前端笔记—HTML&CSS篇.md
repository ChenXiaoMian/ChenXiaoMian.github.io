---
title: 我的前端笔记—HTML&CSS篇
date: 2018-04-14
categories: 前端
tags: [meta, css3, 移动端]
---

## PC端基础meta标签

```html
<!-- 页面关键词-->
<meta name="keywords" content="your tags" />
<!-- 页面描述-->
<meta name="description" content="150 words" />
<!-- 搜索引擎索引方式：robotterms是一组使用逗号(,)分割的值，通常有如下几种取值：none，noindex，nofollow，all，index和follow。确保正确使用nofollow和noindex属性值。-->
<meta name="robots" content="index,follow" />
<!--
    all：文件将被检索，且页面上的链接可以被查询；
    none：文件将不被检索，且页面上的链接不可以被查询；
    index：文件将被检索；
    follow：页面上的链接可以被查询；
    noindex：文件将不被检索；
    nofollow：页面上的链接不可以被查询。
 -->
<!-- 页面重定向和刷新：content内的数字代表时间（秒），既多少时间后刷新。如果加url,则会重定向到指定网页（搜索引擎能够自动检测，也很容易被引擎视作误导而受到惩罚）。-->
<meta http-equiv="refresh" content="0;url=" />
<!-- 清除缓存 -->
<meta http-equiv="pragma" content="no-cache">
<meta http-equiv="cache-control" content="no-cache">
<meta http-equiv="expires" content="0">   
```

## H5基本meta标签

```html
<!-- 设置缩放 -->
<meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no, minimal-ui" />
<!-- 
    width    设置viewport宽度，为一个正整数，或字符串‘device-width’
    device-width  设备宽度
    height   设置viewport高度，一般设置了宽度，会自动解析出高度，可以不用设置
    initial-scale    默认缩放比例（初始缩放比例），为一个数字，可以带小数
    minimum-scale    允许用户最小缩放比例，为一个数字，可以带小数
    maximum-scale    允许用户最大缩放比例，为一个数字，可以带小数
    user-scalable    是否允许手动缩放
-->
<!-- 可隐藏地址栏，仅针对IOS的Safari（注：IOS7.0版本以后，safari上已看不到效果） -->
<meta name="apple-mobile-web-app-capable" content="yes" />
<!-- 仅针对IOS的Safari顶端状态条的样式（可选default/black/black-translucent ） -->
<meta name="apple-mobile-web-app-status-bar-style" content="black" />
<!-- IOS中禁用将数字识别为电话号码/忽略Android平台中对邮箱地址的识别 -->
<meta name="format-detection"content="telephone=no, email=no" />

<!-- 其他meta标签 -->

<!-- 启用360浏览器的极速模式(webkit) -->
<meta name="renderer" content="webkit">
<!-- 避免IE使用兼容模式 -->
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<!-- 针对手持设备优化，主要是针对一些老的不识别viewport的浏览器，比如黑莓 -->
<meta name="HandheldFriendly" content="true">
<!-- 微软的老式浏览器 -->
<meta name="MobileOptimized" content="320">
<!-- uc强制竖屏 -->
<meta name="screen-orientation" content="portrait">
<!-- QQ强制竖屏 -->
<meta name="x5-orientation" content="portrait">
<!-- UC强制全屏 -->
<meta name="full-screen" content="yes">
<!-- QQ强制全屏 -->
<meta name="x5-fullscreen" content="true">
<!-- UC应用模式 -->
<meta name="browsermode" content="application">
<!-- QQ应用模式 -->
<meta name="x5-page-mode" content="app">
<!-- windows phone 点击无高光 -->
<meta name="msapplication-tap-highlight" content="no">

```

## 移动端如何定义字体font-family

```css
@ --------------------------------------中文字体的英文名称
@ 宋体      SimSun
@ 黑体      SimHei
@ 微信雅黑   Microsoft Yahei
@ 微软正黑体 Microsoft JhengHei
@ 新宋体    NSimSun
@ 新细明体  MingLiU
@ 细明体    MingLiU
@ 标楷体    DFKai-SB
@ 仿宋     FangSong
@ 楷体     KaiTi
@ 仿宋_GB2312  FangSong_GB2312
@ 楷体_GB2312  KaiTi_GB2312  
@
@ 说明：中文字体多数使用宋体、雅黑，英文用Helvetica

body { font-family: Microsoft Yahei,SimSun,Helvetica; } 
```

## ios系统中元素被触摸时产生的半透明灰色遮罩怎么去掉

```css
a,button,input,textarea{-webkit-tap-highlight-color: rgba(0,0,0,0;)}
```

## 部分android系统中元素被点击时产生的边框怎么去掉

```css
a,button,input,textarea{
-webkit-tap-highlight-color: rgba(0,0,0,0;);
-webkit-user-modify:read-write-plaintext-only; 
}
```

## 禁止ios和android用户选中文字

```css
.css{-webkit-user-select:none}
```

## 打电话发短信写邮件怎么实现

```html
<a href="tel:0755-10086">打电话给:0755-10086</a>
<a href="sms:10086">发短信给: 10086</a>
<a href="mailto:peun@foxmail.com">peun@foxmail.com</a>
```

## css实现单行文本缩略显示

```css
overflow: hidden;
text-overflow:ellipsis;
white-space: nowrap;
/* 当然还需要加宽度width属来兼容部分浏览。*/
```

## css实现多行文本缩略显示

```css
display: -webkit-box;
-webkit-box-orient: vertical;
-webkit-line-clamp: 3;
overflow: hidden;
text-overflow: ellipsis;
word-break: break-all; /* 追加这一行代码 */
```

## placeholder的字体颜色大小

```css
input::-webkit-input-placeholder { 
    /* WebKit browsers */ 
    font-size:14px;
    color: #333;
}
input:focus::-webkit-input-placeholder{
    color:#EEEEEE;
}
input::-moz-placeholder { 
    /* Mozilla Firefox 19+ */ 
    font-size:14px;
    color: #333;
} 
input:-ms-input-placeholder { 
    /* Internet Explorer 10+ */ 
    font-size:14px;
    color: #333;
}
```

## 常见移动设备的 CSS3 Media Query 整理

```css
/**
 * iPhone 4/4s landscape & portrait
 */
@media only screen
and (min-device-width: 320px)
and (max-device-width: 480px)
and (-webkit-device-pixel-ratio: 2)
and (device-aspect-ratio: 2/3) {

}

/**
 * iPhone 4/4s landscape
 */
@media only screen
and (min-device-width: 320px)
and (max-device-width: 480px)
and (-webkit-device-pixel-ratio: 2)
and (device-aspect-ratio: 2/3)
and (orientation:landscape) {

}

/**
 *  iPhone 5/5s landscape & portrait
 */
@media only screen
and (min-device-width: 414px)
and (max-device-width: 736px)
and (-webkit-min-device-pixel-ratio: 3) {

}

/**
 * iPhone 6 Portrait
 */
@media only screen and (min-device-width: 375px) and (max-device-width: 667px) and (orientation : portrait) { 
    //iPhone 6 Portrait
}

/**
 * 宽度小于等于 320px 
 */
@media screen and (max-width: 320px) {
    
}

/**
 * 宽度大于等于 320px 
 */
@media screen and (min-width: 320px){
    
}
```

## CSS3 盒阴影

`box-shadow` 属性接受值最多由五个不同的部分组成。

`offset-x`  必需。水平阴影的位置。允许负值。
`offset-y`  必需。垂直阴影的位置。允许负值。
`blur`      可选。模糊距离。
`spread`    可选。阴影的尺寸。
`color`     可选。阴影的颜色。请参阅 CSS 颜色值。
`position`  可选。将外部阴影 (outset) 改为内部阴影。

```css
box-shadow: offset-x offset-y blur spread color position;
/* blur */
.right { box-shadow: 0px 0px 50px 0px rgba(0,0,0,0.5) }
/* 多重阴影 */
.foo {
    box-shadow: 20px 20px 10px 0px rgba(0,0,0,0.5) inset, /* inner shadow */
    20px 20px 10px 0px rgba(0,0,0,0.5); /* outer shadow */
}
/* 大杂烩 */
.simple {
    box-shadow: 0px 0px 0px 40px indianred;
}
.multiple {
    box-shadow: 20px 20px 0px 20px lightcoral,
                -20px -20px 0px 20px mediumvioletred,
                0px 0px 0px 40px rgb(200,200,200);
}
/* 弹出效果 */
.popup {
    transform: scale(1);
    box-shadow: 0px 0px 10px 5px rgba(0, 0, 0, 0.3);
    transition: box-shadow 0.5s, transform 0.5s;
}
.popup:hover {
    transform: scale(1.3);
    box-shadow: 0px 0px 50px 10px rgba(0, 0, 0, 0.3);
    transition: box-shadow 0.5s, transform 0.5s;
}
```

## CSS画出三角形

```css
.tip {
    border-color: transparent transparent rgb(0,0,0) transparent;
    border-width: 10px 100px 150px 100px;
    width: 0;
}
```

## CSS画出有边框的三角形

```css
.find-div-body {
    position: relative;
    top:30px;
    right:0px;
    width:400px;
    height:200px;
    padding:8px;
    background-color: #FFFFFF;
    border: #cccccc solid 1px;
    border-radius: 3px;
}
.find-div-body:before {
    box-sizing: content-box;
    width: 0px;
    height: 0px;
    position: absolute;
    top: -16px;
    right:41px;
    padding:0;
    border-bottom:8px solid #FFFFFF;
    border-top:8px solid transparent;
    border-left:8px solid transparent;
    border-right:8px solid transparent;
    display: block;
    content:'';
    z-index: 12;
}
.find-div-body:after {
    box-sizing: content-box;
    width: 0px;
    height: 0px;
    position: absolute;
    top: -18px;
    right:40px;
    padding:0;
    border-bottom:9px solid #cccccc;
    border-top:9px solid transparent;
    border-left:9px solid transparent;
    border-right:9px solid transparent;
    display: block;
    content:'';
    z-index:10
}
```

## 移动web 1像素边框

```css
.div::after {
    content: '';
    width: 200%;
    height: 200%;
    position: absolute;
    top: 0;
    left: 0;
    border: 1px solid #bfbfbf;
    border-radius: 4px;
    -webkit-transform: scale(0.5,0.5);
    transform: scale(0.5,0.5);
    -webkit-transform-origin: top left;
}
```

## flex布局

```css
/* ============================================================
   flex：定义布局为盒模型
   flex-v：盒模型垂直布局
   flex-1：子元素占据剩余的空间
   flex-align-center：子元素垂直居中
   flex-pack-center：子元素水平居中
   flex-pack-justify：子元素两端对齐
   兼容性：ios 4+、android 2.3+、winphone8+
   ============================================================ */
.flex{display:-webkit-box;display:-webkit-flex;display:-ms-flexbox;display:flex;}
.flex-v{-webkit-box-orient:vertical;-webkit-flex-direction:column;-ms-flex-direction:column;flex-direction:column;}
.flex-1{-webkit-box-flex:1;-webkit-flex:1;-ms-flex:1;flex:1;}
.flex-align-center{-webkit-box-align:center;-webkit-align-items:center;-ms-flex-align:center;align-items:center;}
.flex-pack-center{-webkit-box-pack:center;-webkit-justify-content:center;-ms-flex-pack:center;justify-content:center;}
.flex-pack-justify{-webkit-box-pack:justify;-webkit-justify-content:space-between;-ms-flex-pack:justify;justify-content:space-between;}
```

## CSS实现背景透明文字不透明

```css
.parent{
    background:rgba(0,0,0,0.2) none repeat scroll !important; /*实现FF背景透明，文字不透明*/
    background:#000;
    filter:Alpha(opacity=20);/*实现IE背景透明*/ 
    width:500px;
    height:500px;
    color:#F30;
    font-size:32px;
    font-weight:bold;
}
.parent p{
    position:relative; /*实现IE文字不透明*/
}
```

## IE各个版本hack

```css
/*类内部hack：*/
.header {_width:100px;}            /* IE6专用*/
.header {*+width:100px;}        /* IE7专用*/
.header {*width:100px;}            /* IE6、IE7共用*/
.header {width:100px\0;}        /* IE8、IE9共用*/
.header {width:100px\9;}        /* IE6、IE7、IE8、IE9共用*/
.header {width:330px\9\0;}    /* IE9专用*/

/*选择器Hack：*/
*html .header{}        /*IE6*/ 
*+html .header{}    /*IE7*/
```

## 用条件注释判断浏览器版本解决页面兼容问题

```html
<!DOCTYPE html> 
<html> 
<head> 
<title> 用条件注释判断浏览器版本,解决兼容问题 </title> 
<meta charset="utf-8"/> 
</head> 
<body> 
<!--[if IE]>只有IE6,7,8,9浏览器显示(IE10标准模式不支持)<hr/><![endif]--> 
<!--[if !IE]><!-->只有非IE浏览器显示(不包括IE10)<hr/><!--><![endif]--> 
<!--[if IE 9]>IE9浏览器显示<hr/><![endif]--> 
<!--[if IE 8]>IE8浏览器显示<hr/><![endif]--> 
<!--[if IE 7]>IE7浏览器显示<hr/><![endif]--> 
<!--[if IE 6]>IE6浏览器显示<hr/><![endif]--> 
<!--[if lt IE 10]>IE10以下版本浏览器显示(不包括IE10)<hr/><![endif]--> 
<!--[if lte IE 9]>IE9及IE9以下版本浏览器显示(包括IE9)<hr/><![endif]--> 
<!--[if gt IE 6]>IE6以上版本浏览器显示(不含IE6)<hr/><![endif]--> 
<!--[if gte IE 7]>IE7及IE7以上版本浏览器显示(包含IE7)<hr/><![endif]--> 
</body> 
</html>
```

## 参考

[【原】移动web资源整理](http://www.cnblogs.com/PeunZhang/p/3407453.html)
[面试的信心来源于过硬的基础](https://segmentfault.com/a/1190000013331105?utm_source=index-hottest#articleHeader12)
[box-shadow属性详解](http://www.iyangqiong.com/web/318.html)
[text-overflow:ellipsis 文字超出省略号代替不起作用解决方法](http://yunkus.com/text-overflow-ellipsis-do-not-work/)
[有谁能详细讲一下css如何画出一个三角形？怎么想都想不懂？](https://www.zhihu.com/question/35180018)
[移动web 1像素边框 瞧瞧大公司是怎么做的](https://segmentfault.com/a/1190000007604842)
[移动端重构系列4——重置样式](https://www.w3cplus.com/mobile/mobile-terminal-refactoring-reset-style.html)

注：笔记借鉴了其他文章，只用于个人学习收集，如有冒犯，请通知我。

将会持续更新~