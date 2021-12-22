# Server-X: A Pluggable Platform For Local Frontend Development 

[![](https://svrxjs.github.io/svrx-www/assets/images/banner.png)](https://svrxjs.github.io/svrx-www/)

> This page will introduce a brand-new frontend devtool, 
hope it can save you from switching among dev tools, 
while still bring you the same  experience.

I know what you are thinking, we're not newbies, 
as ~~bald~~ experienced frontend programmers, 
we all have several sets of development environment on our computers. 
So why should we change to another devtool?

Look, you are almost there!

So let's do a count game together. 
For the better frontend development experience, 
what exactly is the number of tools and apps you have installed on your computer? 
I guess there're local server, remote debug tool, proxy tool, broswer plugins and etc.
It must cost you a long period of time, 
and must be a hard choice to finallly decide to compose them as a stable development envelopment. 

![20 Tools You MUST Use](https://p1.music.126.net/9zJIuS5vYEV5jgNe0qEubw==/109951164432299270.png)

Besides, most of those tools require a global installation, which means, 
if you got a new computer, you have to install all of those tools back.
And your devtools cannot configure projects separately, usually, 
you have to change your devtool's configuration after switching projects. 

Let's sum up those trouble scenes we're used to during project development:  

- You have to install kinds of tools and apps for local development envelopment
- The local development envelopment is hard to copy or share between computers
- You need to configure tools one by one, and most of them are not isolated among projects
- Sometimes you find NO tools to meet your requests, and it's troublesome to write your own tools
- During developing, you have to start many tools at same time: dev server, mock server and so on
- And you often need to restart those tools to refesh configuration
- ... ...

Based on the pain points above, here comes server-x.

## What is server-x

Like the first half of the name, `server`, you can say, server-x(short as 'svrx') is just a local server,
and it's a powerful, handy and light-weighted server.

Before everything, let's have a look at the simplest use case:

Firstly you need to install the CLI tool of svrx,

```bash
npm install -g @svrx/cli
```

And create a simple page, then start svrx at the root path,

```bash
mkdir example && cd example
echo '<html><body>Hello svrx!</body></html>' > index.html
svrx
```

Visit http://localhost:8000, you can see your page.

![start demo](https://p1.music.126.net/Pq_Ck_EyFtW8lXfgTN09Mg==/109951164417568606.png)

It's convenient to install, quick to start, and except NodeJS, **it doesn't depend on any environment**.
Of course, that's the key feature of any independent and basic dev servers.

So can svrx do more? The answer is yes. 
Svrx also has some practical built-in features like broswer auto openning, page livereloading, proxy and so on.
Ok, you might say, some dev servers can do this too.

Actually, the biggest difference between svrx and other local servers is, 
as the other half of its name `server-x`: `x`.
According to the Internet, `x` can represent "unknown and infinity", 
you can say svrx is a server with infinite possibilities. 
Why? Because there's a most outstanding feature in svrx: it's a platform of plugins.

Through plugins, your svrx can have any function in theory.
Every small function here is an independent plugin, 
you just need to declare them before use, like this: 

```bash
svrx --webpack --qrcode --markdown
```

It's very clear, and with no redundant configuration. 
After your declaration, svrx will help you install and load plugins, then start the service directly.

As you see, svrx is a platform that aggregates plenty of plugins. 
It can be a whole development environment itself. 
The only difference is, **you don't need to care about the plugin installation at all**.
Except the CLI of svrx, **you don't need to install any other tools**.

And what's more, none of plugins is installed globally. 
Svrx installs plugins into the `node_modules` directory of your project', 
so **everything is truly independent**, 
you can customize the development environment for each project 
without concerning about the installation or uninstallation, 
there's no contamination, and your system can keep clean at the same time.

Actually, there're many local dev servers published.
But nearly none of them is like svrx, 
which is light-weighted, easy to use, with complete plugin system and not rely on any project environment. 
Next, let's keep exploring the development experience of svrx through creating a simple frontend project.
You'll see some advanced usages and black-techs, 
that's the most charming part of svrx.

## Init A New Project And Start

Now, we will use [Create React App](https://github.com/facebook/create-react-app) to create a new project.
(As mentioned before, svrx does not rely on any project environment, it's only for the convenience we choose CRA)

```bash
npm init react-app svrx-example
cd svrx-example
```

By default the new project uses `webpack` to bundle code, to start projects like this, 
we need a plugin named [svrx-plugin-webpack](https://github.com/svrxjs/svrx-plugin-webpack).
The purpose of this plugin is to make your `webpack` project work in svrx.
Basically it will read project webpack config, 
and then call [webpack-dev-middleware](https://github.com/webpack/webpack-dev-middleware).

But our new project here is not exposed the config of `webpack`, 
so firstly we need a `webpack.config.js` at the root:

```js
// webpack.config.js
module.exports = require('react-scripts/config/webpack.config')('development');
```

And now we can start our project successfully:

```bash
svrx --webpack
```

Your broswer will auto open http://localhost:8000/  for you: 

![start svrx](https://p1.music.126.net/ObArEQVEbjajRBkeOaJldQ==/109951164429015297.png)

Try edit `src/App.css`, check if the page auto reloaded?

![livereload](https://p1.music.126.net/yIQ1lg2pOhA1ETmGKvQvIQ==/109951164430535611.gif)

## Advanced Usage 1: Configure Your Svrx

By default, svrx will enable some built-in plugins when start, such as 
static serve, proxy service, page livereloading and etc. 
All of those plugins have some default configuration which ensure that svrx can be quickly started. 
Of cource, if you want to make some custom decision, there're two ways.

Firstly, you can pass some params to CLI when starting svrx:

```bash
svrx --port 3000 --https --no-livereload
```

And also, you can create a `.svrxrc.js` or a `svrx.config.js` at the root path of your project, 
then run `svrx` to start your project directly:

```js
// .svrxrc.js
module.exports = {
  port: 3000,
  https: true,
  livereload: false
};
```

All of the options and details can be found at [svrx docs - options](https://svrxjs.github.io/svrx-docs/en/guide/option.html).

##  Advanced Usage 2: Try Other Plugins

Besides built-in plugins, there're also many other independent plugins, 
like `svrx-plugin-webpack` mentioned before.  
When you need other development function(such as remote debug, mock and so on),
all you have to do is just declare the plugin name. 
In fact, it is the independent plugins that provides feature-rich experience.
Next, we will introduce several useful and interesting plugins:

### [localtunnel](https://github.com/svrxjs/svrx-plugin-localtunnel) - Expose Your Local Service

Image that you're working on your page developing intently, suddenly there's a message toast from your boss:

> Let me see how's the page going on?

What do you gonna do? 
I guess you might commit your code first, 
deploy a local service, and give your IP address of LAN to your boss.
Hold on, what if your boss is on a business trip right now, he doesn't have access to your LAN! 
Ok, there's still a plan. You find a server and deploy a test environment for your boss, 
though the deploy might be slow and your boss has to wait for a long while.

Any other solution? Yes, you need [localtunnel](https://github.com/localtunnel/localtunnel) plugin of svrx!
It exposes your local service to `localtunnel.me`, 
so that you can test and share your local code with others easily.
You don't need to deploy your test environment over and over again just for a little code change.

To start `localtunnel`, just add the plugin name at the end:

```bash
svrx --webpack --localtunnel
```

This command will auto install plugin localtunnel and start svrx, 
other people(yes, you don't have to be in a same LAN) 
now can visit the url https://*.localtunnel.me printed by terminal to see your local service:

![localtunnel](https://p1.music.126.net/0tnJ_DkmfTEl_jFdRGepbQ==/109951164429088111.png)

Besides, every local changes made by you can be seen by others, don't worry about the boss anymore.

![localtunnel-livereload](https://p1.music.126.net/Wc2rVrS4qNiUWSl1avHc9Q==/109951164430527349.gif)

### [weinre](https://github.com/svrxjs/svrx-plugin-weinre) - Debug Your Code Remotely

How do you debug your code on the phone? Many developers will choose chrome remote debugging.
But to use it, you need to connect your computer and your mobile phone with a USB first.

Is there a better way? Try plugin [weinre](http://people.apache.org/~pmuellr/weinre/docs/latest/Home.html) of svrx.
It is used to debug your code remotely, and it's wireless.

Let's be back to our example peoject, this time we add two new plugins: 

```bash
svrx --open=external --webpack --weinre --qrcode 
```

Now we start svrx with weinre and qrcode,
and use your phone to visit the project page, 
plugin [qrcode](https://github.com/svrxjs/svrx-plugin-qrcode) here is just to make it easier for your phone to visit the page:

![qrcode](https://p1.music.126.net/xIVX3rkv1dsvrUOlFRXldQ==/109951164430529750.gif)

Then open the debug page of weinre using your computer, the default url is http://${your_ip}:8001,
find the record of your phone, and you can debug your page on the phone now.

![weinre debug tool](https://p1.music.126.net/KaBNi-_EDelz8kE75i5gSg==/109951164429434114.png)

### Customize Your Plugin

Besides those we mentioned, there're many other useful plugins in svrx, 
you can search plugin at the [official site of svrx](https://svrxjs.github.io/svrx-www/plugin?query=svrx-plugin-),
and then compose different plugins to get your own development environment.

![part of plugin list](https://p1.music.126.net/IZDarDVC9sHj69lxWuhljg==/109951164417587560.png)

Of course, if you don't find a proper plugin, you can try to write one yourself.

What can you do with the plugin?
Take the plugin `qrcode` as an example, to print a qrcode on the page, 
you can inject some js and css scripts to the frontend page.
And like the plugin `webpack`, 
you can inject some koa style middlewares to the backend service,
here is the `webpack-dev-middleware`,
intercept requests and modify the responses.

With the greate power of scripts and middlewares injection, 
you can do nearly any thing during local development by creating svrx plugins.
And **it's extremely easy to create a new plugin**! 
Can you believe it? The code size of plugins we mentioned is only 50 lines in average!
Moreover, svrx also provides a command line scaffolding tool for quick plugin development, see [How To Write A Plugin](https://svrxjs.github.io/svrx-docs/en/plugin/contribution.html) for more detail.

## Advanced Usage 3: Route With Hot Reloading

Usually, as a frontend developer, you need to mock api response datas at the very beginning of the project.
So you must have run into situations like:

- change mock data, restart the mock server
- enable / disable api proxy, restart the service
- edit project source code, restart
- ... ...

Ok, even if the mock services are good enough that they don't need to restart now, 
you still need to start a mock server along side your local dev server, or,
you put your mock data into your source code, that's really NOT a good idea! 

So, here comes our dynamic route.
Besides the powerful plugin system, there's another useful and magic feature in svrx: the dynamic route.
Let's get back to our example project, you can have a try on dynamic route like:

```bash
touch route.js # create empty routing file
svrx --webpack --route route.js
```

In `route.js`:

```js
get('/blog').to.json({ title: 'svrx' });
```

Now open `/blog`, you'll see the output `{ title: 'svrx' }`.

With dynamic route, you can **quickly and clearly create your mock data** without intruding into your source code.
And it supports **hot reload**, everytime after editing `route.js`, you don't need to restart svrx, the data will update automatically.

![demo of dynamic route](https://p1.music.126.net/SJNCky1nh6RF_RIi2_kkLw==/109951164418247220.gif)

Of course, the dynamic route can do more than data mocking.
There're some examples of dynamic route:

```js
get('/index.html').to.sendFile('./index.html');
get('/blog').to.redirect('/user');
get('/old/rewrite:path(.*)').to.rewrite('/svrx/{path}');
get('/api(.*)').to.proxy('http://mock.server.com/');
get('/blog')
  .to.header({ 'X-Engine': 'svrx' })
  .json({ code: 200 });
```

As you can see, the syntax of route is very simple, 
you can directly read every actions and clearly know what they gonna do: send files, refirect, rewrite, proxy and etc.
Moreover, if the built-in route actions cannot fit your need, you can expand the route action through plugin by yourself.
Read [How To Use Routes](https://svrxjs.github.io/svrx-docs/en/guide/route.html) for more detail.

## In The Last

> A pluggable frontend server, it just works.

This is the slogan of svrx, which is sufficient to reveal the location of svrx:

- svrx is a useful local dev server, which contains some built-in plugins like static serve, proxy, livereload and etc
- svrx has a powerful plugin system, you can customize your dev environment through plugins

Svrx is trying to bring an elegant development experience to all frontend developers, at the same time, 
we also offer you a platform to quickly create your own feature plugin.
As a svrx user, you can pick suitable plugins to compose a better development environment, 
which can be quickly started without any installation. 
If there's no plugin meet your need, you can also turn to a plugin developer and write your own plugin. 
Surely your ideas can solve many other developer's problems and improve their development experience.

Later, svrx will keep working on more features and plugins of high quality, and your contributions are also welcomed!

## Links

- [svrx - the official site](https://svrxjs.github.io/svrx-www/) docs, API, plugin list
- [svrx - Github](https://github.com/svrxjs/svrx) source code, issue, pr
