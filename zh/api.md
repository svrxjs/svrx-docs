# 参数列表

### root

`string`

启动 svrx 的路径。默认是当前工作目录。

### svrx

`string`

你想使用的 svrx 核心包的版本。默认会使用本地安装的最新版 svrx，如果未安装过，则会安装并使用当前发布过的最新版。

### port

`number`

端口号，默认是 8000。

### https

`boolean`

开启/关闭 https。默认是`false`。

### livereload

`boolean`, `object`

开启/关闭页面自动刷新功能， 默认是开启的。

#### livereload.exclude

`string`, `string[]`

设置文件忽略规则，如果文件符合任意匹配的 pattern，那么该文件的内容变动不会触发页面刷新。

### serve

`boolean`, `object`

本地服务器配置

#### serve.base

`string`

告诉服务器从哪里提供静态文件。 默认情况下，我们会先查找当前工作目录下的文件。

你只需要在你想伺服静态资源的时候设置这个选项。

#### serve.index

`string`

访问根路径时自动展示的 index 文件的文件名，默认是`index.html`。

#### serve.directory 

`boolean`

开启/关闭`serveIndex middleware`， `directory`默认是开启的。

访问根路径时，如果你的 index 文件不存在，`serveIndex middleware`可以提供一个目录中文件列表的视图，而不是返回 404。

### historyApiFallback

`boolean`, `object`

开启/关闭`historyApiFallback middleware`，默认是关闭的。

如果你的 app 使用了[HTML5 History API](https://developer.mozilla.org/en-US/docs/Web/API/History)，那么你可能需要开启这个选项。
`historyApiFallback middleware`会在请求 404 后返回`index.html`页面。

### proxy

`boolean`, `object`, `object[]`

url 转发配置。 你可以在当前域名下设置`proxy`，将不同的 url 转发至不同的后端地址。

```js
module.exports = {
    proxy: {
        '/api': {
            target: 'http://you.backend.server.com'  
        }
    },
}
```

经过如上配置后，一个`/api/path`请求会被转发至`http://you.backend.server.com/api/path`。

你也可以重写路径，就像这样：

```js
module.exports = {
    proxy: {
        '/api': {
            target: 'http://you.backend.server.com',
            pathRewrite: {'^/api' : ''} 
        }
    },
}
```

现在你的`/api/path`请求会被转发至`http://you.backend.server.com/path`。

如果 target 后端的 HTTPS 服务器的证书是无效的，那么请求默认不会被接收。
你可以配置`secure`来改变这个默认设置:

```js
module.exports = {
    proxy: {
        '/api': {
            target: 'https://you.https.server.com',
            secure: false 
        }
    },
}
```

如果你想同时把多个 path 转发到一个相同的地址，可以试试这样做：

```js
module.exports = {
    proxy: [
        {
            context: ['/api', '/wapi', '/pub'],
            target: 'http://you.backend.server.com',
        }  
    ],
}
```

如果你想改变 header 中的 origin 为目标域名，可以试试将`changeOrigin`设为`true`：

```js
module.exports = {
    proxy: {
        '/api': {
            target: 'https://you.https.server.com',
            changeOrigin: true 
        }
    },
}
```

### cors

`boolean`, `object`

开启/关闭跨域资源共享（[CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)）支持。
默认 Cors 是开启的。 

svrx 依赖[koa2-cors](https://github.com/zadzbw/koa2-cors)做跨域资源工程支持，更多使用方法和参数请阅读[koa2-cors 参数文档](https://github.com/zadzbw/koa2-cors#options)。

### registry

`string`

`npm`源地址，设置此选项后，`svrx`将从此源下载插件。 默认的`registry`值是你工作目录下的源地址值，即`npm config get registry`的结果。
