---
sidebarDepth: 2
---

# Configuration

## Table of Contents

[[toc]]

## Configuring Electron Builder

To see available options, check out [Electron Builder Configuration Options](https://www.electron.build/configuration/configuration)

They can be placed under the `builderOptions` key in vue-cli-plugin-electron-builder's plugin options in `vue.config.js`

```javascript
// vue.config.js

module.exports = {
  pluginOptions: {
    electronBuilder: {
      builderOptions: {
        // options placed here will be merged with default configuration and passed to electron-builder
      }
    }
  }
}
```

:::tip
All CLI arguments passed to `electron:build` will be forwarded to electron-builder.
:::

## Webpack Configuration

Your regular config is extended and used for bundling the renderer process (your app). To modify your webpack config for Electron builds only, use the `chainWebpackRendererProcess` function. To modify the webpack config for the [Electron main process](https://electronjs.org/docs/tutorial/application-architecture#main-and-renderer-processes) only, use the `chainWebpackMainProcess` function under VCP Electron Builder's plugin options in `vue.config.js`. To learn more about webpack chaining, see [webpack-chain](https://github.com/mozilla-neutrino/webpack-chain). These functions work similarly to the [`chainWebpack`](https://cli.vuejs.org/config/#chainwebpack) option provided by Vue CLI.

**Note: Do NOT change the webpack output directory for the main process! See changing output directory below for more info. To change the entry point for the main process, use the `mainProcessFile` key, DO NOT modify it through chaining.**

```javascript
// vue.config.js

module.exports = {
  configureWebpack: {
    // Webpack configuration applied to web builds and the electron renderer process
  },
  pluginOptions: {
    electronBuilder: {
      chainWebpackMainProcess: (config) => {
        // Chain webpack config for electron main process only
      },
      chainWebpackRendererProcess: (config) => {
        // Chain webpack config for electron renderer process only (won't be applied to web builds)
      },
      // Use this to change the entrypoint of your app's main process
      mainProcessFile: 'src/myBackgroundFile.js',
      // Use this to change the entry point of your app's render process. default src/[main|index].[js|ts]
      rendererProcessFile: 'src/myMainRenderFile.js',
      // Provide an array of files that, when changed, will recompile the main process and restart Electron
      // Your main process file will be added by default
      mainProcessWatch: ['src/myFile1', 'src/myFile2'],
      // Provide a list of arguments that Electron will be launched with during "electron:serve",
      // which can be accessed from the main process (src/background.js).
      // Note that it is ignored when --debug flag is used with "electron:serve", as you must launch Electron yourself
      // Command line args (excluding --debug, --dashboard, and --headless) are passed to Electron as well
      mainProcessArgs: ['--arg-name', 'arg-value']
    }
  }
}
```

## Changing the Output Directory

If you don't want your files outputted into dist_electron, you can choose a custom folder in VCPEB's plugin options. You can use the `--dest` argument to change the output dir as well.

**Note: It is recommended to add the new directory to your .gitignore file.**

```javascript
// vue.config.js

module.exports = {
  pluginOptions: {
    electronBuilder: {
      outputDir: 'electron-builder-output-dir'
    }
  }
}
```

## TypeScript Options

Typescript support is automatic and requires no configuration, just add the `@vue/typescript` cli plugin. There are a few options for configuring typescript if necessary:

```javascript
// vue.config.js

module.exports = {
  pluginOptions: {
    electronBuilder: {
      // option: default // description
      disableMainProcessTypescript: false, // Manually disable typescript plugin for main process. Enable if you want to use regular js for the main process (src/background.js by default).
      mainProcessTypeChecking: false // Manually enable type checking during webpack bundling for background file.
    }
  }
}
```

:::tip Tip
If you decide to add the `@vue/typescript` plugin to your app later on, make sure to re-invoke the generator of VCP-Electron-Builder with `vue invoke electron-builder`. This will automatically insert missing type definitions to your `background.ts` file.
:::

## Changing the File Loading Protocol

By default, the `app` protocol is used to load files. This allows you to use ES6 `type="module"` scripts, created by Vue CLI's [modern mode](https://cli.vuejs.org/guide/browser-compatibility.html#modern-mode). If, for some reason, you would like to use a different protocol, set it with the `customFileProtocol` option, and change it in your `background.js` file.

```javascript
// vue.config.js
module.exports = {
  pluginOptions: {
    electronBuilder: {
      customFileProtocol: 'myCustomProtocol://./' // Make sure to add "./" to the end of the protocol
      // If you want to use the file:// protocol, add win.loadURL(`file://${__dirname}/index.html`) to your main process file
      // In place of win.loadURL('app://./index.html'), and set customFileProtocol to './'
      customFileProtocol: './'
    }
  }
}

// src/background.js
// ...
win.loadURL('myCustomProtocol://./index.html') // Change it here as well
// ...
```

## Bundling Options

By default, the app is built in [modern mode](https://cli.vuejs.org/guide/browser-compatibility.html#modern-mode). To disable this, use the `--legacy` argument in the `electron:build` command. If your app is already bundled and just needs to be built with electron-builder, pass the `--skipBundle` arg.
