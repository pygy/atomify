Atomify [![Build Status](https://travis-ci.org/atomify/atomify.svg)](https://travis-ci.org/atomify/atomify) [![Gitter](https://badges.gitter.im/Join Chat.svg)](https://gitter.im/atomify/atomify?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge) [![Dependencies Up-to-date](https://david-dm.org/atomify/atomify.png)](https://david-dm.org/atomify/atomify)
===============

Atomic web development - Combining the power of npm, Browserify, Rework and more to build small, fully encapsulated client side modules

## Description

Atomify provides a centralized point of access to [atomify-js](http://github.com/atomify/atomify-js) and [atomify-css](http://github.com/atomify/atomify-css) both in code and on the command line. It also offers a server with live-reload and on-the-fly bundling support to make development a breeze.

## Examples

You can learn from examples in [this repository](https://github.com/atomify/atomify-examples)

## API

`atomify(opts, cb)`

Just like its constituent pieces, atomify is a function that takes an `opts` object and a `callback` function.

### opts

**opts.js** - Options to be passed to [atomify-js](https://github.com/atomify/atomify-js#opts)

**opts.css** - Options to be passed to [atomify-css](https://github.com/atomify/atomify-css#opts)

**opts.assets** - Used to configure `opts.js.assets` and `opts.css.assets` simultaneously (and identically). See links above.

**opts.server** - Options to be passed to the development server

### callback

Just like the callbacks used by `atomify-js` and `atomify-css`, but with a third parameter to denote the type of bundle being provided. `cb(err, src, type)` where type is either `'js'` or `'css'`.

### API Example

```js
// build.js
var atomify = require('atomify')

var jsConfig = './entry.js' // shorthand for {entry: './entry.js'}

var cssConfig = {
  entry: './entry.css'
  , variables: {
    background: '#f00'
  }
}

function cb (err, src, type) {
  if (type === 'js') {
    // do something with JS source bundle
  } else {
    // do something with CSS source bundle
  }
}

atomify({js: jsConfig, css: cssConfig}, cb);
```

If you don't need to access the bundled source simply provide a file path as the output property in your options and atomify will write the file for you.

### atomify.js and atomify.css

As a convenience, you can access `atomify-js` and `atomify-css` via properties on the `atomify` function.

```js
var atomify = require('atomify')

atomify.js === require('atomify-js')
atomify.css === require('atomify-css')
```

## Development server

Atomify includes a development server that provides things like on-the-fly bundling and live reload/browser sync support to make your workflow lightning fast. `atomify.server(opts)` provides basically the same API as `atomify` itself, with a few extra options (documented below) added in. The biggest difference, of course, is that instead of writing to a file or calling a callback function, `atomify.server` responds to http requests. The server can also be configured by including a `server` property in the `opts` object when calling `atomify(opts)`.

Just like with `atomify`, the options passed to `atomify.server` are expected to have a `js` and/or `css` field. When the `entry` option of either of these is requested, the server will return the results of bundling your code. If you don't want to include the actual path to your entry file in your HTML you can also provide an `alias` option field. When the alias path is requested the server will bundle using your `entry` path.

### opts.server

You can provide server-specific options in this field.

**opts.server.port** - Port to listen on. Default: 1337

**opts.server.open** - If provided, open the URL in your default browser

**opts.server.path** - The path to open. Appended to http://localhost:port by default. Default: /default, which is a generated HTML file that includes your CSS and JS bundles automatically.

**opts.server.url** - Full URL to open instead of http://localhost:port/path

**opts.server.hostname** - Use your machine's local hostname (via Node's `os.hostname()`) instead of localhost. Ideal for accessing pages from mobile devices on the same LAN. Aliased as h.

**opts.server.lr** - Enables live-reload support by injecting the live-reload script into any HTML pages served. Supports the following sub-properties.

 * sync: Use BrowserSync. Aliased as `s`. If provided as an object will be used as the [ghostMode](https://github.com/shakyShane/browser-sync/wiki/options#ghostmode) option for BrowserSync.
 * port: Port for BrowserSync server to run on. Default: 3000
 * patterns: Globbing patterns to pass to [browsersync](https://www.npmjs.org/package/browser-sync) for watching. Default: `['**.html']` relative to `process.cwd()` as well as all files in the dependency graph of your JS and CSS bundles.
 * quiet: Suppress file change notifications on the command line. Default: false
 * verbose: Log BrowserSync's debugInfo to the console. Default: false

**opts.server.sync** - Shortcut for specifying `opts.server.lr` as `{sync: true}`.. Aliased as s.

**opts.server.st** - Options to pass to [st](https://www.npmjs.org/package/st) static file server, which is what serves all non-entry/alias requests.

**opts.server.html** - Override the default HTML served at `/default`. Pass either a filepath or a function.

If you pass a function, you'll be passed one options argument with the urls to the bundled JS and CSS. You should insert those into your HTML, and return a string.

```js
{
  server: {
    html: function html (paths, done){
      // it's important to include the body tags so that the livereload snippet from browsersync can be inserted
      var html = '<body>'
        + '<link rel="stylesheet" href="' + paths.css + '">'
        + '<h1>your current url ' + paths.request + '</h1>'
        + '<script src="' + paths.js + '"></script>'
        + '</body>'

      // you can return an error if something went wrong
      done(null, html)
    }
  }
}
```

If you pass a filepath, the bundled JS and CSS will automatically be inserted at the bottom of your file. However, you can place the strings `__ATOMIFY_CSS__` and `__ATOMIFY_JS__` when you want the relevant paths inserted to override this behavior.

**opts.server.spaMode** - When set to `true`, the default HTML will always be served. This is useful for single page apps that have a router.

## package.json config

In order to support atomify turtles all the way down, you can also specify your configuration in package.json. Simply recreate an `opts` object structure in JSON with atomify as the key. (Omit output properties if not a top level package.)

```json
{
  "atomify": {
  	 "server": {
  	 	"lr": true
  	 },
    "js": {
      "entry": "index.js",
      "output": "dist/bundle.js"
    },
    "css": {
      "entry": "index.css",
      "plugins": [
        ["rework-plugin-inline", "src/assets"],
        "rework-default-unit"
      ],
      "output": "example/bundle.css"
    }
  }
}
```

For detailed information on configuring Rework plugins in package.json see the [relevant section](https://github.com/atomify/atomify-css#packagejson-config) of the atomify-css README.

## CLI

Thanks to [subarg](https://github.com/substack/subarg), nearly everything you can do in code or JSON, you can do on the command line. JS options can be specified in a `--js, -j` subarg context and CSS options can be specified in a `--css, -c` subarg context. Server options can be specified in a `--server, -s` subarg context.

If you supply the `--debug, -d` or `--output, -o` args outside the `--js` and `--css` contexts they will apply to both JS and CSS bundles. When providing an `--output` argument that applies to both, omit the file extension and it will be applied correctly for you.

You can also configure aliases by appending them after a `:` in your entry field like `-j [ entry.js:/bundle.js ]`.

Get a complete listing of options by running `atomify --help`

### CLI Examples

```bash
atomify -j [ entry.js bundle.js ]
atomify -j [ -e entry.js -e other.js -o bundle.js -d -w ]
atomify -j [ entry.js -t funkify ] -c [ entry.css ] -o bundle
atomify -j [ src/entry.js:bundle.js ] -c [ styles/entry.css:bundle.css ] --server [ --open ]
```

Any top level args (js, css, server) passed on the command line will override the corresponding configuration defined in package.json. Non-conflicting top level items will be merged with package.json configuration.

## Install

```bash
npm install atomify
```
