# Examples

We can easily add plugins to most projects to simplify our development flow.   

Here are some examples of how to use `svrx` and some common plugins, hope it helps.

- [serve a static page](https://github.com/svrxjs/svrx/blob/master/examples/serve-static-page)
- [proxy](https://github.com/svrxjs/svrx/blob/master/examples/proxy)
- [webpack](https://github.com/svrxjs/svrx-plugin-webpack/tree/master/example)
- [use eruda dev tool on mobile](https://github.com/svrxjs/svrx/tree/master/examples/eruda)
- [weinre and remote debug](https://github.com/svrxjs/svrx/blob/master/examples/weinre)

## Work with CRA and Vue CLI

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


You can also combine with others [plugins](./guide/plugin.md) to get powerful dev experience.
