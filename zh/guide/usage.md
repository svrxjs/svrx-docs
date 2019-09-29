# 用 svrx 启动你的工程

## 常用指令

`@svrx/cli` 提供了以下几个常用指令：

```bash
svrx                    # 启动svrx服务
svrx serve              # 同svrx（serve命令可省略）
svrx --version          # 查看 cli 及当前使用的 svrx 版本
svrx --help             # 命令及参数提示
```

## 在工程中使用 svrx 

#### 启动服务

开始前，首先你需要进入到你的工程目录，我们假设你的工程中已经有一个 `index.html`：

```bash
cd your_project
ls # index.html
```

无需经过任何配置和传参，直接运行`serve`命令即可开启一个简单的本地服务器：

```bash
svrx
```

此时访问 http://localhost:8000 ，就可以看到 `index.html` 中的内容了。

#### 使用参数

如果需要对 `svrx` 进行配置，可以通过命令行传参来实现： 

```bash
svrx --port 3000 --https --no-livereload
```

详细的参数列表可以在[这里](./api.md)查看。

#### 配置持久化

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

## 其它指令

另外，如果你需要关心 `svrx` 的版本安装情况，或是想安装某个特定的 `svrx` 版本，那么你还可以使用下面这些指令：

```bash
svrx ls                 # 查看所有本地已安装的svrx版本
svrx ls-remote          # 查看所有已发布的svrx版本
svrx install <version>  # 安装某个<版本>的svrx
svrx remove <version>   # 从本地移除某个<版本>的svrx
```
