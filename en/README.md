# svrx

svrx(server-x) is a pluggable platform built for efficient front-end development.

![](https://p1.music.126.net/tkkgZQepIA1mvhrv8IzbuA==/109951164358239571.gif)


As a front-end developer, to meet different kind of development requirements, usually we will have one or more set of fixed development environment,
in which may include a local dev server and many other debug tools.
It's difficult to maintain a development environment: you need to install and configure every tool separately.
Besides, you may also need to enable or disable a tool when switching among projects.

To solve the problem, we plan to integrate all the development services and tools into a pluggable platform, 

and name it `svrx`.
With `svrx`, you can freely pick and combine any services(plugins) you want, 
like static serve, proxy, remote debugging and etc, without concerning about plugin installation.
 
Now, `svrx` makes it possible for us to easily customize the development environment for each project,
and instead of downloading many other packages, all you need to do is just install `svrx`.

## Features

- static serve
- proxy
- page live reload
- [routing](./guide/route.md) with hot reloading 
- and you can do more: [plugin](./guide/plugin.md)

## Install & Usage

View [quick start](./quick-start.md) for more details.


## Official Plugins

+ [svrx-plugin-webpack](https://github.com/svrxjs/svrx-plugin-webpack):
    support of webpack, including building, hot-reload and etc.
+ [svrx-plugin-localtunnel](https://github.com/svrxjs/svrx-plugin-localtunnel): 
    expose your localhost to the world for easy testing and sharing.
+ [svrx-plugin-weinre](https://github.com/svrxjs/svrx-plugin-weinre): 
    remote debugging based on weinre.
+ [svrx-plugin-eruda](https://github.com/svrxjs/svrx-plugin-eruda): 
    open a dev tool on mobile based on eruda
+ [svrx-plugin-qrcode](https://github.com/svrxjs/svrx-plugin-qrcode): 
    display the qrcode of current page.
+ [svrx-plugin-markdown](https://github.com/svrxjs/svrx-plugin-markdown): 
    preview markdown file, support hot-reload, auto-scroll...

    
