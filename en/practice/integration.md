# Integrations

Initialize a project with [create-react-app](https://github.com/facebook/create-react-app) or [vue-cli](https://github.com/vuejs/vue-cli):

```bash
# CRA
npm init react-app my-app

# Vue CLI
npm i @vue/cli -g
vue create my-app
```

Create a file named `webpack.config.js` in root directory:

```js
// CRA
module.exports = require('react-scripts/config/webpack.config')('development');
```

```js
// Vue CLI
module.exports = require('@vue/cli-service/webpack.config');
```

And it works like a charm 🎉

```bash
svrx --webpack
```

![image](https://user-images.githubusercontent.com/2230882/65511690-5299f800-df0a-11e9-95ca-ff88cd65b4ef.png)

![image](https://user-images.githubusercontent.com/2230882/66377471-43c04480-e9e4-11e9-84c3-ccca43c00766.png)


You can also combine with others [plugins](../plugin/usage.md) to get powerful dev experience.
