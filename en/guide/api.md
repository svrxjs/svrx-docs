# API Reference

### svrx(option)

Get the instance of Svrx

**Usage**

```js
const svrx = require('@svrx/svrx');

const server = svrx({
  port: 8002
});
```

**Param**

- option: [see option reference](./option.md)

**Return**

the instance of Svrx 

### server.start()

Start svrx

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

Manually refresh the browser

**Usage**

```js
server.reload();
```

## server.on

Attach event listener

### builtin events

#### 1. `ready`

Emit when svrx start

```js
server.on('ready', port => {});
```

#### 2. `plugin`

Emit after plugin building, as same as `hook.onCreate`, see [How to create plugin](../contribute/plugin.md) for more details

```js
server.on('plugin', async ({io, events, config, router, injector, logger, middleware }=>{
    // you logic here
}))
```

#### 3. `file:change`

emit after file change (make sure that livereload is enable)

## server.off

unbind specific event

```js
server.on('file:change', handler);
server.off('file:change', handler);
```

## server.emit

```js
server.emit('custom-event', { param1: 1 });
```
