---
title: vue-music 移动端音乐webApp学习总结
date: 2018-02-02
categories: vue
tags: [vue,vuex,vue router,axios,jsonp]
---

# github地址

[vue-music](https://github.com/ChenXiaoMian/vue-music)

## 技术栈

- [x] vue-cli脚手架(vue + vuex + vue router + es6)
- [x] axios、jsonp
- [x] better-scroll、lyric-parser
- [x] vue-lazyload

## 完成组件

基础组件

- [x] confirm：确认对话框组件
- [x] listview：通讯录列表组件
- [x] loading：加载态组件
- [x] no-result：无结果展示组件
- [x] progress-bar：进度条组件
- [x] progress-circle：圆形进度条组件
- [x] scroll：移动端滚动组件
- [x] search-box：搜索框组件
- [x] search-list：搜索列表组件
- [x] slider：轮播图组件
- [x] switches：开关切换组件
- [x] top-tip：顶部消息提示组件
- [x] song-list：歌曲列表组件

业务组件

- [x] add-song：添加歌曲到列表组件
- [x] disc：歌单详情页组件
- [x] m-header：页面头部组件
- [x] music-list：歌曲列表页面组件
- [x] player：播放器内核组件
- [x] playlist：播放列表组件
- [x] rank：排行榜页面组件
- [x] recommend：推荐页面组件
- [x] search：搜索页面组件
- [x] singer：歌手页面组件
- [x] singer-detail：歌手详情页组件
- [x] suggest：搜索提示列表组件
- [x] tab：顶部导航栏组件
- [x] top-list：排行榜详情页组件
- [x] user-center：用户中心页组件

## 知识点整理

### 前端如何独立解决跨域问题？

解决跨域问题常用的解决方案有两个：

* JSONP：利用script标签可跨域的特点，在跨域脚本中可以直接回调当前脚本的函数。
* CORS：服务器设置HTTP响应头中Access-Control-Allow-Origin值，解除跨域限制。

但是这两个跨域方案都存在一个致命的缺陷，严重依赖后端的协助。

如何独立解决跨域问题？

一般的做法是通过本地配置nginx反向代理进行处理的，除此之外，还可以通过nodejs来进行代理接口。

#### JSONP原理及封装

[jsonp原理](https://www.cnblogs.com/dowinning/archive/2012/04/19/json-jsonp-jquery.html)其实就是通过`<script>`标签`src`属性的跨域能力，在远程服务器上设法把数据装进json格式的文件里，供客户端调用和进一步处理，客户端在对JSON文件调用成功之后，也就获得了自己所需的数据，剩下的就是按照自己需求进行处理和展现了。

jsonp封装
```javascript
// npm install jsonp --save
import originJSONP from 'jsonp'

// url拼接参数
export default function jsonp(url,data,option){
    url += (url.indexOf('?') < 0 ? '?' : '&') + param(data);
    // 返回promise对象
    return new Promise((resolve,reject)=>{
        originJSONP(url,option,(err,data)=>{
            if(!err){
                resolve(data);
            }else{
                reject(err);
            }
        })
    })
}
// obj对象拼接成字符串
function param(data){
    let url = '';
    for(var k in data){
        let value = data[k] !== undefined ? data[k] : '';
        url += `&${k}=${encodeURIComponent(value)}`;
    }
    return url ? url.substring(1) : ''
}
```

#### nodejs代理接口

请求数据遇到hosts和referer限制？前端是无法修改request.headers，因此可以通过nodejs手动代理请求。

```javascript
var express = require('express');
var axios = require('axios');

var app = express();
var apiRoutes = express.Router();

apiRoutes.get('/getDiscList',function(req,res){
  var url = 'https://c.y.qq.com/splcloud/fcgi-bin/fcg_get_diss_by_tag.fcg';
  axios.get(url,{
    headers: {
      referer: 'https://y.qq.com/portal/playlist.html',
      host: 'y.qq.com'
    },
    params: req.query
  }).then((response)=>{
    res.json(response.data);
  }).catch((e)=>{
    console.log(e);
  });
});

app.use('/api',apiRoutes);
```

参考：
> * [nodejs服务实现反向代理，解决本地开发接口请求跨域问题](https://www.cnblogs.com/canfoo/p/6912306.html)
> * [前端开发如何独立解决跨域问题](https://segmentfault.com/a/1190000010719058)

### 通讯录组件右侧快速入口(数组排序)

开发歌手列表的时候，需要按热门及字母的顺序来显示，右侧则需要一个快速入口的索引列表（类似通讯录）。思路就是先将歌手数据前10条添加到热门数组中，另外将剩余的数据按字母归类放置在各自的数组里，最后对字母进行排序操作。

```javascript
_normalizeSinger (list) {
    let map = {
        hot: {
            title: HOT_NAME,
            items: []
        }
    }
    list.forEach((item,index)=>{
    // 取前10条添加到热门数组
    if(index < HOT_SINGER_LEN){
        map.hot.items.push(new Singer({
            id: item.Fsinger_mid,
            name: item.Fsinger_name
        }))
    }
    // 按字母归类
    const key = item.Findex
    if(!map[key]){
        map[key] = {
            title: key,
            items: []
        }
    }
    map[key].items.push(new Singer({
        id: item.Fsinger_mid,
        name: item.Fsinger_name
    }))
    })
    // 处理map得到有序列表
    let hot = [];
    let ret = [];
    for(let key in map){
        let val = map[key]
        if(val.title.match(/[a-zA-Z]/)){
            ret.push(val)
        }else if(val.title === HOT_NAME){
            hot.push(val)
        }
    }
    // 排序操作
    ret.sort((a, b)=>{
        return a.title.charCodeAt(0) - b.title.charCodeAt(0)
    })

    return hot.concat(ret);
}
```

### 工厂模式运用（歌手数据，歌曲数据）

歌手页面循环数据的时候，需要获取每个歌手的姓名，ID及头像，这时候就运用到工厂模式。歌曲数据类也一样，多了`getLyric()`处理歌词的原型继承方法。

```javascript
// singer.js
export default class Singer {
    constructor({id, name}) {
        this.id = id
        this.name = name
        this.avatar = `https://y.gtimg.cn/music/photo_new/T001R300x300M000${id}.jpg?max_age=2592000`
    }
}
// song.js
export default class Song{
    constructor({id,mid,singer,name,album,duration,image,url}){
        this.id = id,
        this.mid = mid,
        this.singer = singer,
        this.name = name,
        this.album = album,
        this.duration = duration,
        this.image = image,
        this.url = url
    }
    getLyric () {
        if (this.lyric) {
            return Promise.resolve(this.lyric)
        }
        return new Promise((resolve,reject)=>{
            getLyric(this.mid).then((res)=>{
                if(res.retcode === ERR_OK){
                    this.lyric = Base64.decode(res.lyric)
                    resolve(this.lyric)
                }else{
                    reject('no lyric')
                }
            })
        })
    }
}
```

[constructor](http://es6.ruanyifeng.com/#docs/class#constructor-方法)方法是类的默认方法，通过new命令生成对象实例时，自动调用该方法。

构造函数的prototype属性，在 ES6 的“类”上面继续存在。事实上，类的所有方法都定义在类的prototype属性上面。

```javascript
class Point {
  constructor() {
    // ...
  }

  toString() {
    // ...
  }

  toValue() {
    // ...
  }
}

// 等同于

Point.prototype = {
  constructor() {},
  toString() {},
  toValue() {},
};
```

在类的实例上面调用方法，其实就是调用原型上的方法。具体参照[Class 的基本语法](http://es6.ruanyifeng.com/#docs/class)


### vue组件之间的通信（scroll组件为例）

#### 子组件向父组件派生事件

```javascript
// 子组件
import BScroll from 'better-scroll'
export default {
    props: {
        probeType: {
            type: Number,
            default: 1
        },
        click: {
            type: Boolean,
            default: true
        },
        data: {
            type: Array,
            default: null
        },
        listenScroll: {
            type: Boolean,
            default: false
        },
        pullup: {
            type: Boolean,
            default: false
        },
        beforeScroll: {
            type: Boolean,
            default: false
        },
        refreshDelay: {
            type: Number,
            default: 20
        }
    },
    mounted () {
        setTimeout(() => {
            this._initScroll()
        },20)
    },
    methods: {
        _initScroll () {
            if(!this.$refs.wrapper){
                return
            }
            this.scroll = new BScroll(this.$refs.wrapper,{
                probeType: this.probeType,
                click: this.click
            })
            if(this.listenScroll){
                let me = this
                this.scroll.on('scroll',(pos)=>{
                    me.$emit('scroll',pos)
                })
            }
            if(this.pullup){
                this.scroll.on('scrollEnd',()=>{
                    if(this.scroll.y <= (this.scroll.maxScrollY + 50)) {
                        this.$emit('scrollToEnd')
                    }
                })
            }
            if(this.beforeScroll){
                this.scroll.on('beforeScrollStart',()=>{
                    this.$emit('beforeScroll')
                })
            }
        },
        enable () {
            this.scroll && this.scroll.enable()
        },
        disable () {
            this.scroll && this.scroll.disable()
        },
        refresh () {
            this.scroll && this.scroll.refresh()
        },
        scrollTo () {
            this.scroll && this.scroll.scrollTo.apply(this.scroll,arguments)
        },
        scrollToElement () {
            this.scroll && this.scroll.scrollToElement.apply(this.scroll,arguments)
        }
    },
    watch: {
        data () {
            setTimeout(()=>{
                this.refresh()
            },this.refreshDelay)
        }
    }
}
```
```html
<!-- 父组件 -->
<scroll class="listview" :data="data" ref="listview" :listenScroll="listenScroll" @scroll="scroll" :probeType="probeType">
<!-- ... -->
</scroll>
```

#### 父组件调用子组件方法

```javascript
methods: {
    refresh () {
        this.$refs.listview.refresh()
    }
}
```

## vscode+eslint用法

不管是多人合作还是个人项目，代码规范是很重要的。这样做不仅可以很大程度地避免基本语法错误，也保证了代码的可读性。

首先在vscode安装eslint插件，安装并配置完成 ESLint 后，我们继续回到 VSCode 进行扩展设置，依次点击 文件 > 首选项 > 设置 打开 VSCode 配置文件,添加如下配置

```json
{
    "workbench.colorTheme": "Default Light+",
    "files.autoSave":"off",
    "eslint.validate": [
       "javascript",
       "javascriptreact",
       "html",
       { "language": "vue", "autoFix": true }
     ],
     "eslint.options": {
        "plugins": ["html"]
     },
     // 这样每次保存的时候就可以根据根目录下.eslintrc.js你配置的eslint规则来检查和做一些简单的fix。
     "eslint.autoFixOnSave": true
}
```

参考:

> * [ESLint的用法](https://juejin.im/post/59097cd7a22b9d0065fb61d2#heading-8)
> * [vscode插件和配置推荐](https://github.com/varHarrie/YmxvZw/issues/10)

## 总结一下

由于课程内容太多，很多细节也没法做记录，只记录了印象比较深刻的几个点，另外的部分只能等回头想起来或者用到的时候翻源码或者视频来巩固了。这门课让我收获了很多，感觉做起项目来思路也清晰了不少，特别是组件的拆分，vuex的使用，动画交互及很多工具类的写法。嗯，文采有限...

谢谢您的品读，此处抛砖引玉，希望大家共同探讨学习。