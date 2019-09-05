# [How To Use Routes

## Getting Started

You can try the following commands to start `svrx` routing quickly:

```bash
touch route.js # create empty routing file
svrx --route route.js 
```

In your `route.js`:

```js
get('/blog').to.json({ title: 'svrx' });
```

Then open `/blog`, you'll see the json output `{title: 'svrx'}`.

### Features

- __support hot reloading__ ( check it out by editing your `route.js` now)
- easy writing, clear reading
- support expanding through [plugin](#plugin)

## Syntax

`[method](selector).to.[action](payload)` 

For example:

```js
post('/blog').to.proxy('http://music.163.com');
```

This rule can be translate into:

- method: post
- selector: /blog
- action: proxy
- payload: 'http://music.163.com'

> _'to' is just a preposition word here, which can be omitted_


## Route

### Route Methods

`svrx` route supports all the http methods defined by [methods](https://github.com/jshttp/methods/blob/master/index.js). 

> ⚠️ 'delete' is a reserved word of javascript, so you might use del() to create a DELETE method.


### Route Matching

The match rule of `svrx` route is based on [`path-to-regexp`](https://github.com/pillarjs/path-to-regexp), 
which is also used by [express](https://expressjs.com/en/guide/routing.html) and [koa](https://github.com/ZijianHe/koa-router).

The following is just some briefs of matching rules, 
please check [`path-to-regexp` doc](https://github.com/pillarjs/path-to-regexp) for more detail. 


#### Common Rules

- `/svrx/:id`: named parameters matching, default parameter rule is `(\w+)`
- `/svrx/:id(hello|world)`: named parameters matching with custom parameter matching rule 
- `/svrx(.*)`: unnamed parameters matching 
- `/\/svrx\/(.*)$/`: __use regexp directly for complicated routes__

#### Parameters

- named parameters like `/:id`, can be accessed through `ctx.params.id`
- unnamed parameters like `/(hello|world)/(.*).html`, can be accessed through `ctx.params[0]` and `ctx.params[1]`
- regexp like `/\/svrx\/(.*)$/`, can be accessed through `ctx.params[i]` in order

#### Parameters Mapping

In fact, except [Action:handle](#handle), 
most actions do not have the ability to access the koa context, 
so we need `parameters mapping` for some actions.

Take [sendFile](#send-file) as an example:

```js
get('/html/:path.(html|htm)').to.sendFile('./{path}.{0}')
```

- `/html/index.html` will send `${root}/html/index.html`
- `/html/home.htm` will send `${root}/html/home.htm`

## Action List

You can use [__Route API for Plugins__](#plugin) to write your own action.

### send

Send response content.

```js
get('/blog').to.send({ title: 'this is a blog' });
```

`send` is a syntactic sugar for `ctx.body` of koa. 
And there're some default behaviors for different payload types.

- `string`
  - if started with `<`, like `<html>`, the `Content-Type` header will be set as `text/html`
  - if not, return `text/plain`
- `object` or `array` or `number` or `boolean` ...
  - return `json`, the `Content-Type` will be `application/json`


### sendFile {#send-file}

Send file content,
and it will auto set the `Content-Type` header according to the file extension.

```js
get('/index.html').to.sendFile('./index.html');
```

- root path = `serve.base` || `root`
- __⚠️support parameters mapping__, for example:
    ```js
    get('/file/:id.html').to.sendFile('./assets/{id}.html')
    ```

### json

Send `json` response, despite the type of payload.

```js
get('/blog').to.json({title: 'svrx'});
```

### redirect(target\[, code])

Server side redirecting.

- target: target path
- code: http code, default is `302`

```js
get('/blog').to.redirect('/user');
```

> __⚠️support parameters mapping__, for example:

`get('/blog/:path(.*)').to.redirect('/user/{path}')`


### header

Set response headers. 
`header` doesn't send any response content, 
so you can chain this action to other actions.

```js
get('/blog')
  .to.header({ 'X-Engine': 'svrx' })
  .json({ code: 200 });
```

### rewrite

Rewrite routes.

> __⚠️support parameters mapping__

```js
get('/old/rewrite:path(.*)').to.rewrite('/svrx/{path}')
get('/svrx(.*)').to.send('Hello svrx')
```

Both `/old/1` and `/svrx/1` will return `Hello svrx`.

> `rewrite` doesn't send any response content, 
you can chain this action to other actions.

### proxy(target\[, options]) {#proxy}

Proxy path to target server.

- target: target server
- options: same as [proxy.options](./api.md#proxy) 
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

`handle` is a powerful action, it defines a middleware of koa, 
which means all actions above can be implemented by`handle`,
but the cost is the reduction of code readability.

```js
get('/hello-world').to.handle((ctx)=>{
  ctx.type = 'html'
  ctx.body = '<body>Hello World</body>'
});
```

> Instead of using handle, 
it is recommended to use 'smaller' actions,
you can customize your own actions using route api for plugins,
see next section.

## Route API for Plugins {#plugin}

You can create a new action in your own plugin.
There's a `router` object in your `hooks.onCreate`,
which has 3 methods inside:

- action: register an action just like `proxy`, `json`, ...
- load: load a routing file
- route: define a router in scripts

Please read [How To Write A Plugin](../contribute/plugin.md#router) for more information.

## Examples

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
```



