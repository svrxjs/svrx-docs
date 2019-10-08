# 项目实战

由于`svrx`提供了丰富灵活的插件系统，我们几乎可以在所有开发场景中使用`svrx`。 

这里我们整理了一些`svrx`及常见插件的使用场景，希望这些配置能对你有帮助。

- [在静态页面启用服务器](https://github.com/svrxjs/svrx/blob/master/examples/serve-static-page)
- [代理](https://github.com/svrxjs/svrx/blob/master/examples/proxy)
- [webpack](https://github.com/svrxjs/svrx-plugin-webpack/tree/master/example)
- [在移动端开启 eruda dev tool](https://github.com/svrxjs/svrx/tree/master/examples/eruda)
- [利用 weinre 进行远程调试](https://github.com/svrxjs/svrx/blob/master/examples/weinre)

## 配合 CRA 和 Vue CLI 使用

使用 [create-react-app](https://github.com/facebook/create-react-app) 或 [vue-cli](https://github.com/vuejs/vue-cli) 初始化项目：

```bash
# CRA
npm init react-app my-app

# Vue CLI
npm i @vue/cli -g
vue create my-app
```

项目根目录创建文件 `webpack.config.js`：

```js
// CRA
module.exports = require('react-scripts/config/webpack.config')('development');
```

```js
// Vue CLI
module.exports = require('@vue/cli-service/webpack.config');
```

好了，就这些 🎉

```bash
svrx --webpack
```

![image](https://user-images.githubusercontent.com/2230882/65511690-5299f800-df0a-11e9-95ca-ff88cd65b4ef.png)

![image](https://user-images.githubusercontent.com/2230882/66377471-43c04480-e9e4-11e9-84c3-ccca43c00766.png)


结合 [插件](./guide/plugin.md) 使用更香哦 (ˇˍˇ)