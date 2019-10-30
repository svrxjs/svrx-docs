# Getting Started


Let's get started with svrx now!

## Install svrx

We strongly recommend to use official cli tool to manage `svrx`.
You can use `svrx` at any place in your system after installing `@svrx/cli` globally. 

```bash
npm install -g @svrx/cli
```

HINT: `@svrx/cli` is used to install and load `svrx` with a specific version(default to the latest one),
which has nothing to do with the version of `@svrx/cli` itself. 
You can specify the svrx version you want in command line or the `.svrxrc.js` file.  

`@svrx/cli` stored the versions of `svrx` into `~/.svrx`, you can use another directory by setting the `SVRX_DIR` environment variable.

## Starting A Project With Svrx

### Common Commands

There are several commands in `@svrx/cli` you might use frequently:

```bash
svrx                    # start svrx service
svrx serve              # same as svrx ('serve' can be omitted)
svrx --version          # check the version of cli and svrx currently use
svrx --help             # print help info
```

### Usage 

#### Start svrx

Before we start, you need to cd into the root of your project first.
Let's say you've already got an `index.html` in your project:

```bash
cd your_project
ls # index.html
```

And without any other config, just run `serve` command to start the dev server: 

```bash
svrx
```

Then visit http://localhost:8000 to see the content of `index.html`.

#### Command Line Options

You can pass options to change the default behavior through command line:  

```bash
svrx --port 3000 --https --no-livereload
```

Check out the full option reference doc [here](./guide/option.md).

#### .svrxrc.js

And also, you can write down all your options by creating a file named `.svrxrc.js` or `svrx.config.js` in the root path of your project.

```js
// .svrxrc.js
module.exports = {
  port: 3000,
  https: true,
  livereload: false
};
```

And then run `serve` command, `svrx` will read your options from the config file automatically.

#### Global Settings

Sometimes, you might want some global settings on svrx.
Let's say, all your projects need `https` protocol,
it's really unnecessary to set `https` to `true` in every project.
You can put the global config file to the directory in which we stored the svrx packages,
the default path is, as mentioned before, `~/.svrx/config`.

```js
// ~/.svrx/config/.svrxrc.js
module.exports = {
  https: true,
};
```

Yes, the structure of global config file is same as those in each project.
And next time, wherever you start svrx, the `https` protocol will be enabled by default.

Moreover, the global configs have the lowest priority,
you can overwrite global configs in your project at any time.

### Other Commands

If you want to know the installed versions of `svrx`, 
or want to install a specific version, you might need the following commands:

```bash
svrx ls                 # list all installed versions of svrx
svrx ls-remote          # list all published versions of svrx
svrx install <version>  # install a specific <version> of svrx
svrx remove <version>   # remove a specific <version> of svrx
```
