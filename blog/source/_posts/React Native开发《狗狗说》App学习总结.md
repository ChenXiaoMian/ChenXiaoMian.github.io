---
title: React Native开发《狗狗说》App学习总结
date: 2017-12-08 15:34:05
categories: react native
tags: [react,react native,app,android]
---

# github地址

[react-native-dogapp](https://github.com/ChenXiaoMian/react-native-dogapp)

# 搭建React脚手架

- [x] 可以解析JSX语法
- [x] 配置babel编译ES6语法
- [x] 实现代码的热替换，浏览器实时刷新查看效果
- [x] 支持SCSS预处理器
- [x] 编译完成自动打开浏览器
- [x] 支持图片、图标字体等资源的编译
- [x] 区分开发环境和生产环境

# 运行项目

```bash
# 克隆项目
git clone https://github.com/ChenXiaoMian/react-o2o-app.git
# 进入目录
cd react-o2o-app
# 安装依赖
npm install
# 运行开发环境 访问 http://localhost:8080/
npm run dev
# 运行开发环境 Mock后台数据接口 访问 http://localhost:3000
npm run server
# 打包发布
npm run dist
```

# 学习参考教程

> * [React 模仿大众点评 webapp 手记](http://www.imooc.com/article/16082)
> * [webpack](https://doc.webpack-china.org/guides/getting-started/)搭建开发环境及生产环境。
> * [sass](https://www.sass.hk/guide/)css预处理
> * [react](https://reactjs.org/docs/state-and-lifecycle.html)运用组件化开发思想、虚拟DOM，
> * [react-route](https://reacttraining.com/react-router/web/example/basic)配套路由
> * [redux](http://www.redux.org.cn/)数据状态管理，[阮一峰Redux 入门教程](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html)
> * [react-redux](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_three_react-redux.html)React-Redux 的用法
> * [react-swipe](https://github.com/voronianski/react-swipe)轮播图插件
> * [初探 React Router 4.0](http://www.jianshu.com/p/e3adc9b5f75c/)
> * [react-router 4.0 下服务器如何配合BrowserRouter](http://www.cnblogs.com/YZH-chengdu/p/6855237.html)

# 完成功能

- [x] 开发首页
- [x] 开发城市页面
- [x] 开发搜索结果页
- [x] 开发商户详情页
- [x] 开发登录页
- [x] 收藏功能
- [x] 开发用户中心页
- [x] 评价功能

# 要点小记

组件化开发的思想主要体现在代码分离的三个层上，分别是page(页面层)，subpage(智能组件)，component(木偶组件)。木偶组件主要根据props传值的内容来显示，智能组件主要是获取数据的逻辑代码。

## 基本概念

* `react-router` React Router 核心
* `react-router-dom` 用于 DOM 绑定的 React Router
* `react-router-native` 用于 React Native 的 React Router
* `react-router-redux` React Router 和 Redux 的集成

## 路由配置

```javascript
<HashRouter>
    <div>
      <Switch>
        <Route exact path="/" component={Home}/>
        <Route path="/city" component={City}/>
        <Route path="/search/:category/:keyword?" component={Search}/>
        <Route path="/detail/:id" component={Detail}/>
        <Route path="/login/:router?" component={Login}/>
        <Route path="/user" component={User}/>
        <Route component={NotFound}/>
      </Switch>
    </div>
</HashRouter>
```
### 1、`<BrowserRouter>`和`<HashRouter>`

在react-router 4.0版本中，API与先前版本相比有了很大的修改，在2.0、3.0中常用的`<Router>`组件作为路由底层配置组件不再常用，取而代之的是四个各有不同的路由组件：

* `<BrowserRouter>`一个使用了 HTML5 history API 的高阶路由组件，保证你的 UI 界面和 URL 保持同步。
* `<HashRouter>`不支持 location.key 和 location.state。另外由于该技术只是用来支持旧版浏览器，因此更推荐大家使用 BrowserRouter。
* `<MemoryRouter>`组件在内存中保存“URL”信息，不会修改浏览器的地址栏，往往用于React Native或测试环境等非浏览器环境。
* `<StaticRouter>`组件从名字能看出它从不修改路由，这在服务器端渲染时很有用。

`<BrowserRouter>`和`<HashRouter>`都可以实现前端路由的功能，区别是前者基于url的`pathname`段，后者基于`hash`段。

前者：`http://127.0.0.1:3000/article/num1`

后者：`http://127.0.0.1:3000/#/article/num1（不一定是这样，但#是少不了的）`

这样的区别带来的直接问题就是当处于二级或多级路由状态时，刷新页面，`<BrowserRouter>`会将当前路由发送到服务器（因为是pathname），而`<HashRouter>`不会（因为是hash段）。

在react-router 4.0 的文档中有这样一段话：

> 注意： 使用 hash 的方式记录导航历史不支持 location.key 和 location.state。 在以前的版本中，我们为这种行为提供了 shim，但是仍有一些问题我们无法解决。 任何依赖此行为的代码或插件都将无法正常使用。 由于该技术仅用于支持传统的浏览器，因此在用于浏览器时可以使用 `<BrowserHistory>` 代替。

### 2、react-router 还是 react-router-dom？

在 React 的使用中，我们一般要引入两个包，`react` 和 `react-dom`，那么 `react-router` 和 `react-router-dom` 是不是两个都要引用呢？
非也，坑就在这里。他们两个只要引用一个就行了，不同之处就是后者比前者多出了 `<Link>` `<BrowserRouter>` 这样的 DOM 类组件。
因此我们只需引用 `react-router-dom` 这个包就行了。当然，如果搭配 `redux` ，你还需要使用 `react-router-redux`。

### 3、path可选url-params

```javascript
<Route path="/search/:category/:keyword?" component={Search}/>
```

在开发搜索组件路由的时候，路由参数有两个参数或者三个参数的情况。这时候`/:keyword?`是可选的参数，只需要在末尾加个`?`号。


### 4、history.push

学习教程的时候，history是通过`import { history } from 'react-router'`引入的。4.0以上没找到这个对象，最后发现在页面组件的`this.props.history`找到了。参照[初探 React Router 4.0](http://www.jianshu.com/p/e3adc9b5f75c/) 一文得到的。


## 列表加载更多

首先需要准备3个状态

```javascript
this.state = {
    data : [],  //存储数据
    hasMore : false,  //记录是否还有更多数据
    isLoadingMore : false,  //是否正在加载
    page : 1  //记录下一页的页码
}
```

然后，还需要加一个loadMoreData的方法，即在点击“加载更多”时会触发的方法。加载首页数据和加载更多数据，这两个函数可以提取一些公共代码，具体的写法可以是：

```javascript
loadFirstPageData() {
    // 加载首页数据，result
    // 处理数据
    this.resultHandle(result)
}
loadMoreData() {
    // 加载下一页的数据，result
    // 处理
    this.resultHandle(result)
}
resultHandle() {
    // 解析数据，更改 state
}
```

以上都是在引入LoadMore组件（该组件后面会教大家创建，现在还没创建）之前需要做的准备工作，那么这些准备工作该怎么用到`LoadMore`组件中呢？通过以下代码来体会一下。

```javascript
{
    this.state.hasMore
    ? <LoadMore isLoadingMore={this.state.isLoadingMore} loadMoreFn={this.loadMoreData.bind(this)}/>
    : <div></div>
}
```

总结一下以上的准备数据在LoadMore组件中的应用。

* hasMore控制组件的显示和隐藏
* isLoadingMore控制组件是显示“加载中...”（此时点击失效）还是“点击加载更多”
* loadMoreData函数会在点击组件时触发，并加载下一页数据
* page记录下一页的页码，会在loadMoreData函数中使用并累加

创建LoadMore组件，并用上上面传递过来的数据。

```javascript
render() {
    return (
        <div className="load-more" ref="wrapper">
        {
            this.props.isLoadingMore
            ? <span>加载中...</span>
            : <span onClick={this.loadMoreHandle.bind(this)}>点击加载更多</span>
        }
        </div>
    )
}
loadMoreHandle () {
    this.props.loadMoreFn()
}
```

如何实现上拉加载效果，上面代码中有`ref="wrapper"`，实现思路是：监控 window 的`scroll`方法，然后获取`ref="wrapper"`的DOM，利用`getBoundingClientRect()`方法获得距离顶部的高度，然后看是否触发 `loadMore`方法。

```javascript
componentDidMount () {
    const loadMoreFn = this.props.loadMoreFn
    const wrapper = this.refs.wrapper
    // console.log(wrapper)
    let timeoutId
    function callback(){
        const top = wrapper.getBoundingClientRect().top
        const windowHeight = window.screen.height
        if(top && top < windowHeight){
            // 当wrapper已经被滚动到暴露在页面可视范围之内的时候，触发加载更多
            loadMoreFn()
        }
    }

    window.addEventListener('scroll',function(){
        if(this.props.isLoadingMore){
            return
        }
        if(timeoutId){
            clearTimeout(timeoutId)
        }
        timeoutId = setTimeout(callback,100)
    }.bind(this),false)
}
```

## 让`<br/>`换行

有`<br />`的代码在页面中不换行，而是直接显示`<br />`。`dangerouslySetInnerHTML`, 让React正常显示你的html代码 这个`prop`的命名是故意这么设计的，以此来警告，它的prop值（一个对象而不是字符串）应该被用来表明净化后的数据。

```javascript
<div className="headlineText" dangerouslySetInnerHTML={{__html: data.lineText}}></div>
```

## 搜索结果列表

如果在搜索结果页头部的输入框中再次输入内容重新进行搜索时，就需要多一步处理。

```javascript
componentDidUpdate (prevProps,prevState) {
    const keyword = this.props.keyWord
    const category = this.props.category
    // 搜索条件完全相等时，忽略
    if(keyword === prevProps.keyWord && category === prevProps.category){
        return
    }

    // 重置state
    this.setState(initialState)
    // 重新加载数据
    this.loadFirstPageData()
}
```

这里需要理解`componentDidMount`和`componentDidUpdate`两个生命周期的不同。

* 页面初次渲染，会走`componentDidMount`
* 页面再次渲染，就不会走`componentDidMount`，而只走`componentDidUpdate`

## 登录后的跳转

```javascript
<Route path="/login/:router?" component={Login}/>
```

在路由配置中，登录组件也增加了一个可选参数`router`即登录之后需要跳转的页面。即在哪个页面登录的，登录完了之后还要再跳转到哪个页面，这种功能很常见。

```javascript
// 读取Redux中是否有用户信息
doCheck (){
    const userinfo = this.props.userinfo
    if(userinfo.username){
        // 已经登录
        this.goUserPage()
    }else{
        // 尚未登录
        this.setState({
            checking : false
        })
    }
}
// 登录成功之后的处理
loginHandle (username) {
    if(username.trim()==''){
        alert('手机号不能为空')
        return
    }
    // 保存用户名
    const actions = this.props.userInfoActions   //存储用户id到Redux
    let userinfo = this.props.userinfo
    userinfo.username = username
    actions.login(userinfo)
    // 跳转链接
    const params = this.props.match.params
    const history = this.props.history
    if(params.router){
        // 跳转到指定的页面
        history.push('/'+decodeURIComponent(params.router))
    }else{
        // 跳转到默认页面-用户中心
        this.goUserPage()
    }
}
```

在商户详情页立即购买按钮的事件上，先判断是否登录，如果没登录则跳到登录页(且带上可调回的路由参数)

```javascript
// 验证登录
loginCheck () {
    const id = this.props.id
    const userinfo = this.props.userinfo
    const history = this.props.history
    if(!userinfo.username){
        history.push('/login/'+encodeURIComponent('detail/'+id))
        return false
    }
    return true
}
```

# 最后

这算是在学习react期间，完成的一个比较完整的项目。文章主要是记录比较重要的要点，分享一下自己的收获，遇坑的点在react-router 4.0上，版本更新太快，视频用的许多方法都用不上，还有redux和react-redux的概念比较难理解，这里推荐一篇文章[React-Redux](https://github.com/bailicangdu/react-pxq)。

谢谢您的品读，此处抛砖引玉，希望大家共同探讨学习。