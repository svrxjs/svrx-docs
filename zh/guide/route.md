# Routing 路由的使用

## 快速起步

你可通过以下命令来快速尝试 svrx 的动态路由功能

```bash
touch route.js # create empty routing file
svrx --route route.js 
```

在 `route.js` 中

```js
get('/blog').to.json({ title: 'svrx' });
```

打开 `/blog`，你将看到 `{title: 'svrx'}` 的 json 输出

### 特性

- __支持 hot reloading__ ( 通过编辑 `route.js` 来验证 )
- 简单的书写，直观的阅读
- 支持通过[插件](#plugin)来扩展和分发

## 语法

`[method](selector).to.[action](payload)` 


举个例子

```js
post('/blog').to.proxy('http://music.163.com');
```

分别对应

- method: post
- selector: /blog
- action: proxy
- payload: 'http://music.163.com'

> _其中 to 只是一个介词，可以省略_


## 路由

### 路由方法

svrx 路由支持 [methods](https://github.com/jshttp/methods/blob/master/index.js) 定义的 http methods

> ⚠️ 由于 delete 与 js 保留字冲突，你可以使用 del() 来创建 DELETE 方法


### 路由匹配

svrx 路由基于 [`path-to-regexp`](https://github.com/pillarjs/path-to-regexp) 匹配, 
和两大平民 Node 框架 [express](https://expressjs.com/en/guide/routing.html) 和 [koa](https://github.com/ZijianHe/koa-router) 相同.

查看 [`path-to-regexp`](https://github.com/pillarjs/path-to-regexp) 来了解更详尽的匹配规则，以下做简要概述

#### 规则解析举例

- `/svrx/:id`: 具名匹配，规则默认为 `(\w+)`
- `/svrx/:id(hello|world)`: 具名且指定匹配规则 
- `/svrx(.*)`: 不具名匹配 
- `/\/svrx\/(.*)$/`: __对于复杂规则的路由，也可以直接使用正则表达式__

#### 匹配参数

- 具名匹配如`/:id`, 可以通过 `ctx.params.id` 获得
- 匿名匹配如`/(hello|world)/(.*).html`，可以通过 `ctx.params[0]` 以及 `ctx.params[1]` 依次获得匹配参数
- 正则匹配如`/\/svrx\/(.*)$/`，`ctx.params[i]` 依次获得正则的子匹配结果


#### 参数快捷映射

除 [Action:handle](#handle) 外，大部分 action 在语法上不具备操作 koa 上下文( context ) 的能力，部分 action 做了 参数快捷映射 的支持

以 [sendFile](#send-file) 为例

```js
get('/html/:path.(html|htm)').to.sendFile('./{path}.{0}')
```

- `/html/index.html` 会发送 `${root}/html/index.html`
- `/html/home.htm` 会发送 `${root}/html/home.htm`

## Action 清单

你可以使用路由的 [__插件接口__](#plugin) 来扩展 Action

### send

发送响应内容

```js
get('/blog').to.send({ title: 'this is a blog' });
```

send 是 koa 框架 `ctx.body` 的语法糖，当 payload 类型不同时有以下默认行为

- `string`
  - 如果包含以`<`开头的字符比如`<html>`, `Content-Type` 头会设置为 `text/html`, 浏览器会渲染为 html
  - 否则返回`text/plain`
- `object` or `array` or `number` or `boolean` ...
  - 返回 `json`, `Content-Type` 为 `application/json`


### sendFile {#send-file}

发送文件内容，根据自动文件后缀设置 `Content-Type`

```js

get('/index.html').to.sendFile('./index.html');

```

- 根路径 = `serve.base` || `root`
- __⚠️支持参数映射__, 如下例

```js
get('/file/:id.html').to.sendFile('./assets/{id}.html')
```


### json

无论 payload 为什么类型，都发送 `json` 响应

```js
get('/blog').to.json({title: 'svrx'});
```


### redirect(target\[, code])

服务端跳转

- target: 目标 path
- code: http code, default `302`



```js
get('/blog').to.redirect('/user');
```

> __⚠️支持参数映射__, 如下例

`get('/blog/:path(.*)').to.redirect('/user/{path}')`


### header

设置响应头，由于 header 并不发送响应内容，你可以串联其他 action

```js
get('/blog')
  .to.header({ 'X-Engine': 'svrx' })
  .json({ code: 200 });
```


### rewrite

路由重写

> __⚠️支持参数映射__

```js
get('/old/rewrite:path(.*)').to.rewrite('/svrx/{path}')
get('/svrx(.*)').to.send('Hello svrx')
```

则`/old/1`和`/svrx/1` 都会返回`Hello svrx`

> 由于 rewrite 并不发送响应内容，你也可以串联其他 action

### proxy(target\[, options]) {#proxy}

代理，将 path 代理到 target 服务器。

- target: 目标服务器
- options: 同 [proxy.options](./api.md#proxy) 
    - changeOrigin
    - secure
    - pathRewrite

```js
get('/api(.*)').to.proxy('http://mock.server.com/')
get('/test(.*)').to.proxy('http://mock.server.com/', {
  secure: false,
})
get('/test/:id').to.proxy('http://{id}.dynamic.server.com/')
```

### handle {#handle}

handle 即定义一个 koa 的中间件，属于全能力 action，以上所有功能都可以使用 handle 来实现，
代价就是可读性的降低.

```js

get('/hello-world').to.handle((ctx)=>{
  ctx.type = 'html'
  ctx.body = '<body>Hello World</body>'
});

```

> 尽可能抽取通用能力为自定义 action，请参考「插件接口」小节

## 插件接口 {#plugin}

插件的 `hooks.onCreate` 会注入名为 `router` 对象, 包含三个方法

- action: 注册一个与 proxy、json 等同级的 action
- load: 加载一个 routing file
- route: 脚本式的定义 router

详细请参考 [插件开发指南](../contribute/plugin.md#router)


## 完整范例文件

```js
get('/handle(.*)').to.handle((ctx) => { ctx.body = 'handle'; });
get('/blog(.*)').to.json({ code: 200 });
get('/code(.*)').to.send('code', 201);
get('/json(.*)').to.send({ json: true });
get('/text(.*)').to.send('haha');
get('/html(.*)').to.send('<html>haha</html>');
get('/rewrite:path(.*)').to.rewrite('/query{path}');
get('/redirect:path(.*)').to.redirect('localhost:9002/proxy{path}');
get('/api(.*)').to.proxy('http://mock.server.com/')
get('/test(.*)').to.proxy('http://mock.server.com/', {
  secure: false,
})
get('/test/:id').to.proxy('http://{id}.dynamic.server.com/')
get('/query(.*)').to.handle((ctx) => {
  ctx.body = ctx.query;
});
get('/header(.*)')
  .to.header({ 'X-From': 'svrx' })
  .json({ user: 'svrx' });
get('/user').to.json({ user: 'svrx' });

get('/sendFile/:path(.*)').to.sendFile('./{path}');
get('/sendFile/:path(.*)').to.sendFile(
  libPath.join(__dirname, '../fixture/plugin/serve/{path}'),
);
```



