# 说说 Server-X 的免安装插件机制

![](https://p1.music.126.net/lXoJC7CSdxC0nzhOPxn93g==/109951164693223169.jpg)

[Server-X](https://svrx.io/) 是云音乐前端团队开发的一款轻量级本地开发服务器，本文将以 Server-X 的免安装插件机制为例，向大家介绍一种**插件的免安装方案**。如果你对 Server-X 感兴趣，建议先阅读这篇文章 [《Server-X：一款可能提升你十倍工作效率的工具》](https://docs.svrx.io/zh/blog/introduction.html) ，以了解 Server-X 的基本使用方式。当然，没阅读也没关系，本文仅仅是**讨论一般插件平台的插件安装、加载和管理方式**，不需要任何背景知识，你把 Server-X 当做一个插件化的简单的本地开发服务器就好啦。

## 飞速浏览下 Server-X 插件的使用姿势

首先，请允许我花一秒钟介绍下 Server-X 的插件使用方式：

```bash
# 在你的工程目录命令行中输入
svrx --localtunnel # 启动 Server-X，并使用 localtunnel 插件 
```

没了。是的，**声明即使用，完全不用安装**，就是这么简单。

这里的 [localtunnel](https://github.com/svrxjs/svrx-plugin-localtunnel) 即为 Server-X 的一个插件（它的作用是实现内网穿透），运行这条命令后，我们会开启一个 Server-X 的本地服务以及一些核心的功能，然后自动下载 / 加载 `localtunnel` ，并启动这个插件。

当然，你还可以一次启动多个插件：

```bash
svrx --webpack --markdown --qrcode
```

你可能注意到了，在使用插件的过程中，我们并没有要求用户去下载任何插件，用户声明插件即可使用。这和一般的插件平台使用插件的方式是不同的，比如我们使用 webpack 的插件时，需要先修改 `package.json` 文件，把这些插件下载到本地工程的`node_modules` 中，再去配置文件里声明这些插件，就像这样：

![webpack 插件](https://p1.music.126.net/GEiNu84BbGuKZigH3Si0Pw==/109951164638588357.png)

这样的插件使用方式确实有点繁琐，而 Server-X 的插件不需要下载（也几乎不需要配置），难道我们把所有插件都放到 Server-X 的包里了吗？这当然不可能！实际上，为了减少用户使用路径，同时让用户的工作目录保持整洁，**Server-X 内置了一整套插件包管理逻辑**。正是得益于此，用户可以不再关心任何插件包相关的操作，不需要下载安装，不需要更新插件，更不需要卸载插件，一切的一切，都交给 Server-X 来处理。

下面呢，我们就具体来介绍下 Server-X 的这套免安装插件包管理机制。

## 包下载

由于我们的插件都是一个个 npm 包，所以我们只需要考虑如何下载一个 npm 包。最初我们的想法是以库的形式引入 `npm` ，然后安装插件包：

```js
const npm = require('npm');
npm.install();
```

但是这样有个很大的性能问题，`npm` 包的大小有约 25 M，这对于一个命令行工具来说很不 OK。

于是我们想，既然 Server-X 是依赖 `node` 环境的一个本地服务器，那么用户的本地肯定有 `npm`，我们能不能利用本地的 `npm` 来下载插件包呢？答案是肯定的：

```js
const npm = require('global-npm');
npm.install();
```

这里我们可以使用 [global-npm](https://www.npmjs.com/package/global-npm) 或者其它类似的包，他们的作用是**根据环境变量信息找到并加载本地的 `npm`**。这样，我们的 Server-X 包大小就得到了完美的“大瘦身”。

## 存储

插件包下载后，存储位置也是一个问题。默认地，`npm` 会把下载的包存放在当前目录的 `node_modules` 中。在 Server-X 的使用场景里， 包管理器默认会把插件包文件存放在用户工程项目的 `node_modules`，这样的好处就是插件包做到了工程粒度的隔离。但是，由于插件包是由 Server-X 下载的，而且肯定，我们不会把插件作为一个 `devDependency` 添加进用户工程目录下的 `package.json` 文件，这会修改用户的文件，不符合我们的预期。由此就产生了一个矛盾，即 `node_modules` 中存在插件包，但是 `package.json` 中又没有声明插件包。

这么乍一看其实是没有问题的，如果用户安装了所有工程依赖和 Server-X 插件，是可以正常启动 Server-X 运行工程的。但是这里有一个 npm 冷知识：对于在 `node_modules` 中存在，但又没有在 `package.json` 中声明的依赖，npm 在执行 `install` 命令时，会对它们进行剪枝（prune）操作。这是 npm 的一种优化，即如果某些依赖没有被事先声明，那它们就会在下一次 `install` 操作中被移除。

![npm remove packages](https://p1.music.126.net/cgZpeXB_KXte_oWx3cxu4w==/109951164628574489.png)

所以，一旦用户在某个时候又运行了一次 `npm install xxx`， 比如新增一个工程依赖，或者新增一个 Server-X 插件（前面讲过插件其实也是用 `npm install` 来安装的），就会有之前某些已安装插件的依赖被 npm 移除！这就导致我们在下一次运行 Server-X 启动工程时会收到依赖丢失的报错。

正是因为 npm 的这个特性，我们不得不放弃将插件包存储在用户工程 `node_modules` 目录中的方案。经过仔细思考和设计，我们最终决定**全局存储插件包**，将 `~/.svrx/plugins` 目录（当然用户也可以修改成别的目录）作为插件包的存放地址，里面的插件包将按照 `${插件名}/${version}` 的路径存放，如：

```bash
# ~/.svrx/plugins
├── localtunnel
│   └── 1.0.1
├── vconsole
│   └── 1.0.0
└── webpack
    ├── 1.0.1
    └── 1.0.2
```

如此我们的插件包便逃过了 npm 的“误伤”，不过，因为存储位置的改变，插件包的加载逻辑也要做相应的调整。

## 加载

Server-X 的插件加载逻辑其实有点复杂，不过我们可以用一个简化版的流程图来帮助理解：

![简化流程图](https://p1.music.126.net/KovN-m9rdz2w63LPP3oqdg==/109951164631391726.png)

首先，Server-X 支持用户通过直接传入 `path` 参数来指定某个插件包路径的方式启动插件，并且显然，用户就是上帝，永远优先级最高（手动狗头），所以我们先对 `path` 参数进行了判断。如果存在该参数，我们直接从这个路径加载插件包。

然后，Server-X 作为一个本地开发服务器，工作区划分肯定是以工程为粒度的。所以我们也应尊重工程本地的插件依赖包：如果 `node_modules` 中存在该插件包（主要是用户手动安装的情况），那我们就直接加载这个工程中的插件包。

最后，上一节我们提到，插件包是被全部托管在一个全局文件夹中的，可以说 99% 的情况下，我们的插件都是从这个文件夹加载的。内部逻辑简单来说就是：查询文件夹中是否存在该插件，**有则加载，无则下载**最佳（一般是最新）的一个插件版本。不过这里其实还有个细节要考虑，那就是用户如果指定了插件的版本号，我们还需要判断全局文件夹中是否存在相应版本的插件，如果没有，我们需要下载该版本。

以上其实是 Server-X 插件加载的一个简化版流程，复杂的部分——如果你同时也在思考的话，可能隐隐约约也会觉察——比方说在文件夹中查询插件时，真的只是简单判断文件存在与否吗？默认总是加载最新版本的插件吗？这些问题，我们将拆成后续几个小节慢慢说。

## 插件包与核心包的版本匹配问题

每个插件平台一定比较头疼的一个问题，就是用户所用的核心包（一般来讲就是插件平台本身）与插件包的版本匹配问题。有时候核心包有大更新（BREAKING CHANGE）时，旧的插件包的版本不一定能匹配上，反之亦然。于是我们肯定希望，在出现版本不匹配问题时，能对用户作出提示，并且，像 Server-X 这样的插件免安装管理模式，应该能自动根据核心包版本匹配并安装相应的插件，理论上用户根本不会感知核心包或者插件包有版本这一概念。

要做到自动匹配核心包和插件包，首先我们需要想办法将它们的版本关联起来。Server-X 采用的是插件开发者声明的方法，即插件开发者可以在插件 `package.json` 中的 `engines` 字段下声明插件正常运行所需的 Server-X 环境，如：

```json
{ "engines" : { "svrx" : "^1.0.0" } }
```

由于 `package.json` 中的字段可以在下载这个包之前直接由 `npm view` 命令读取，

![npm view engines](https://p1.music.126.net/maga5UyX16fGlRIAQTX_1A==/109951164635357391.png)

我们就可以结合当前用户使用的 Server-X 核心包版本轻松判断出最佳匹配的插件版本，再对该版本进行下载。版本匹配这里我们选择了 [semver](https://www.npmjs.com/package/semver) 来做判断：

```js
semver.satisfies('1.2.3',  '1.x || >=2.5.0 || 5.0.0 - 7.2.3')  // true
```

所以，在上一节讲述的插件加载流程里，当用户没有指定具体版本时，我们加载的目标插件包并不一定是该插件的最新（latest）版本，而是**根据 `engines` 字段做了 semver 检查后最匹配的一个版本**。

## 自动更新

在大家了解了 Server-X 的插件机制后，我们经常被问的一个问题是，插件包更新了怎么办？实际上，这是任何插件机制都会遇到的问题。一般的解决方案是，比如 webpack 的插件，我们安装插件时会把版本信息写到 `package.json` 中，如 `html-webpack-plugin@^3.0.0`，这样，当 `v3.1.0` 发布后，我们“下次重新安装”这个插件包时，可以自动更新成最新的版本。但是请注意，“下次重新安装”指的是我们移除本地依赖后重新安装这个 npm 包，然而实际使用过程中，我们并不会频繁去更新这些工程中的依赖，所以绝大多数情况下，我们没有办法及时享受到最新版本的插件。这是用户自行安装插件都会面临的问题。

那如果是插件免安装机制呢？我们是不是可以每次加载都默认加载最新版本？当然可以，因为加载的具体版本可以由内部的加载机制决定。但是，这样做有一个弊端：如果每次加载插件都去判断（npm view）一次该插件是否有最新版，有最新版还要下载（npm install）新的版本包，太浪费时间了！等所有插件都加载好，服务启动，黄花菜都凉了！

怎么才能做到自动更新插件的同时，又不拖慢加载速度呢？于是 Server-X 采用了一个折中的方案，我们在每次服务启动后，才对所有使用中的插件做版本更新检查、新版本下载，并且这一切都是**在子进程中进行**的，绝不会阻塞服务正常运行。这样，下一次启动 Server-X 时，这些插件就都是最新版本了。

不过这样的方案仍然有一些细节需要注意，比如，包下载出错怎么办？在下载过程中用户突然中断程序怎么办？这些特殊情况都会造成下载的插件包文件不完整、不可用。对此我们可以尝试**使用临时文件夹作为插件包下载的暂存区**，等确认插件包下载成功后再将临时文件夹内的文件移动到目标文件夹（`~/.svrx/plugins`）中，就像这样：

```js
const tmp = require('tmp');
const tmpPath = tmp.dirSync().name;   // 生成一个随机临时文件夹目录
const result = await npm.install({
    name: packageName,
	path: tmpPath,  // npm 下载到临时文件夹 
	global: true,
	// ...
});  
const tmpFolder = libPath.join(tmpPath, 'node_modules', packageName);  
const destFolder = libPath.join(root, result.version);  
  
// 复制到目标文件夹  
fs.copySync(tmpFolder, destFolder, {  
  dereference: true, // ensure linked folder is copied too  
});
```

所以如果插件下载失败，我们的目标文件夹中就不会存在这个插件包，于是下一次启动时我们会尝试重新下载该插件。下载失败的部分插件包呢，由于是在临时文件夹中，会定期被我们的系统清理，也不用担心垃圾残余，超级环保！

## 过期包清理

其实，我们真正需要关心的残余文件，是那些已经存在于插件文件夹中的、过期的插件包。因为自动更新机制的存在，我们会在每次插件更新后下载新的版本存放到插件文件夹中，下一次也是直接启动新的插件版本，这样一来老版本的插件包就没用了，如果不能及时清理，可能会占用用户的存储空间。

具体的清理逻辑也很简单，就是在做自动更新这一步的同时，**找出插件存储目录中版本不是最新且不是当前用户指定的版本**，然后批量删除文件夹。此外，考虑到工程师洁癖，Server-X 本身也有**自清理**逻辑，如果用户卸载了 Server-X，那么 Server-X 会自动清除所有存储在本地的配置、核心包和插件包，可以说基本做到了零污染。

## 总结

以上呢，就是 Server-X 对于插件包管理的一些设计细节和踩坑总结了。如果你只看标题和粗体加黑文字的话，那么你就会发现其实不看其它文字好像也还 OK？（嘿嘿嘿～） 总的来说，从包的下载、存储、加载，到版本管理，其实都存在了一些开发前我们可能没考虑周全的问题，如果你正好也打算做一个插件平台，或是遇到了类似的场景，希望 Server-X 的这套免安装插件包管理机制能对你有所帮助。

## Links

> 可以 star 支持一下嘛

- [Server-X 包管理  GitHub 主要源码](https://github.com/svrxjs/svrx/blob/master/packages/svrx-util/lib/package-manager/package-manager.js)
- [Server-X 项目主页](https://svrx.io/)
