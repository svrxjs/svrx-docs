<style>
.markdown-body h4{
  font-size: 1.2em;
  padding-top: 1em;
}
.markdown-body h3{
  font-size: 1.45em;
}
</style>

# 插件开发指南

> 使用[`svrx-create-plugin`](https://github.com/svrxjs/svrx-create-plugin)帮助你更容易的开发插件

## 第一个 svrx 插件

让我们实现第一个插件 —— [`svrx-plugin-hello-world`](https://github.com/svrxjs/svrx-plugin-hello-world)，
它用来在终端和浏览器器端分别打印用户欢迎语

### **文件结构**


```js
└── svrx-plugin-hello-world
  ├── client.js
  ├── index.js
  └── package.json
```

> *其实只有`package.json` 和 `index.js`*是必须的

以下分别做下介绍

- `package.json`

```js
{
  "name" : "svrx-plugin-hello-world",
  "engines": {
    "svrx" : "0.0.x"
  }
  ...略..
}
```

在`package.json`中只有 2 个字段是 svrx 依赖的

- `name`: 包名要求必须是 `svrx-plugin` 开头，这是为了方便使用 npm 服务来进行插件查找
- `engines.svrx`: 定义此插件的可运行 svrx 版本，svrx 会自动加载最新的匹配插件，
  **engines.svrx** 可以是一个 [semver](https://www.npmjs.com/package/semver)，如`0.1.x`或`~0.1`等

#### - `index.js`

```js
module.exports = {
  configSchema: {
    user: {
      type: 'string',
      default: 'svrx',
      description: 'username for hello world'
    }
  },

  assets: {
    script: ['./client.js']
  },

  hooks: {
    async onCreate({ logger, config }) {
      logger.log(`Hello ${config.get('user')} from server`);
    }
  }
};
```

其中

1. `configSchema` 插件参数定义，请参考[参数定义](#schema)了解更多,这里我们定义了一个默认为`'svrx'`的字符串类型的参数 `user`
2. `assets`: 前端资源配置，他们会被自动注入到前端脚本中

   - `style`: css 脚本注入，默认注入 html 头部，本例没有使用
   - `script`: script 脚本注入，默认注入 html 尾部

   > 注入前端资源都会被合并成一个资源

3. `hook.onCreate`: 插件创建的钩子，在这里我们可以通过注入的服务组件来扩展插件，这里仅介绍使用到的服务

   - `logger`: 日志服务
   - `config`: 获取用户参数的组件

     本例做的是在终端打印关于 user 参数的欢迎语

`hook.onCreate` 实际上还接受多种组件服务，具体请[参考插件服务组件](#server)

#### - `client.js`

前端注入脚本中有一个**「全局变量 svrx」**， 挂载了一些 svrx 在前端的内置服务。

> 这个全局 svrx 仅在插件脚本内部可以访问，不用担心全局污染

```js
const { config } = svrx;

config.get('user').then(user => {
  console.log(`Hello ${user} from browser`);
});
```

这里我们仅仅是做了一个用户相关控制台 log 输出

与服务端的 config 组件不一样，这里的 config 是通过 websocket 传递的，所以接口返回的是 promise

> svrx 其实还暴露了其它前端服务，具体请参考[前端 API](#client)

### '发布' && 运行

在根目录，我们直接运行`npm publish .`尝试发布，会发现发布失败了，因为`hello-world`插件已经被[官方发布了](https://github.com/svrxjs/svrx-plugin-hello-world)。

所以我们跳过这一步，直接运行

```js
svrx -p hello-world
```

检查下浏览器端和命令行终端将可以看到 'Hello svrx from browser' 的类似日志，说明插件已经生效

也可以通过脚本的方式来启动

```js
const svrx = require('svrx');

svrx({
  plguins: ['hello-world']
}).start();
```

> 进一步阅读: [如何本地测试?](#test)

## 插件服务 API {#server}

所有插件服务都可以在两个地方被注入

- 插件定义中的`hooks.onCreate`钩子

```js
module.exports = {
  hooks: {
    async onCreate({
      middleware,
      injector,
      events,
      router,
      config,
      logger,
      io
    }) {
      // use component here
    }
  }
};
```

- svrx 实例的`plugin`事件回调，此事件会在所有插件组装完毕后执行

```js
svrx().on('plugin', async ({ logger, io }) => {
  // use component here
});
```

接下来，我们依次说明这 7 个组件

### middleware

middleware 负责添加满足 [koa 风格的中间件](https://koajs.com/) 完成后端逻辑的注入

#### - `middleware.add(name, definition)`

增加一个中间件

_Usage_

```js
middleware.add('hello-world-middleware', {
  priority: 100,
  async onRoute(ctx, next) {}
});
```

_Param_

- `name` \[String]: 定义一个唯一的中间件名

  > debug 模式下根据此命名可以 track 请求响应的管线

- `definition.priority` \[Number]: 默认为 10. svrx 会根据 priority 由高到低的组装中间件，即请求会先经过高优先级的插件
- `definition.onRoute` \[Function]: 满足 koa 规范的中间件定义，如果 `definition` 是一个函数，则会自动成为 `definenition.onRoute`

#### - `middleware.del(name)`

删除一个中间件

_Usage_

```js
middleware.del('hello-world-middleware');
```

**Param**

- `name`: 传入`add`的名称


### injector

`injector`用来改写响应流或注入前端资源

#### - `injector.add(type, definition)`

增加一种注入资源, 目前支持原生的 js 和 css 注入，他们注入规则如下

- 样式会合并到`/svrx/svrx-client.css`下，并注入到 html 的`head`闭合标签前
- 脚本会合并到`/svrx/svrx-client.js`，并注入到 html 的`body`闭合标签前

**Usage**

```js
injector.add('script', {
  content: `
    console.log('hello world')
  `
});
```

上例会往`/svrx/svrx-client.js` 注入 content 字段中的脚本内容

**Param**

- `type`: 目前仅支持`script`以及`style`
- `definition.content` \[String]: 资源的内容
- `definition.filename` \[String]: 资源文件，必须是绝对地址

> content 的优先级高于 filename，二者取其一即可

#### - `injector.replace(pattern, replacement)`

自定义 html 的替换规则

**Usage**

```js
injector.replace(/svrx/g, 'server-x');
```

上例将 html 中的 svrx 替换为 server-x

**Param**

- `pattern` \[String|RegExp]
- `replacement` \[String|Function]


> `injector.replace`的使用与`String.prototype.replace`完全一致
> 资源注入就是通过`injector.replace`实现的

### events

内置事件监听器，支持 async sorted emitter，即可以依次调用监听函数，并可在任意一个事件监听中终止事件传递

#### - `events.on(type, fn)`

**Usage**

```js
events.on('hello', async payload => {});

events.on('hello', async (payload, ctrl) => {
  ctrl.stop(); // stop events emit, only works in sorted emit mode
});
```

**Param**

- `type` \[String]: 事件名
- `fn(payload, ctrl)`: 事件回调，它包含 2 个入参
  - `payload` \[String]: 事件参数, 通过 emit 传入
  - `ctrl` \[Object]: 控制器, 调用`ctrl.stop()`可终止 sorted emit

如果回调返回一个`Promise`(比如 async function)，视为一个异步回调

#### - `events.emit(type, param, sorted)`

**Usage**

```js
// sorted emit, handler will be called one by one
events.emit('hello', { param1: 'world' }, true).then(() => {
  console.log('emit is done');
});
// parallel emit
events.emit('hello', { param1: 'world' }).then(() => {
  console.log('emit is done');
});
```

**Param**

- `type` \[String]: 事件名
- `payload`: 事件参数，被`on`中注册的回调函数接收
- `sorted` \[Boolean]: 默认`false`, 是否串行阻塞的触发事件

**Return**

`Promise`

#### - `events.off(name, handler)`

**Usage**

```js
events.on('hello'); // remove all hello's handler
events.on('hello', handler); // remove specific handler
```

#### 内置事件

- `plugin`: 当插件准备完毕后触发
- `file:change`: 当文件发生变更时被触发
- `ready`: 当服务启动时触发，如果你需要在服务启动处理部分逻辑(如获取服务端口)，请注册这个事件

### config

插件配置管理

#### - `config.get(path)`

获取一个插件配置

**config 内部维护的数据结构为不变数据**，对于配置有实时性要求的，必须使用 `config.get` 来取值，确保获得最新配置项目

**Usage**

```js
config.get('user'); // get the user param  of plugin
config.get('user.name'); // get the user.name param  of plugin
```

**你也可以获取全局参数，只需要加`$.`前缀**

```js
config.get('$.port'); // get the server's port
config.get('$.root'); // get the svrx working directory
```

**Param**

- `field`: 配置名，深层配置用`.`分割，比如`user.name`


**Return**

配置值

#### - `config.set(field, value)`


设置配置项

**Usage**

```js
config.set('a.b.c', 'hello'); // deep set
config.get('a'); // => { b: { c: 'hello' } }
```

**Param**

- `field`: 配置名，深层配置用`.`分割，比如`user.name`
- `value` 配置值

#### - `config.watch([field, ]handler)`


监听配置变化, 配置变化检查会在`set`、`del`、`splice`方法后被触发

```js
config.watch('a.b.c', (evt)=>{
  console.log(evt.affect('a.b.c')) => true
  console.log(evt.affect('a')) // => true
  console.log(evt.affect('a.c')) // => false
})
config.set('a.b.c', 'hello');
```

**Param**

- `field`: 字段名, `field`是可选的，如不传入则所有 config 变更都会触发
- handler(evt): 监听回调
  - evt.affect \[Function]: 判断本次变更是否影响某字段

#### - `config.del(field)`


删除某个配置项

**Usage**

```js
config.del('a.b.c');
config.get('a.b.c'); //=> undefined
```

**Param**

配置名

#### - `config.splice(field, start[, delCount[, items...])`

数组 splice 的 config 版本

除了`field`外，[其他参数与 Array.prototype.splice 一致](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/splice)


### router

从插件层面扩展或注册[Routing 模块](../guide/route.md#plugin)

#### - `router.route(register)`

快捷注册路由，与[Routing DSL](../guide/route.md)一致

**Usage**

```js
const {route} = router;

route({all, get, post}=>{

  all('/blog').to.send('Hi, Blog')
  get('/user').to.send('Hi, user')
  post('/user').to.send({code: 200, data: 'Success!'})

})
```

**Param**

- register(...methods): 注册回调
=======
- register(methods): 注册回调


#### - `router.action(name, builder)`

注册一个与 `proxy`、`json` 类似的 action


**Usage**

```js
const { action } = router;
action('hello', user => ctx => {
  ctx.body = `hello ${user}`;
});
route(({ all }) => {
  all('/blog').hello('svrx'); //=> response 'hello svrx'
});
```

**Param**

- `name` \[String]: action 名，即后续的调用方法名
- `builder(payload)`
  - payload: 即传入 action 方法的参数，如上栗的`'svrx'`
  - Return: builder 返回标准的[koa 中间件](https://koajs.com)

#### - `router.load(filename)`

手动加载一个 route 文件，与[启动参数 route 一致](../guide/route#start)

_加载的文件同样支持 hot reloading_

**Usage**

```js
await router.load('/path/to/route.md');
```

**Param**

- `filename`: 路由文件文件的绝对路径

**Return**

`Promise`

### logger

日志模块，它的分级可以通过`logger.level`来控制(默认为`warn`)

```js
svrx({
  logger: {
    level: 'error'
  }
});
```

或从 cli 端

```bash
svrx --logger.level error
```

上例将会输出`warn`以上的日志，如`notify`、`error`

### - `logger[level](msg)`

svrx 提供了多种级别的日志，分别是`slient`, `notify`, `error` , `warn`(默认日志分级), `info`, `debug`

**Usage**

```js
logger.notify('notify'); // show `nofity`
logger.error('error'); // show `error` and `notify`
logger.warn('warn'); // show `warn`、`error` and `norify`

```

> `logger.notify` 由于会非常常用，所以它有一个 alias `logger.log`

### io

io 负责插件后端与前端的通信， 请结合 [client 端的 io](#client-io) 查看

#### - `io.on( type, handler )`

监听浏览器端发送消息(即通过[浏览器端`io.emit`发送](#client-io))

```js
io.on('hello', payload => {
  console.log(payload); // =>1
});
```

**Param**

- `type`: 事件名
- `handler(payload)`: 事件回调
  - `payload`: 事件参数

#### - `io.emit(type, payload)` {#client-io}

发送 io 事件到服务端

**Usage**

client side

```js
const { io } = svrx;
io.emit('hello', 1);
```

server side

```js
hooks: {
  async onCreate({io}){
    io.on('hello', payload=>{
      console.log(payload) // =>1
    })
  }
}
```

**Param**

- `type`: 事件名
- `payload`: 事件参数

> 注意事件参数必须是可序列化的，因为要通过网络传输

#### - `io.off(type[, handler])`

解除监听

**Usage**

```js
io.off('hello'); //=> remove all hello handlers
io.off('hello', handler); //=> remove specific handler
```

#### - `io.register(name, handler)`

注册一个 io 服务， 它可以在客户端或服务端以`io.call`的方式调用

**Usage**

```js
io.register('hello.world', async payload => {
  return `Hello ${payload}`;
});
```

**Param**

- `name` \[String]: 服务名，`call` 中会被使用
- `handler`: 服务逻辑实现，异步请返回 Promise 或使用 async/await

#### - `io.call(name, payload)` {#call}

调用注册的服务

**Usage**

上例如下调用会返回 'Hello svrx'

```js
io.call('hello.world', 'svrx').then(data => {
  console.log(data); // => Hello svrx
});
```


**Param**

- `name` \[String]: 服务名
- `payload` \[Any]: 服务入参

**Return**

Promise

## 客户端 API {#client}

客户端 API 统一通过 svrx 全局变量暴露，如下例

```js
const { io, events, config } = svrx;
```

以下依次说明

### io

通信模块，负责与服务端通信

#### - `io.on(type, handler)`

监听服务端 io 事件

**Usage**

server side

```js
io.emit('hello', 1);
```

client side

```js
const { io } = svrx;
io.on('hello', payload => {
  console.log(payload); //=>1
});
```

> 注意后端 `io.emit` 属于广播，所有前端页面都会收到信息

**Param**

- `type`: 事件名
- `handler(payload)`: 事件回调
  - `payload`: 事件参数

#### - `io.emit(type, payload)` {#client-io}

发送 io 事件到服务端

**Usage**

client side

```js
const { io } = svrx;
io.emit('hello', 1);
```

server side

```js
{
  hooks: {
    async onCreate({io}){
      io.on('hello', payload=>{
        console.log(payload) // =>1
      })
    }
  }
}
```

**Param**

- `type`: 事件名
- `payload`: 事件参数

> 注意事件参数必须是可序列化的，因为要通过网络传输


#### - `io.off(type[, handler])`

解除监听

**Usage**

```js
io.off('hello'); //=> remove all hello handlers
io.off('hello', handler); //=> remove specific handler
```

#### - `io.call(name, payload)`

在 client 端的 [io.call](#call) 与服务端完全一致，**但要确保 payload 是可序列化的**,因为会经过网络传输

**Usage**

```js
io.call('hello.world', 'svrx').then(data => {
  console.log(data);
});
```

**Param**

- `name` \[String]: 服务名
- `payload` \[Any]: 服务入参

**Return**

Promise

### events

浏览器端内部的事件发射器，这部分和[服务端 events](#events)完全一致，不做赘述

**Usage**

```js
const { events } = svrx;
events.on('type', handler);
events.emit('type', 1);
events.off('type');
```

> events 与 io 的不同在于 io 是 服务端与客户端的通信，而 events 是单端的事件触发器

### config

客户端的`config`模块与服务端几乎一致，唯一区别是从同步接口编程了 Promise 化的异步接口(因为 socket 的网络通信)

#### - `config.get(field)`


**Usage**

```js
config.get('$.port').then(port => {
  // get the server port
});

config.get('user').then(port => {
  //get the param belongs to current plugin
});
```

> 注意获取的参数是属于脚本的，如果获取全局请假`$.`前缀

#### - `config.set`、`config.splice`、`config.del`

上述三个方法和`get`一样，与服务端表现一致，不过返回值变为 Promise

**Usage**

```js
config.set('a.b', 1).then(() => {
  config.get('a.b').then(ab => {
    console.log(ab); // => 1
  });
});
```

### 参数定义 {#schema}

svrx 支持一种基于 [json-schema](https://json-schema.org/) 扩展的参数定义，以内部定义的参数为例

**Usage**

```js
module.exports = {
  root: {
    type: 'string',
    default: process.cwd(),
    description: 'where to start svrx',
    ui: false
  },
  route: {
    description: 'the path of routing config file',
    anyOf: [{ type: 'string' }, { type: 'array' }]
  },
  logger: {
    description: 'global logger setting',
    type: 'object',
    properties: {
      level: {
        type: 'string',
        default: 'error',
        description:
          "set log level, predefined values: 'silent','notify','error','warn', 'debug'"
      }
    }
  }
};
```

**Field 详解**

- `type` \[String]: [JSON-Schema 允许的字段类型](https://json-schema.org/understanding-json-schema/reference/type.html) ，可以是`array`、`string`、`number`、`boolean`、`object`、`null`
- `default` \[Any]: 默认值
- `required` \[Boolean]: 是否是必须的, 默认是`false`
- `properties` \[Object]: 定义属性，每层属性又是一层 schema 定义
- `ui`: svrx 的扩展字段，是否要在 [svrx-ui](https://github.com/x-orpheus/svrx-ui) 中被展示
- `anyOf`: 属性任选其一，如
  ```of
  route: {
    description: 'the path of routing config file',
    anyOf: [{ type: 'string' }, { type: 'array' }],
  },
  ```
  > 进一步的 JSON Schema 定义请参考[官方文档](https://json-schema.org/)

## 如何测试? {#test}

不建议大家发布测试插件到 npm, 可以通过以下方式来进行本地测试

```js
svrx({
  plguins: [
    {
      name: 'hello-world',
      path: '/path/to/svrx-plugin-hello-world'
    }
  ]
}).start();
```

指定 path 参数后，svrx 会加载本地包，而不是从 npm 中获取

## 更容易的插件开发 —— [`svrx-create-plugin`](https://github.com/x-orpheus/svrx-create-plugin)

svrx 官方提供的脚手架帮助你更容易的开发和发布你的插件

