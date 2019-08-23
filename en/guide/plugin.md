# How To Use Plugins

## Command Line

You can use plugins through command line options, eg:

```bash
svrx --plugin markdown -p qrcode # -p is alias of --plugin
svrx --markdown --qrcode         # set a pluginName to true to start a plugin quickly
svrx --plugin "qrcode?ui=false"  # use 'name?querystring' to add params to a plugin
svrx --plugin "webpack@0.0.3"    # use 'name@version' to specify the version of plugin
```

## `.svrxrc.js` config file

And also, you can enable and config a plugin through `plugins` in `.svrxrc.js` file, eg:

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

Then run `svrx` in the root place of your project, we'll install plugin `markdown`, `qrcode`, `webpack`, and start them automatically.
