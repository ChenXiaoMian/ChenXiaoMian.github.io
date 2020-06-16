---
title: 利用 Redis 解决 NodeJS 中 Session 存储问题
date: 2020-06-15
categories: node
tags: [node,redis,session,cookie]
---

## 业务场景

在用户访问网站时，我们经常需要记录一些信息，比如

* 会话状态管理（如用户登录状态、购物车信息）
* 个性化设置（如用户自定义设置、主题等）
* 浏览器行为跟踪（如跟踪分析用户行为，淘宝的商品推荐）
* 用户身份（如权限较高的页面，普通用户无法访问）

这时候我们可以借助 Cookie，以下来自 MDN 的官方解释

> HTTP Cookie（也叫Web Cookie或浏览器Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。通常，它用于告知服务端两个请求是否来自同一浏览器，如保持用户的登录状态。Cookie使基于无状态的HTTP协议记录稳定的状态信息成为了可能。

## Cookie & Session

众所周知，`HTTP`是无状态的协议。这个的意思就是说，如果发送两个完全一样的请求，那么收到的响应也会完全相同。然而在实际生活中，这明显不符合许多场景。因为每个人虽然都点击了按钮，但我们应该收到不同的内容。服务器需要对我们做出区分，这时候`cookie`就登场了。我发出请求，服务器在响应里面加一个`Set-Cookie`，到我们浏览器里设了一个`cookie`（点开devtool->Application->Cookies查看），下一次发送请求的时候，我的header里面就带有`cookie`了，服务器看到`cookie`，就知道我是谁了。这样就完成了一次认证。

但是接下来还有一个问题：服务器资源极其宝贵，如果每次都认证会造成资源浪费。加之，如果我希望能够暂时性地在当前会话存储一些信息，存储在`cookie`会显得非常浪费，而且放在客户端不够安全。因此`session`就来了。`session` 中文意思名为“会话”，是一种解决方案，代表客户端和服务端的一次通信过程，在这个过程中如果客户端需要记录数据，服务端会暂时把数据挂载到 `session` 对象上，当请求结束响应时，将 `session` 中挂载的数据持久化到客户端的 `cookie`上，清空 `session`，关闭会话。`cookie` 可以看做一个信息容器，借助浏览器的环境对服务端的数据进行持久化存储，随后每次都会在 HTTP 请求头中携带并发送至服务端，这样服务端就可以辨识请求的来源。

## Set-Cookie

Set-Cookie是request的header。header的格式是NAME=VALUE然后用分号‘;’分隔开来。
其中有几个设置比较常用：

* expires=Date (设置cookie的到期时间)
* secure (仅仅只在https下使用)
* HttpOnly (使得cookie不能被客户端JavaScript修改)
* maxAge （cookie的保持时间，以毫秒为单位）

## Session 的存储方案

### 存储在 Cookie 中

1. 安装

```
npm install koa-session koa --save
```

2. app.js

```js
const Koa = require('koa');
const app = new Koa();
const session = require('koa-session');

app.keys = ['mykey'];
const config = {
	key: 'myAppSessKey',
	maxAge: 86400000,
	overwrite: true,
	httpOnly: true,
	signed: true,
};

app.use(session(config, app));
app.use(ctx => {
	if (ctx.path === '/favicon.ico') return;
	let n = ctx.session.views || 0;
	ctx.session.views = ++n;
	ctx.body = n + ' views';
});

app.listen(3000,() => {
  console.log('server is running at http://localhost:3000');
});
```
上述代码使用内存来保持 `Session` 信息，记录用户访问次数(`views`)。启动服务后，在浏览器中访问 `http://localhost:3000/`，并不断刷新页面，就能看到页面上的`views`数值一直在增加。如果换个浏览器继续访问`http://localhost:3000/`，则 `views` 将会重新从`1`开始计数。

假如此时重启服务器，两个浏览器的计数都会重新从`1`开始 在实际应用环境中服务器重启的情况时有发生，所以将 `Session` 单纯地放在内存中容易丢失，因此 需要加入`Redis` 做持久化。而且，在实际应用环境中，线上的服务器肯定不止台，多台服务器之间 `Session` 信息同步也要依靠 `Redis` 务来实现。

另外，在 `Cookie` 中存储 `Session` 是我们经常使用的方法，虽然简便，却存在两个致命的缺点：

1.`session` 同步问题，假设我们现有两个站点，分别为`a.hello.com` `b.hello.com`，如果要实现多站点共享 `Cookie`，基本就是把 `Cookie `的 `Domain` 属性设置成 `.hello.com`，这样的话，我们在`a.hello.com`完成了登录，进入`b.hello.com`会进行自动登录。此时，我们在`a.hello.com`修改了 `session`（把用户权限从0设置为1），同时会修改浏览器的 `Cookie`。再进入`b.hello.com`，`b.hello.com`服务器的 `session` 是老的（用户权限为0），由于新的 `session` 会根据新 `Cookie` 解析出用户权限为1，这和原本的 `session` 内容冲突，出于安全考虑，服务器会放弃新的 `session`，继续采用老的 `session`，造成了 `session` 同步失败，引发问题。

2.`Cookie` 的大小是有限制的，一般为 `4KB`，任何 `Cookie` 大小超过限制都被忽略，且永远不会被设置，万一我们的状态信息超过4KB，就会丢失。

### 存储在 Redis 中

而 `Redis` 是一个基于内存的键值数据库，它由 C 语言实现的，与 Nginx / NodeJS 工作原理近似，同样以单线程异步的方式工作，先读写内存再异步同步到磁盘，读写速度上比 MongoDB 有巨大的提升。因此目前很多超高并发的网站/应用都使用 Redis 做缓存层，普遍认为其性能明显好于 MemoryCache。当并发达到一定程度时，即可考虑使用 Redis 来缓存数据和持久化 Session。

无论是采用 MongoDB 还是 Redis 都能解决 Cookie 存储的缺点，因为采用数据库存储后，Cookie 只负责存储 Key，所有的状态信息保存在同个数据库中，根据 Key 去查找数据库中的数据，再进行相应的读取、修改、删除。

显而易见，对于 Session 这种读写场景频繁，CRUD 操作频繁的持久化内容，使用 Redis 进行存储简直是小而美。

**安装 Redis & RedisDesktopManager**

自行百度...

**创建 Node 服务**

参照 [koa-session2](https://github.com/Secbone/koa-session2#readme) 官网。

1. 安装 

```
npm install koa-session2 koa ioredis --save
```

2. redis-store.js

```js
const Redis = require('ioredis');
const { Store } = require('koa-session2');

class RedisStore extends Store {
	constructor(config) {
		super();
		this.redis = new Redis(config);
	}
	async get(sid, ctx) {
		let data = await this.redis.get(`SESSION:${sid}`);
		return JSON.parse(data);
	}
	async set(session, { sid = this.getID(24), maxAge = 1000000 } = {}, ctx) {
		try {
			await this.redis.set(`SESSION:${sid}`, JSON.stringify(session), 'EX', maxAge/1000);
		} catch(e) {
		}
		return sid;
	}
	async destroy(sid, ctx) {
		return await this.redis.del(`SESSION:${sid}`);
	}
}

module.exports = RedisStore;
```

3. app.js

```js
const Koa = require('koa');
const app = new Koa();
const session = require('koa-session2');
const Store = require('./utils/redis-store');
const redisConfig = {
	port: 6379,
	host: '127.0.0.1',
	family: 4,
	password: 123456,
	db: 0
};

app.keys = ['mykey'];

let store = new Store(redisConfig);
const config = {
	key: 'myAppSessKey',
	maxAge: 86400000,
	overwrite: true,
	httpOnly: true,
	signed: true,
	store
};

app.use(session(config, app));
app.use(ctx => {
	if (ctx.path === '/favicon.ico') return;
	let n = ctx.session.views || 0;
	ctx.session.views = ++n;
	ctx.body = n + ' views';
});

app.use(ctx => {
	// refresh session if set maxAge
	ctx.session.refresh();
});

app.listen(3000,() => {
  console.log('server is running at http://localhost:3000');
});
```

最后，我们成功利用 `Redis` 解决 NodeJS 中 `Session` 存储问题，同时做到了安全，健壮，快速的三个要性。

其实 Redis 还可以实现很多功能，作为一个缓存中间层，在某些场景下，可以大大优化我们的应用的性能，比如在数据更新不频繁，但用户读取频繁的场景下，可以将数据保存在 Redis 中，通过 Node 返回。
