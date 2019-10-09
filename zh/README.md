# svrx

svrx(server-x) 是一个渐进且易于使用的、插件化的前端开发工作台。

作为前端开发，在不同的开发需求下，一般来说我们会有一套或者多套固定的开发环境。
它可能包括本地服务器以及各种用于调试工程的小工具。
维护这样的开发环境是很麻烦的：你不仅需要单独安装每一个工具，还需要对每一个工具进行设置。
此外，针对不同的工程，你还需要有选择地去开启或关闭某个功能。

`svrx` 做的，就是利用插件机制来整合各种前端开发服务，
让前端开发者可以自由挑选所需的功能，如静态伺服、代理、远程调试等，且无需关心这些功能插件的安装过程。
有了`svrx`这样一个前端开发工作台，我们可以轻松做到一份配置对应一套开发环境，实现真正的一键启动开发服务。

## 特性预览

- 静态伺服
- 代理
- 自动重载页面
- 支持 Hot Reloading 的[路由功能](./guide/route.md)
- 更多的可能： [插件化](./plugin/usage.md)

## 安装和使用

你可以在[这里](./quick-start.md)阅读使用指南。

## 官方插件

+ [svrx-plugin-webpack](https://github.com/svrxjs/svrx-plugin-webpack):
    提供 webpack 相关功能，编译、hot-reload 等
+ [svrx-plugin-localtunnel](https://github.com/svrxjs/svrx-plugin-localtunnel): 
    暴露你的本地服务为公开的外网域名
+ [svrx-plugin-weinre](https://github.com/svrxjs/svrx-plugin-weinre): 
    基于 weinre，在电脑上远程调试手机页面
+ [svrx-plugin-eruda](https://github.com/svrxjs/svrx-plugin-eruda): 
    基于 eruda，直接在手机上开启 dev tool 进行调试
+ [svrx-plugin-qrcode](https://github.com/svrxjs/svrx-plugin-qrcode): 
    在页面上或 console 中展示当前页面的二维码
+ [svrx-plugin-markdown](https://github.com/svrxjs/svrx-plugin-markdown): 
    展示 markdown 类型的文件，提供实时预览、hot-reload、自动滚动定位等功能
