# 前言

上一次做了[路由的相关配置](https://segmentfault.com/a/1190000018026161)，原本计划今天要做vuex部分，但是想了想，发现vuex单独的客户端部分穿插解释起来很麻烦，所以今天改做服务端部分。
服务端部分做完，再去做vuex的部分，这样就会很清晰。

vue ssr是分两个端，一个是客户端，一个是服务端。
所以要做两个cli3的配置。
那么下面就直接开始做吧。

### 修改package.json的命令

```
//package.json :client代表客户端 :server代表服务端
//使用VUE_NODE来作为运行环境是node的标识
//cli3内置  命令 --no-clean 不会清除dist文件
    "scripts": {
        "serve:client": " vue-cli-service serve",
        "build":"npm run build:server -- --silent && npm run build:client -- --no-clean --silent",
        "build:client": "vue-cli-service build",
        "build:server": "cross-env VUE_NODE=node vue-cli-service build",
        "start:server": "cross-env NODE_ENV=production nodemon nodeScript/index"
    }

```

### 修改vue.config.js配置
添加完相关脚本命令之后，我们开始改造cli3配置。
首先要require('vue-server-renderer')
然后再根据VUE_NODE环境变量来决定编译的走向以及生成不同的环境清单

先做cli3服务端的入口文件

```
// src/entry/server.js
import {
    createApp
} from  '../main.js'

export default context => {

    return new Promise((resolve, reject) => {
        const {
            app,
            router
        } = createApp(context.data)
        //根据node传过来的路由 来调用router路由的指向
        router.push(context.url)

        router.onReady(() => {
            //获取当前路由匹配的组件数组。
            const matchedComponents = router.getMatchedComponents()
            //长度为0就是没找到该路由所匹配的组件
            //可以路由设置重定向或者传回node  node来操作也可以
            if (!matchedComponents.length) {

                return reject({
                    code: 404
                })
            }

            resolve(app)

        }, reject)
    })
}


```

这里是cli3的配置

```

//vue.config.js

const ServerPlugin = require('vue-server-renderer/server-plugin'),//生成服务端清单
      ClientPlugin = require('vue-server-renderer/client-plugin'),//生成客户端清单
      nodeExternals = require('webpack-node-externals'),//忽略node_modules文件夹中的所有模块
      VUE_NODE = process.env.VUE_NODE === 'node',
      entry = VUE_NODE ? 'server' : 'client';//根据环境变量来指向入口

module.exports = {
    css: {
        extract: false//关闭提取css,不关闭 node渲染会报错
    },
    configureWebpack: () => ({
        entry: `./src/entry/${entry}`,
        output: {
            filename: 'js/[name].js',
            chunkFilename: 'js/[name].js',
            libraryTarget: VUE_NODE ? 'commonjs2' : undefined
        },
        target: VUE_NODE  ? 'node' : 'web',
        externals: VUE_NODE ? nodeExternals({
            //设置白名单
            whitelist: /\.css$/
        }) : undefined,
        plugins: [//根据环境来生成不同的清单。
            VUE_NODE ? new ServerPlugin() : new ClientPlugin()
        ]
    }),
    chainWebpack: config => {
        config.resolve
            .alias
                .set('vue$', 'vue/dist/vue.esm.js')
        config.module
            .rule('vue')
                .use('vue-loader')
                    .tap(options => {
                        options.optimizeSSR = false;
                        return options;
                    });
        config.module
            .rule('images')
                .use('url-loader')
                    .tap(options => {
                        options = {
                            limit: 1024,
                            fallback:'file-loader?name=img/[path][name].[ext]'
                        }
                        return options;
                    });
    }
}

```



### node相关配置

用于node渲染 必然要拦截get请求的。然后根据get请求地址来进行要渲染的页面。

官方提供了[vue-server-renderer插件](https://ssr.vuejs.org/zh/api/)
大概的方式就是 node拦截所有的get请求，然后讲获取到的路由地址，传给前台，然后使用router实例进行push


再往下面看之前 先看一下官方文档

创建BundleRenderer
[createBundleRenderer](https://ssr.vuejs.org/zh/api/#createbundlerenderer)
将 Vue 实例渲染为字符串。
[renderToString](https://ssr.vuejs.org/zh/api/#renderer-rendertostring)
渲染应用程序的模板
[template](https://ssr.vuejs.org/zh/api/#template)
生成所需要的客户端或服务端清单
[clientManifest](https://ssr.vuejs.org/zh/api/#clientmanifest)



先创建 服务端所需要的模板
```
//public/index.nodeTempalte.html
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width,initial-scale=1.0">
        <link rel="icon" href="/favicon.ico">
        <meta charset="utf-8">
        <title>vuessr</title>
    </head>
    <body>
        <!--vue-ssr-outlet-->
    </body>
</html>

```

node部分

先创建三个文件
index.js //入口
proxy.js //代理
server.js //主要配置


```
//server.js
const fs = require('fs');
const { resolve } = require('path');
const express = require('express');
const app = express();
const proxy = require('./proxy');
const { createBundleRenderer } = require('vue-server-renderer')

//模板地址
const templatePath = resolve(__dirname, '../public/index.nodeTempalte.html')
//客户端渲染清单
const clientManifest = require('../dist/vue-ssr-client-manifest.json')
//服务端渲染清单
const bundle = require('../dist/vue-ssr-server-bundle.json')
//读取模板
const template = fs.readFileSync(templatePath, 'utf-8')
const renderer = createBundleRenderer(bundle,{
    template,
    clientManifest,
    runInNewContext: false
})


//代理相关
proxy(app);
//请求静态资源相关配置
app.use('/js', express.static(resolve(__dirname, '../dist/js')))
app.use('/css', express.static(resolve(__dirname, '../dist/css')))
app.use('/font', express.static(resolve(__dirname, '../dist/font')))
app.use('/img', express.static(resolve(__dirname, '../dist/img')))
app.use('*.ico', express.static(resolve(__dirname, '../dist')))


//路由请求
app.get('*', (req, res) => {
    res.setHeader("Content-Type", "text/html")
    //传入路由 src/entry/server.js会接收到  使用vueRouter实例进行push
    const context = {
        url: req.url
    }

    renderer.renderToString(context, (err, html) => {
        if (err) {
            if (err.url) {
                res.redirect(err.url)
            } else {
                res.status(500).end('500 | 服务器错误');
                console.error(`${req.url}: 渲染错误 `);
                console.error(err.stack)
            }
        }
        res.status(context.HTTPStatus || 200)
        res.send(html)
    })
})
module.exports = app;



//proxy.js

const proxy = require('http-proxy-middleware');

function proxyConfig(obj){
    return {
        target:'localhost:8081',
        changeOrigin:true,
        ...obj
    }
}
module.exports = (app) => {
    //代理开发环境
    if (process.env.NODE_ENV !== 'production') {
        app.use('/js/main*', proxy(proxyConfig()));
        app.use('/*hot-update*',proxy(proxyConfig()));
        app.use('/sockjs-node',proxy(proxyConfig({ws:true})));
    }
}


//index.js
const app = require('./server')

app.listen(8080, () => {
  console.log('\033[42;37m DONE \033[40;33m localhost:8080 服务已启动\033[0m')
})




```

做完这一步之后，就可以预览基本的服务渲染了。
后面就只差开发环境的配置，以及到node数据的传递(vuex)

```
    npm run build
    npm run start:server
    打开localhost:8080
    F12 - Network - Doc 就可以看到内容
```


### 最终目录结构
```
|-- vuessr
    |-- .gitignore
    |-- babel.config.js
    |-- package-lock.json
    |-- package.json
    |-- README.md
    |-- vue.config.js
    |-- nodeScript //node 渲染配置
    |   |-- index.js
    |   |-- proxy.js
    |   |-- server.js
    |-- public//模板文件
    |   |-- favicon.ico
    |   |-- index.html
    |   |-- index.nodeTempalte.html
    |-- src
        |-- App.vue
        |-- main.js
        |-- router.config.js/。路由集合
        |-- store.config.js//vuex 集合
        |-- assets//全局静态资源源码
        |   |-- 备注.txt
        |   |-- img
        |       |-- logo.png
        |-- components//全局组件
        |   |-- Head
        |       |-- index.js
        |       |-- index.scss
        |       |-- index.vue
        |       |-- img
        |           |-- logo.png
        |-- entry//cli3入口
        |   |-- client.js
        |   |-- server.js
        |   |-- 备注.txt
        |-- methods//公共方法
        |   |-- 备注.txt
        |   |-- mixin
        |       |-- index.js
        |-- pages//源码目录
        |   |-- home
        |   |   |-- index.js
        |   |   |-- index.scss
        |   |   |-- index.vue
        |   |   |-- img
        |   |   |   |-- flow.png
        |   |   |   |-- head_portrait.jpg
        |   |   |   |-- logo.png
        |   |   |   |-- vuessr.png
        |   |   |-- vue
        |   |   |   |-- index.js
        |   |   |   |-- index.scss
        |   |   |   |-- index.vue
        |   |   |-- vueCli3
        |   |   |   |-- index.js
        |   |   |   |-- index.scss
        |   |   |   |-- index.vue
        |   |   |-- vueSSR
        |   |   |   |-- index.js
        |   |   |   |-- index.scss
        |   |   |   |-- index.vue
        |   |   |-- vuex
        |   |       |-- index.js
        |   |       |-- index.scss
        |   |       |-- index.vue
        |   |-- router//路由配置
        |   |   |-- index.js
        |   |-- store//vuex配置
        |       |-- all.js
        |       |-- gather.js
        |       |-- index.js
        |-- static//cdn资源
            |-- 备注.txt

```
**[github欢迎watch](https://github.com/AlanSean/vuessr)**
