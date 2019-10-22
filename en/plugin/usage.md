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

## scope

Svrx plugin also support [npm-scope](https://docs.npmjs.com/misc/scope).

You can specific a scoped plugin with `@<scope>/<name>`.

For example, if the package name of a plugin is `svrx-plugin-foo`,
and the scope name is `bar`, then you can use it like so:

```bash
svrx --@bar/foo 
svrx -p @bar/foo
svrx -p "@bar/foo@0.0.3"
```

Or:

```js
// .svrxrc.js
module.exports = {
  plugins: [
    '@bar/foo',
    {
      name: '@bar/foo',
      options: {
        bar: false,
      },
    },
  ],
};
```

> You can easily create a scoped plugin through our official tool [svrx-create-plugin](https://github.com/svrxjs/svrx-create-plugin). 
