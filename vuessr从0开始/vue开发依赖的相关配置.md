# vue开发依赖的相关配置
[Vue SSR 指南]( https://ssr.vuejs.org/zh/)

今天先做客户端方面的配置，明天再做服务端的部分。
那么马上开始吧~
#### 修改部分代码
脚手架生成的代码肯定是不适合我们所用的 所以要修改一部分代码

```
//App.vue
<template>
    <div id="app">
        <router-view></router-view>
    </div>
</template>

<script>
export default {
    name: 'app'
}
</script>

<style>
    html,body,#app,#app>*{
        width: 100%;
        height: 100%;
    }
    body{
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        font-size: 16px;
        margin: 0;
        overflow-x: hidden;
    }

    img{
        width: 200px;
    }
</style>

```

修改main.js
[为什么要这么做？为什么要避免状态单例](https://ssr.vuejs.org/zh/guide/structure.html#%E9%81%BF%E5%85%8D%E7%8A%B6%E6%80%81%E5%8D%95%E4%BE%8B)
main.js 是我们应用程序的「通用 entry」。
在纯客户端应用程序中，我们将在此文件中创建根 Vue 实例，并直接挂载到 DOM。
但是，对于服务器端渲染(SSR)，责任转移到纯客户端 entry 文件。
main.js 简单地使用 export 导出一个 createApp 函数：

```
import Vue from 'vue';
Vue.config.productionTip = false;
import App from './App.vue';
//router store 实例
//这么做是避免状态单例
export function createApp() {

    const app = new Vue({
        //挂载router，store
        render: h => h(App)
    })
    //暴露app实例
    return { app };
}


```
#### 添加Vue.config.js配置

webpack的入口文件有两个，一个是客户端使用，一个是服务端使用。
[为何这么做？](https://ssr.vuejs.org/zh/guide/build-config.html)
今天只做客户端部分。

```
    src/vue.config.js
    module.exports = {
        css: {
            extract: false//关闭提取css,不关闭 node渲染会报错
        },
        configureWebpack: () => ({
            entry: './src/entry/client'
        })
    }
    app.$mount('#app')

```

### 根目录创建 entry 文件夹，以及webpack入口代码

```
    mdkir  entry
    cd  entry
    创建 入口文件
        client.js 作为客户端入口文件。
        server,js 作为服务端端入口文件。 //先创建不做任何配置
    entry/client.js

        import { createApp } from '../main.js';

        const { app } = createApp();

        app.$mount('#app');

```

####  路由和代码分割

[官方说明的已经很清楚了，我就不做过多介绍了，下面直接展示代码](https://ssr.vuejs.org/zh/guide/routing.html)

添加新路由，这里将存放pages的相关路由

```
src/pages/router/index.js

/**
 *
 * @method componentPath 路由模块入口
 * @param  {string} name 要引入的文件地址
 * @return {Object}
 */
function componentPath (name = 'home'){
    return {
        component:() => import(`../${name}/index.vue`)
    }
}

export default [
    {
        path: '/home',
        ...componentPath(),
        children: [
            {
                path: "vue",
                name: "vue",
                ...componentPath('home/vue')
            },
            {
                path: "vuex",
                name: "vuex",
                ...componentPath('home/vuex')
            },
            {
                path: "vueCli3",
                name: "vueCli3",
                ...componentPath('home/vueCli3')
            },
            {
                path: "vueSSR",
                name: "vueSSR",
                ...componentPath('home/vueSSR')
            }
        ]

    }
]

```
src/router.config.js作为路由的总配置 易于管理
```

//路由总配置
    import Vue from 'vue';
    import VueRouter from 'vue-router';

    Vue.use(VueRouter);
    //为什么采用这种做法。
    //如果以后有了别的大模块可以单独开个文件夹与pages平级
    //再这里导入即可。这样易于管理

    // pages
    import pages from './pages/router';

    export function createRouter() {
        return new VueRouter({
            mode: 'history',
            routes: [
                {
                    path: "*",
                    redirect: '/home/vue'
                },
                ...pages
            ]
        })
    }
```

更新main.js

```
import Vue from 'vue';
Vue.config.productionTip = false;
import App from './App.vue';
+ import { createRouter } from './router.config.js'
//router store 实例
//这么做是避免状态单例
export function createApp() {
+   const router = createRouter()
    const app = new Vue({
+       router,
        render: h => h(App)
    })
    //暴露app,router实例
    return { app, router };
}
```
更新 client.js
由于使用的路由懒加载，所以必须要等路由提前解析完异步组件，才能正确地调用组件中可能存在的路由钩子。
```
// client.js
import { createApp } from '../main.js';

const { app, router } = createApp();

router.onReady( () => {
    app.$mount('#app');
})


```

vuex 明天再做

[项目github地址](https://github.com/AlanSean/vuessr )

[项目公网地址](https://adm.hqboke.cn/home/vueCli3)
