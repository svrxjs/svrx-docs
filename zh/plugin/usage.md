# 插件的使用

## 通过命令行

你可以通过命令行的方式去使用插件，例如：

```bash
svrx --plugin markdown -p qrcode # -p 是 --plugin 的缩写
svrx --markdown --qrcode         # 在命令行中设置某个插件名为 true 也可以快速开启一个插件
svrx --plugin "qrcode?ui=false"  # 可以用 name?querystring 进行插件传参
svrx --plugin "webpack@0.0.3"    # name@version 可以声明插件版本
```

## 通过`.svrxrc.js`配置

同样的，你也可以通过配置 `.svrxrc.js` 中的 `plugins` 字段来启用或配置插件，如：

```js
// .svrxrc.js
module.exports = {
  plugins: [
    'markdown',
    {
      name: 'qrcode',
      options: {
        ui: false,
      },
    },
    {
      name: 'webpack',
      version: '0.0.3',
    },
  ],
};
```

然后在项目根目录执行`svrx`命令， `svrx`会自动帮你安装`markdown`，`qrcode`和`webpack`插件，并运行它们。

## scope

svrx 插件支持 [npm-scope](https://docs.npmjs.com/misc/scope)。

在使用这样的 scope 插件时，你可以通过 `@<scope>/<name>` 的格式来声明这个插件。

比如有一个插件包名为 `svrx-plugin-foo`，且它的 scope 为 `bar`，那么你可以通过下面的方式使用它：

```bash
svrx --@bar/foo 
svrx -p @bar/foo
svrx -p "@bar/foo@0.0.3"
```

或者：

```js
// .svrxrc.js
module.exports = {
  plugins: [
    '@bar/foo',
    {
      name: '@bar/foo',
      options: {
        bar: false,
      },
    },
  ],
};
```

> 你可以使用官方脚手架 [svrx-create-plugin](https://github.com/svrxjs/svrx-create-plugin) 一键创建带有 scope 的插件。 
