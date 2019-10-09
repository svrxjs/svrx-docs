# 结合主流脚手架使用

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


结合 [插件](../plugin/usage.md) 使用更香哦 (ˇˍˇ)