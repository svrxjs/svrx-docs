# Option Reference

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

Enable/Disable https service. Default to `false`.

### route 

`string`

Specify the configuration file of __routing__, see [routing dsl guide](./route.md) for more detail.   

```js
svrx --route route.js
```

It supports __hot-reload__, which means that you can update your routing rules without restarting svrx manually.


### livereload

`boolean`, `object`

Enable/Disable auto page live reload.
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

Enable auto opening browser after svrx started. 
By default, it will open `http://localhost:${port}` automatically.
 
You can also specific the opening page by setting `open` to:

- `true`: same as `'local'`
- `'local'`: open local url, `http://localhost:${port}` 
- `'external'`: open external url, `http://${your_ip}:${port}/`
- `'home.html'`: same as `'local/home.html'`, open `http://localhost:${port}/home.html` 

Set `open` to `false` to disable auto open browser.


### historyApiFallback

`boolean`, `object`

Enable/disable `historyApiFallback middleware`. It is set to `false` by default.

This option is necessary when your app is using [HTML5 History API](https://developer.mozilla.org/en-US/docs/Web/API/History), 
`historyApiFallback middleware` will serve a `index.html` page instead of 404 responses.

### proxy

> proxy is also supported in [route configuration file](./route.md#proxy)

`boolean`, `object`, `object[]`

Proxying urls when you want to send some requests to different backend servers on the same domain.

```js
module.exports = {
    proxy: {
        '/api': {
            target: 'http://you.backend.server.com'  
        }
    },
}
```

Now a request to `/api/path` will be proxied to `http://you.backend.server.com/api/path`.

And you can also rewrite the path, eg:

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

Then your request to `/api/path` will be proxied to `http://you.backend.server.com/path`.

A backend server running on HTTPS with an invalid certificate will not be accepted by default.
If you want to, modify your config like this:

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

If you want to proxy multiple paths to a same target, try:

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

Please note that by default, the option value of `changeOrigin` is `true`, 
which means `svrx` will always set the origin of host header to the target hostname during CORS. 
If you don't want this feature, just set `changeOrigin` to `false`:

```js
module.exports = {
    proxy: {
        '/api': {
            target: 'https://you.https.server.com',
            changeOrigin: false 
        }
    },
}
```

### cors

`boolean`, `object`

Enable/disable cross-origin resource sharing([CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)).
Cors is enabled by default. 

svrx makes use of [koa2-cors](https://github.com/zadzbw/koa2-cors) package.
Check out its [option documentation](https://github.com/zadzbw/koa2-cors#options) for more advanced usages.

### registry

`string`

Specific the npm registry where `svrx` should download plugins from.
By default, `svrx` will use the registry set at your workspace, you can check with command:  

```bash
npm config get registry
```

### path 

`string` 

The local path of the svrx core package. 
ONLY in development mode, to load a local svrx core package.    
