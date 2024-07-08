# React 入门

## 语法

参考 [React 文档](https://zh-hans.react.dev/learn)。半个小时学习80% 的 React 概念。

### [React 技术栈系列教程](https://www.ruanyifeng.com/blog/2016/09/react-technology-stack.html)

- [react-router](https://github.com/remix-run/react-router?tab=readme-ov-file)
- [redux](https://github.com/reduxjs/redux)
- [axios](https://github.com/axios/axios)
- [webpack](https://github.com/webpack/webpack)
- ...
## 脚手架

工程化需要脚手架。

[Create React App](https://github.com/facebook/create-react-app) （CRA）Create React apps with no build configuration.

```sh
npx create-react-app my-app --template typescript
cd my-app
npm start
```
It will create a directory called my-app inside the current folder.
```
my-app
├── README.md
├── node_modules
├── package.json
├── .gitignore
├── public
│   ├── favicon.ico
│   ├── index.html
│   └── manifest.json
└── src
    ├── App.css
    ├── App.js
    ├── App.test.js
    ├── index.css
    ├── index.js
    ├── logo.svg
    └── serviceWorker.js
    └── setupTests.js
```

除此之外，[Create React App](https://github.com/facebook/create-react-app)还提供了很多创建React App 工程化的配置建议。如官网下述文字所说，学习React语法去 [reactjs.org](https://zh-hans.react.dev/learn)，配置启动React工程去[Create React App](https://github.com/facebook/create-react-app)。

> This website is only about Create React App.The documentation for React itself is located on a separate website: [reactjs.org](https://zh-hans.react.dev/learn).

比如怎么[Adding a Router](https://create-react-app.dev/docs/adding-a-router), 怎么[Proxying API Requests in Development](https://create-react-app.dev/docs/proxying-api-requests-in-development),[Fetching Data with AJAX Requests](https://create-react-app.dev/docs/fetching-data-with-ajax-requests)，怎么设置[Adding TypeScript 支持](https://create-react-app.dev/docs/adding-typescript)等等。

### [npm run eject](https://create-react-app.dev/docs/available-scripts/#npm-run-eject)

Create React App 实际上也是帮我们封装配置了诸如打包工具（默认是Webpack）等，而不是说如上所示目录，没有使用打包工具，只是被`react-scripts`封装起来了，所有相关的配置都在这里面，要暴露出来，高度自定义就需要运行`npm run eject`命令。但一般没有必要运行。

### `react-scripts`分析

`Create React App` 实际上感觉和`Angular Cli`是类似的产品。

`react-scripts`是一个 npm 包， 安装在`$ProjectPath/node_modules/react-scripts`下。

#### 目录结构

```
.
├── LICENSE
├── README.md
├── bin
│   └── react-scripts.js
├── config
│   ├── env.js
│   ├── getHttpsConfig.js
│   ├── jest
│   │   ├── babelTransform.js
│   │   ├── cssTransform.js
│   │   └── fileTransform.js
│   ├── modules.js
│   ├── paths.js
│   ├── webpack
│   │   └── persistentCache
│   │       └── createEnvironmentHash.js
│   ├── webpack.config.js
│   └── webpackDevServer.config.js
├── lib
│   └── react-app.d.ts
├── package.json
├── scripts
│   ├── build.js
│   ├── eject.js
│   ├── init.js
│   ├── start.js
│   ├── test.js
│   └── utils
│       ├── createJestConfig.js
│       └── verifyTypeScriptSetup.js
├── template
│   └── README.md
└── template-typescript
    └── README.md
```
#### 分析
`npm run start`等命令实际上运行的是：
```json
"scripts": {
	"start": "react-scripts start",
	"build": "react-scripts build",
	"test": "react-scripts test",
	"eject": "react-scripts eject"
},
```

查看`react-scripts/scripts/start.js`中的内容可以得知：
```js
const fs = require('fs');
const chalk = require('chalk');
const webpack = require('webpack');
const WebpackDevServer = require('webpack-dev-server');
const clearConsole = require('react-dev-utils/clearConsole');
const checkRequiredFiles = require('react-dev-utils/checkRequiredFiles');
const {
  choosePort,
  createCompiler,
  prepareProxy,
  prepareUrls,
} = require('react-dev-utils/WebpackDevServerUtils');
const openBrowser = require('react-dev-utils/openBrowser');
const paths = require('../config/paths');
const config = require('../config/webpack.config.dev');
const createDevServerConfig = require('../config/webpackDevServer.config');

const useYarn = fs.existsSync(paths.yarnLockFile);
const isInteractive = process.stdout.isTTY;

```
`webpack`实际上是在这里引入的，配置文件也是在这里读取的。

#### `npm run start`怎么读取到的项目目录

npm run start 实际运行的是，那它怎么读取到的我的项目目录从而读取对应的文件的呢？查阅`config/paths.js`后发现，是通过`node.js`的 `filesystem` 接口。
```js
const appDirectory = fs.realpathSync(process.cwd());
```
进而：
```js
const resolveApp = relativePath => path.resolve(appDirectory, relativePath);

module.exports = {
  dotenv: resolveApp('.env'),
  appPath: resolveApp('.'),
  appBuild: resolveApp(buildPath),
  appPublic: resolveApp('public'),
  appHtml: resolveApp('public/index.html'),
  appIndexJs: resolveModule(resolveApp, 'src/index'),
  appPackageJson: resolveApp('package.json'),
  appSrc: resolveApp('src'),
  appTsConfig: resolveApp('tsconfig.json'),
  appJsConfig: resolveApp('jsconfig.json'),
  yarnLockFile: resolveApp('yarn.lock'),
  testsSetup: resolveModule(resolveApp, 'src/setupTests'),
  proxySetup: resolveApp('src/setupProxy.js'),
  appNodeModules: resolveApp('node_modules'),
  appWebpackCache: resolveApp('node_modules/.cache'),
  appTsBuildInfoFile: resolveApp('node_modules/.cache/tsconfig.tsbuildinfo'),
  swSrc: resolveModule(resolveApp, 'src/service-worker'),
  publicUrlOrPath,
};
```
拿到对应的文件。也可以看到`react-scripts`会读取哪些文件（没有webpack.config.json）。
### 如何更改Webpack配置

没办法直接修改通过Create React Apps创建项目的Webpack(除了eject)，需要类似[react-app-rewired](https://github.com/timarney/react-app-rewired/blob/master/README_zh.md)的工具：此工具可以在不 'eject' 也不创建额外 react-scripts 的情况下修改 create-react-app 内置的 webpack 配置，然后你将拥有 create-react-app 的一切特性，且可以根据你的需要去配置 webpack 的 plugins, loaders 等。
## [Next.js](https://github.com/vercel/next.js)

框架的框架。

Used by some of the world's largest companies, Next.js enables you to create full-stack web applications by extending the latest React features, and integrating powerful Rust-based JavaScript tooling for the fastest builds.

# References

1. [create-react-app 源码解析之react-scripts](https://juejin.cn/post/6844903605783232526)
