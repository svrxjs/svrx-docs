# svrx

[![Greenkeeper badge](https://badges.greenkeeper.io/x-orpheus/svrx-docs.svg)](https://greenkeeper.io/)

svrx(server-x) 是一个渐进且易于使用的、插件化的前端开发工作台。

作为前端开发，在不同的开发需求下，一般来说我们会有一套或者多套固定的开发环境。
它可能包括本地服务器以及各种用于调试工程的小工具。
维护这样的开发环境是很麻烦的：你不仅需要单独安装每一个工具，还需要对每一个工具进行设置。
此外，针对不同的工程，你还需要有选择地去开启或关闭某个功能。

`svrx` 做的，就是利用插件化整合各种小功能的前端开发工作台。
做到一份配置对应一套开发环境， 一键启动开发服务。

## 特性预览

- 静态伺服
- 代理
- 自动重载页面
- 支持 Hot Reloading 的[路由功能](./zh/guide/route.md)
- 更多的可能： [插件化](./zh/guide/plugin.md)

## 安装和使用

你可以在[这里](/zh/usage.md)阅读使用指南。

## 官方插件

+ [svrx-plugin-webpack](https://github.com/x-orpheus/svrx-plugin-webpack):
    提供 webpack 相关功能，编译、hot-reload 等
+ [svrx-plugin-portal](https://github.com/x-orpheus/svrx-plugin-portal): 
    自动生成一个外部能够访问的域名映射到本地服务
+ [svrx-plugin-weinre](https://github.com/x-orpheus/svrx-plugin-weinre): 
    基于 weinre，在电脑上远程调试手机页面
+ [svrx-plugin-eruda](https://github.com/x-orpheus/svrx-plugin-eruda): 
    基于 eruda，直接在手机上开启 dev tool 进行调试
+ [svrx-plugin-qrcode](https://github.com/x-orpheus/svrx-plugin-qrcode): 
    在页面上或 console 中展示当前页面的二维码
+ [svrx-plugin-markdown](https://github.com/x-orpheus/svrx-plugin-markdown): 
    展示 markdown 类型的文件，提供实时预览、hot-reload、自动滚动定位等功能
+ [svrx-plugin-nei](https://g.hz.netease.com/cloudmusic-frontend/svrx-plugin-nei): 
    直接在开发页面一键开启某个 [NEI](https://nei.netease.com/) 接口的 mock 功能
    
