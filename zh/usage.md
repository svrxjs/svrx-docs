# 使用指南

## 安装

我们推荐使用 cli 工具来管理和使用 `svrx`， 全局安装 cli 后你将可以在系统的任意地方使用 `svrx`。

```bash
npm install -g svrx-cli
```

注意： cli 的作用是安装和加载你指定的`svrx`版本<!--（默认为['最新版']()）-->，这与 cli 本身的版本无关。

`svrx-cli` 将安装的 `svrx` 包存储在 `~/.svrx`， 你可以通过设置 `SVRX_DIR` 环境变量来改变这个存储路径。

## 使用

`svrx-cli` 提供了以下几个常用指令：

```bash
svrx                    # 启动svrx服务
svrx serve              # 同svrx（serve命令可省略）
svrx --version          # 查看 cli 及当前使用的 svrx 版本
svrx --help             # 命令及参数提示
```

使用时，首先你需要进入到你的工程目录，我们假设你的工程中已经有一个 `index.html`：

```bash
cd your_project
ls # index.html
```

无需经过任何配置和传参，直接运行`serve`命令即可开启一个简单的本地服务器：

```bash
svrx
```

此时访问 http://localhost:8000 ，就可以看到 `index.html` 中的内容了。

如果需要对 `svrx` 进行配置，可以通过命令行传参来实现： 

```bash
svrx --port 3000 --https --no-livereload
```

详细的参数列表可以在[这里](/zh/api.md)查看。

当然，你也可以在你的工程目录下建立 `.svrxrc.js` 或 `svrx.config.js` 文件，将上面的命令行参数持久化下来：

```js
// .svrxrc.js
module.exports = {
  port: 3000,
  https: true,
  livereload: false
};
```

然后直接运行 `serve` 命令， `svrx` 会自动读取你的配置。

另外，如果你需要关心 `svrx` 的版本安装情况，或是想安装某个特定的 `svrx` 版本，那么你还可以使用下面这些指令：

```bash
svrx ls                 # 查看所有本地已安装的svrx版本
svrx ls-remote          # 查看所有已发布的svrx版本
svrx install <version>  # 安装某个<版本>的svrx
```
## 插件使用

你可以通过命令行的方式去使用插件，例如：

```bash
svrx --plugin markdown -p qrcode # -p 是 --plugin 的缩写
svrx --markdown --qrcode         # 在命令行中设置某个插件名为 true 也可以快速开启一个插件
svrx --plugin "qrcode?ui=false"  # 可以用 name?querystring 进行插件传参
svrx --plugin "webpack@0.0.3"    # name@version 可以声明插件版本
```

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

然后在项目根目录执行`svrx`命令， `svrx`会自动帮你安装`markdown`和`qrcode`插件，并运行它们。

## 常见使用场景

由于`svrx`提供了丰富灵活的插件系统，我们几乎可以在所有开发场景中使用`svrx`。 

这里我们整理了一些`svrx`及常见插件的使用场景，希望这些配置能对你有帮助。

- [在静态页面启用服务器](https://github.com/x-orpheus/svrx/blob/master/examples/serve-static-page)
- [代理](https://github.com/x-orpheus/svrx/blob/master/examples/proxy)
- [webpack](https://github.com/x-orpheus/svrx/blob/master/examples/webpack)
- [在移动端开启 eruda dev tool](https://github.com/x-orpheus/svrx/tree/master/examples/eruda)
- [portal - 内网穿透](https://github.com/x-orpheus/svrx/blob/master/examples/portal)
- [利用 weinre 进行远程调试](https://github.com/x-orpheus/svrx/blob/master/examples/weinre)
