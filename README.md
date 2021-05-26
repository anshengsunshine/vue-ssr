# vue-ssr
vue-ssr；引入vue-server-renderer完成服务端渲染


# SSR概念- server side Render
将一个Vue组件在服务端渲染为HTML字符串并发送到浏览器，最后再将这些静态标记“激活”为可交互应用程序的过程称为服务端渲染。
**SSR 解决的问题：**
* 首屏渲染速度慢
* SEO不友好
**SSR的问题：**
* 开发条件受限
* 构建部署要求多-nodejs渲染
* 服务端负载大
[预渲染和服务端渲染的区别？](https://www.zhihu.com/question/273930443)

# 实现 vue ssr 的具体过程
**创建工程 vue cli 3**
`vue create ssr`

**安装依赖**
`渲染器：vue-server-renderer`
`nodejs服务器：express`
`npm i vue-server-renderer express -D`

**编写服务端启动的脚本**
在根目录下创建 server文件夹，即 *src/server*
`src/server/index.js   --->nodejs服务器`

**路由**
路由支持仍使用 vue-router
**安装**
`npm i vue-router -S`
**配置**
创建`./src/router/index.js`
因为每个请求应该都是全新的、独立的应用程序实例，以便不会有交叉请求造成的状态污染。所以这里导出的是创建Router实例的工厂函数。
创建 Home.vue、About.vue，更新App.vue

**入口**
app.js

**服务端入口**
entry-server.js

**客户端入口**
entry-client.js

**webpack**
创建 vue.config.js
> `const VueSSRServerPlugin = require("vue-server-renderer/server-plugin"); const VueSSRClientPlugin = require("vue-server-renderer/client-plugin"); const nodeExternals = require("webpack-node-externals");`
> `const merge = require("lodash.merge");`
> `const TARGET_NODE = process.env.WEBPACK_TARGET === "node"; const target = TARGET_NODE ? "server" : "client";`
> 
> `module.exports = {`
> `    css: {`
> `        extract: false`
> `    },`
> `    outputDir: './dist/' + target, configureWebpack: () => ({`
> `        // 将 entry 指向应用程序的 server / client 文件`
> `        entry: ./src/entry-${target}.js,`
> `        // 对 bundle renderer 提供 source map 支持`
> `        devtool: 'source-map',`
> `        // 这允许 webpack 以 Node 适用方式处理动态导入(dynamic import)，`
> `        // 并且还会在编译 Vue 组件时告知 vue-loader 输送面向服务器代码(server-oriented code)。`
> `        target: TARGET_NODE ? "node" : "web", node: TARGET_NODE ? undefined : false, output: {`
> `            // 此处告知 server bundle 使用 Node 风格导出模块`
> `            libraryTarget: TARGET_NODE ? "commonjs2" : undefined`
> `        },`
> `        // 外置化应用程序依赖模块。可以使服务器构建速度更快，并生成较小的 bundle 文件。`
> `        externals: TARGET_NODE`
> `            ? nodeExternals({`
> `                // 不要外置化 webpack 需要处理的依赖模块。`
> `                // 可以在这里添加更多的文件类型。例如，未处理 *.vue 原始文件，`
> `                // 你还应该将修改 global（例如 polyfill）的依赖模块列入白名单`
> `                whitelist: [/\.css$/]`
> `            })`
> `            : undefined, optimization: {`
> `                splitChunks: undefined`
> `            },`
> `        // 这是将服务器的整个输出构建为单个 JSON 文件的插件。`
> `        // 服务端默认文件名为 vue-ssr-server-bundle.json`
> `        plugins: [TARGET_NODE ? new VueSSRServerPlugin() : new VueSSRClientPlugin()]`
> `    }),`
> `    chainWebpack: config => {`
> `        config.module`
> `        .rule("vue")`
> `        .use("vue-loader")`
> `        .tap(options => {`
> `            merge(options, {`
> `                optimizeSSR: false`
> `            });`
> `        });`
> `    }`
> `};`

***安装依赖***
` npm i webpack-node-externals  lodash.merge -D`
***脚本配置***
`npm i cross-env -D`
*定义创建脚本-package.json*
`"scripts":{`
`	"build:client": "vue-cli-service build",`
`	"build:server": "cross-env WEBPACK_TARGET=node vue-cli-service build --mode server",`
`	"build": "npm run build:server && npm run build:client"`
`}`

# 构建
**构建流程**-如图

**代码结构**
`src`
`	components`
`		foo.vue`
`		Bar.vue`
`		Baz.vue`
`	App.vue`
`	app.js   //通用入口`
`	entry-client.js  //客户端入口，仅运行于浏览器`
`	entry-server.js  //服务端入口，仅运行于服务器`

# **通用入口**
用于创建 vue 实例，创建 app.js

# **服务端入口**
为每次请求创建单个 vue实例 并处理首屏，创建 src/entry-server.js

安装 nodemon，用来自动调试node文件的改变。
`npm i nodemon -g`
安装vue服务器的渲染器
`npm i vue vue-server-renderer -S`
> 确保版本相同且匹配

# **服务端知识**
01-index.js
在根目录下新建 server 文件夹，与src 同级；
新建 server/index.js  server/index2.js
运行：
	`cd sever`
	`node .\index.js`
	`node .\index2.js`
可以在浏览器打开：localhost:3000