# Starting A Project With Svrx

## Common Commands

There are several commands in `@svrx/cli` you might use frequently:

```bash
svrx                    # start svrx service
svrx serve              # same as svrx ('serve' can be omitted)
svrx --version          # check the version of cli and svrx currently use
svrx --help             # print help info
```

## Usage 

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

Check out the full option reference doc [here](./api.md).

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

## Other Commands

If you want to know the installed versions of `svrx`, 
or want to install a specific version, you might need the following commands:

```bash
svrx ls                 # list all installed versions of svrx
svrx ls-remote          # list all published versions of svrx
svrx install <version>  # install a specific <version> of svrx
```
