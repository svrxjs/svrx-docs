# API 索引

### svrx(option)

获得 svrx 实例

**Usage**

```js
const svrx = require('@svrx/svrx');

const server = svrx({
  port: 8002
});
```

**Param**

- option: [查看 option 索引](./option.md)

**Return**

Svrx 实例

### server.start()

启动 Svrx

**Usage**

```js
server.start().then(port => {
  console.log(port);
});
```

**Return**

Promise

## server.close()

**Usage**

```js
server.close().then(() => {
  console.log('Svrx has closed');
});
```

**Param**

**Return**

Promise

## server.reload()

主动刷新浏览器

**Usage**

```js
server.reload();
```

## server.on

绑定事件

### 内置事件

#### 1. `ready`

在服务启动时触发

```js
server.on('ready', port => {});
```

#### 2. `plugin`

在 build 插件后触发，与插件开发的`hook.onCreate`钩子接受同样的参数，请参考[插件开发指南](../contribute/plugin.md)

```js
server.on('plugin', async ({io, events, config, router, injector, logger, middleware }=>{
    // you logic here
}))
```

#### 3. `file:change`

在文件变化后触发(必须 livereload 为 true)

## server.off

解绑事件

```js
server.on('file:change', handler);
server.off('file:change', handler);
```

## server.emit

```js
server.emit('custom-event', { param1: 1 });
```
