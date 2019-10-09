# How To Write A Plugin

>**Use [`svrx-create-plugin`](https://github.com/svrxjs/svrx-create-plugin)** to help you create plugins more easily

### First Plugin

Let's create our first plugin —— [`svrx-plugin-hello-world`](https://github.com/svrxjs/svrx-plugin-hello-world)

### File Structure

```js
└── svrx-plugin-hello-world
  ├── client.js
  ├── index.js
  └── package.json
```

> _Only `package.json` and `index.js`_ are required

- `package.json`

```js
{
  "name" : "svrx-plugin-hello-world",
  "engines": {
    "svrx" : "0.0.x"
  }
}
```

Only two fields are required by `package.json`

- `name`: package name must be a string beginning with `svrx-plugin`, to help `svrx` find it in npm.
- `engines.svrx`: Define the runnable svrx version of this plugin，svrx will automatically load the latest matching plugin
  **engines.svrx** can be a valid [semver](https://www.npmjs.com/package/semver)，like `0.1.x` or `~0.1`

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

Where

1. `configSchema` Plugin param definition based on JSON Schema ，checkout [JSON Schema](#schema) for more detail.
   Here we just set a `user` field, it is a string
2. `assets`: client resource setting，they will be automatically injected into the page

   - `style`: css resource injection
   - `script`: script resource injection

     > All resources will be merged into a single file.

3. `hook.onCreate`: a function invoke when plugin been creating, we can control it by injected service
   in this example, we only use two services

   - `logger`: logger service
   - `config`: config service

Check out setion [service](#server) to find out other services that available in `hook.onCreate`

#### - `client.js`

There is a global variable named `svrx` will be injected into all pages,
it has some built-in services

> `svrx` is only accessible inside the plugin script, don't worry about global pollution

```js
const { config } = svrx;

config.get('user').then(user => {
  console.log(`Hello ${user} from browser`);
});
```

You will see `Hello svrx from browser` in console panel

Unlike in server side, config in client is passed through websocket, so api is async, and return a promise

> svrx also provide other client services, please check out [client api](#client) for help

### publish && running

There'll be a failure when trying to publish it by `npm publish`. 
Beacuse `svrx-plugin-hello-world` has been published by [offical team](https://github.com/svrxjs/svrx-plugin-hello-world)

So we skip this step and try the plugin directly

```js
svrx -p hello-world
```

Check the terminal and browser console log, 
we will find `Hello svrx from browser`, 
which means plugin has worked successfully.

You can also run it in programmatical way

```js
const svrx = require('@svrx/svrx');

svrx({
  plugins: ['hello-world']
}).start();
```

> Further reading: [How to test plugin?](#test)

## Server Side Service {#server}

You can use serivce in two places

- `hooks.onCreate`

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
      // use service here
    }
  }
};
```

- `plugin` event

```js
svrx(config).on('plugin', async ({ logger, io }) => {
  // use service here
});
```

We will explain these 7 services in turn

### middleware

Middleware is used for adding [koa-style middleware](https://koajs.com/) to implement backend logic injection

#### - `middleware.add(name, definition)`

Adding [koa-style middleware](https://koajs.com/)

_Usage_

```js
middleware.add('hello-world-middleware', {
  priority: 100,
  async onRoute(ctx, next) {}
});
```

_Param_

- `name` \[String]: A unique middleware name

  > in debug mode, it can be used to track the middleware call process

* `definition.priority` \[Number]: default is `10`.
  Svrx will assemble the middleware according to the priority from high to low, that is, the request will be passed to the high priority plugin first.
* `definition.onRoute` \[Function]: A [koa-style](https://koajs.com) middleware .If `definition` is a function, it will automatically become `definenition.onRoute`

#### - `middleware.del(name)`

Delele a middleware with the specified name

_Usage_

```js
middleware.del('hello-world-middleware');
```

**Param**

- `name`: middleware name

### injector

`injector` is used for rewriting the response and inject client resources.

#### - `injector.add(type, resource)`

Add a resource for injection , only js and css has been supported.

The rule as follow.

- The style will be merged into `/svrx/svrx-client.css` and
  injected before the closing `head` tag
- The script will be merged into `/svrx/svrx-client.js` and
  injected before the closing `body` tag

**Usage**

```js
injector.add('script', {
  content: `
    console.log('hello world')
  `
});
```

The `content` will be merged into `bundle script`

**Param**

- `type`: Only support `script` and `style`
- `resource.content` \[String]: resource content
- `resource.filename` \[String]: the path of resource file, must be a absolute path

> Content has a higher priority than filename, so you can take one of them.

#### - `injector.replace(pattern, replacement)`

`injector.repalce` will transform response body with some or all matches of a pattern replaced by a replacement.

**Usage**

```js
injector.replace(/svrx/g, 'server-x');
```

The above example replace all `svrx` with `server-x`

**Param**

- `pattern` \[String|RegExp]
- `replacement` \[String|Function]

> The usage of `injector.replace`is exactly the same as [`String.prototype.replace`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace)
>
> resource injection is based upon `injector.replace`

### events

Built-in event listener, support async&sorted emitter, , which can call the listen function in turn, and can terminate the event delivery at any time.

#### - `events.on(type, fn)`

**Usage**

```js
events.on('hello', async payload => {});

events.on('hello', async (payload, ctrl) => {
  ctrl.stop(); // stop events emit, only works in sorted emit mode
});
```

**Param**

- `type` \[String]: event name
- `fn(payload, ctrl)`: callback that has two params
  - `payload` \[String]: event data that pass through `emit`
  - `ctrl` \[Object]: control object, call `ctrl.stop()` to stop 'sorted emit'

> If the callback returns a `Promise` (such as async function), it will be treated as an asynchronous watcher.

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

- `type` \[String]: event name
- `payload`: event data
- `sorted` \[Boolean]: default is `false`, whether to pass events serially

**Return**

`Promise`

#### - `events.off(name, handler)`

Remove event watcher

**Usage**

```js
events.off('hello'); // remove all hello's handler
events.off('hello', handler); // remove specific handler
```

#### builtin events

- `plugin`: triggered after plugin building.
- `file:change`: triggered when any file changes
- `ready`: triggered when server starts, if you need to handle logic after server startup (such as getting the server port), you can register this event

### config

Config service used to modify or query the options passed by user;

#### - `config.get(path)`

Get the config of this plugin.

**Config is based on immutable data** , you must always use `config.get` to ensure getting the latest config.

**Usage**

```js
config.get('user'); // get the user param  belong to this plugin
config.get('user.name'); // get the user.name param  belong to this plugin
```

**if you need to get global config，just add prefix `$.`**

```js
config.get('$.port'); // get the server's port
config.get('$.root'); // get the svrx working directory
```

**Param**

- `field`: field path，deep getter must be separated by `.`, such as `user.name`

**Return**

The value of the field

#### - `config.set(field, value)`

Modify the config

**Usage**

```js
config.set('a.b.c', 'hello'); // deep set
config.get('a'); // => { b: { c: 'hello' } }
```

**Param**

- `field`: field path，deep setter must be separated by `.`, such as `user.name`
- `value`: field value

#### - `config.watch(field, handler)`

Listening for configuration changes,
change digest will be triggered after the `set`, `del`, `splice` method calls

```js
config.watch((evt)=>{
  console.log(evt.affect('a.b.c')) => true
  console.log(evt.affect('a')) // => true
  console.log(evt.affect('a.c')) // => false
})

config.watch('a.b', (evt)=>{
    console.log('a.b has been changed')
})
config.set('a.b.c', 'hello');
```

**Param**

- `field`: field path，deep watcher must be separated by `.`, such as `user.name`
- `handler(evt)`: watch handler
  - `evt.affect(field)` \[Function]:
    detect whether specific field has been changed

#### - `config.del(field)`

Remove some field

**Usage**

```js
config.del('a.b.c');
config.get('a.b.c'); //=> undefined
```

#### - `config.splice(field, start[, delCount[, items...])`

The `Array.prototype.slice`

Except for `field`，params are identical to [Array.prototype.splice](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice)

**Example**

```js
config.set('a.b.c', [1, 2, 3]);
config.splice('a.b.c', 1, 1);
config.get('a.b.c'); // => [1,3]
```

### router

Extending [Routing DSL](../guide/route.md#plugin)

#### - `router.route(register)`

Register route，as same as [Routing DSL](../guide/route.md)

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

- register({...methods}):
  - methods: corresponding to [HTTP methods](../guide/route.md#method)


#### - `router.action(name, builder)`

Register an action , like [`proxy`](../guide/route.md#proxy) or [`json`](../guide/route.md#json)

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

- `name` \[String]: action name
- `builder(payload)`
  - payload: payload that passed to action，like `'svrx'` in above example
  - Return: builder must return a [koa style](https://koajs.com) middleware

#### - `router.load(filename)`

Load routing file manually ，Same as [options `--route`](../guide/route#start)

_Also support hot reloading_

**Usage**

```js
await router.load('/path/to/route.md');
```

**Param**

- `filename`: absolute path for routing file

**Return**

`Promise`

### logger

Logger module, who's level can be controlled by `logger.level` (default is `warn`)

```js
svrx({
  logger: {
    level: 'error'
  }
});
```

Or cli way

```bash
svrx --logger.level error
```

Above example will output log that more than `warn`, such as `notify`, `error`

### `logger[level](msg)`

Svrx provides multiple levels of logging: `slient`, `notify`, `error`, `warn` (default), `info`, `debug`

**Usage**

```js
logger.notify('notify'); // show `nofity`
logger.error('error'); // show `error` and `notify`
logger.warn('warn'); // show `warn`、`error` and `norify`
```

> `logger.log` is an alias for `logger.notify`

### io

io is used for the communication between server and client. Please check it out in [client-side io](#client-io)

#### - `io.on( type, handler )`

Listening for client messages (send by [client side `io.emit`](#client-io))

```js
io.on('hello', payload => {
  console.log(payload); // =>1
});
```

**Param**

- `type`: message type
- `handler(payload)`: handler for message
  - `payload`: message data

#### - `io.emit(type, payload)`

Send message to client

**Usage**

Client side

```js
const { io } = svrx;
io.emit('hello', 1);
```

Server side

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

- `type`: message type
- `payload`: message data

> Message payload must be serializable because they are transmitted over the network

#### - `io.off(type[, handler])`

Remove the message watcher

**Usage**

```js
io.off('hello'); //=> remove all hello handlers
io.off('hello', handler); //=> remove specific handler
```

#### - `io.register(name, handler)`

Register io service， which can be invoked by `io.call` in client and server.

**Usage**

```js
io.register('hello.world', async payload => {
  return `Hello ${payload}`;
});
```

**Param**

- `name` \[String]: service name used by `io.call`
- `handler`: a Function return Promise, implement service logic

#### - `io.call(name, payload)` {#call}

Invoke registered service

**Usage**

```js
io.call('hello.world', 'svrx').then(data => {
  console.log(data); // => Hello svrx
});
```

**Param**

- `name` \[String]: service name
- `payload` \[Any]: service payload will passed to service handler

**Return**

Promise

## Client API {#client}

The client APIs are uniformly exposed through global variable `svrx`

```js
const { io, events, config } = svrx;
```

### io

Responsible for communicating with the server

#### - `io.on(type, handler)`

Listening server-side message

**Usage**

Server side

```js
io.emit('hello', 1);
```

Client side

```js
const { io } = svrx;
io.on('hello', payload => {
  console.log(payload); //=>1
});
```

> **Note that the `io.emit()` in server-side is a broadcast**
> and all pages will receive a message for server.

**Param**

- `type`: message type
- `handler(payload)`: message handler
  - `payload`: message payload passed by `io.emit`

#### - `io.emit(type, payload)` {#client-io}

Send client message to server side

**Usage**

Client side

```js
const { io } = svrx;
io.emit('hello', 1);
```

Server side

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

- `type`: message type
- `payload`: message data passed to message handler

> `payload` must be serializable because it is transmitted over the network.

#### - `io.off(type[, handler])`

Remove io watcher

**Usage**

```js
io.off('hello'); //=> remove all hello handlers
io.off('hello', handler); //=> remove specific handler
```

#### - `io.call(name, payload)`

[io.call](#call) on the client side is exactly the same as the server, but make sure the payload is serializable\*\* because it will be transmitted over the network

**Usage**

```js
io.call('hello.world', 'svrx').then(data => {
  console.log(data);
});
```

**Param**

- `name` \[String]: service name
- `payload` \[Any]: service payload

**Return**

Promise

### events

This part is exactly the same as [server events](#events), no retelling

**Usage**

```js
const { events } = svrx;
events.on('type', handler);
events.emit('type', 1);
events.off('type');
```

> The difference between events and io is that
> io is used for the communication between the server and the client,
> but events is a single-ended communication.

### config

`config` in client is almost identical to the server, the only difference is: the client version is asynchronous (because of network communication)

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

> _if you need to get global config，just add prefix `$.`_

#### - `config.set`、`config.splice`、`config.del`

The above three methods are the same as `get`, which is consistent with the server, but the return value becomes Promise.

**Usage**

```js
config.set('a.b', 1).then(() => {
  config.get('a.b').then(ab => {
    console.log(ab); // => 1
  });
});
```

### config schema {#schema}

Svrx config schema is based on [JSON Schema](https://json-schema.org/)

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

**Field Details**

- `type` \[String]: [JSON-Schema field type](https://json-schema.org/understanding-json-schema/reference/type.html) ，can be an `array`,`string`,`number`,`boolean`,`object` or `null`
- `default` \[Any]: default value
- `required` \[Boolean]: whether it is required, default is`false`
- `properties` \[Object]: child fields
- `ui`: svrx extension ，whether show the config in [svrx-ui](https://github.com/x-orpheus/svrx-ui)
- `anyOf`: Choose one of the items, such as
  ```of
  route: {
    description: 'the path of routing config file',
    anyOf: [{ type: 'string' }, { type: 'array' }],
  },
  ```
  > For further understanding, please refer to [Official Documentation](https://json-schema.org/)

## How to test plugin? {#test}

It is not recommended that you publish the test plugin to npm.
You can do local testing in the following ways.

```js
svrx({
  plugins: [
    {
      name: 'hello-world',
      path: '/path/to/svrx-plugin-hello-world'
    }
  ]
}).start();
```

After specifying the path parameter,
svrx loads the local package instead of installing it from npm

## More easier plugin development

**Use Official [`svrx-create-plugin`](https://github.com/svrxjs/svrx-create-plugin)** to help you create plugins more easily
