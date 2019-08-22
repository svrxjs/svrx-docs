# API Reference

### root

`string`

Where to start svrx. Default to the current working directory.

### svrx

`string`

The version of svrx you want to use.
Default to the latest version installed locally, if not installed, it will the use the latest published version.

### port

`number`

Specify a port number to listen for requests on, default to 8000.

### https

`boolean`

Enable/disable https service. Default to `false`.

### route 

`string`

Specific the configuration file of __routing__, see [routing dsl guide](./route.md) for more detail.   

```js
svrx --route route.js
```

It supports __hot-reload__, which means that you can update your routing rules without restarting svrx manually.


### livereload

`boolean`, `object`

Enable/disable auto page live reload.
Livereload is enabled by default.

#### livereload.exclude

`string`, `string[]`

Specify patterns to exclude from file watchlist. 
If a file matches any of the excluded patterns, the file change won’t trigger page reload.

### serve

`boolean`, `object`

The set of dev server options. 

#### serve.base

`string`

Tell the server where to serve static content from. By default, we'll looking for contents at the current working path. 

This option is necessary only when you want to serve static files. 


#### serve.index

`string`

Name of the index file to serve automatically when visiting the root location, default to `index.html`.

#### serve.directory 

`boolean`

Enable/disable `serveIndex middleware`. `directory` is enabled by default.

When visiting the root location, if there's no index file exists, `serveIndex middleware` displays a view of filelist in the directory instead of a 404 page.


###  open

`boolean`, `string`

是否在 svrx 启动后自动打开浏览器， 默认是自动打开本地地址`http://localhost:${port}`。
 
你也可以用`open`指定需要打开的页面：

- `false`: 禁用自动打开浏览器
- `true`: 同 `'local'`
- `'local'`: 打开本地地址，如`http://localhost:${port}` 
- `'external'`: 打开外部，`http://10.242.111.80:${port}/` ( 根据你的内网 IP )
- `'home.html'`: 同`'local/home.html'` 打开 `http://localhost:${port}/home.html` 


### historyApiFallback

`boolean`, `object`

开启/关闭`historyApiFallback middleware`，默认是关闭的。

如果你的 app 使用了[HTML5 History API](https://developer.mozilla.org/en-US/docs/Web/API/History)，那么你可能需要开启这个选项。
`historyApiFallback middleware`会在请求 404 后返回`index.html`页面。

### proxy

> proxy 也支持在 [route 文件中动态配置](./route.md#proxy)

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
