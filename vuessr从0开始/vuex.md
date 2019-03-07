# vuex的相关配置
上一章做了[node以及vue-cli3的配置](https://segmentfault.com/a/1190000018043697),今天就来做vuex的部分。
先打开[官方文档-数据预取和状态](https://ssr.vuejs.org/zh/guide/data.html)

看完之后，发现大致的逻辑就是
利用mixin，拦截页面渲染完成之前，查看当前实例是否含有'asyncData'函数(由你创建以及任意名称)，
如果含有就进行调用，并且传入你需要的对象比如(store,route)


例子
```
// pages/vuex/index.js
export default {
    name: "vuex",
    asyncData({
        store,
        route
    }) {
        // 触发 action 后，会返回 Promise
        return store.dispatch('fetchItem', route.path)
    },
    computed: {
        // 从 store 的 state 对象中的获取 item。
        item() {
            return this.$store.state.items[this.$route.path]
        }
    }
}


//mixin

import Vue from 'vue';
Vue.mixin({
    beforeMount() {
        const {
            asyncData
        } = this.$options
        if (asyncData) {
            this.dataPromise = asyncData({
                store: this.$store,
                route: this.$route
            })
        }
    }
})
```


上面只是个大概的示例，下面开始正式来做吧。
    先创建一些文件
        src/store.config.js  跟router.config.js 一样，在服务端运行避免状态单例
        src/pages/store/all.js 全局公共模块
```
//  src/pages/store/all.js
    const all = {
        //开启命名空间
        namespaced: true,
        //ssr注意事项 state 必须为函数
        state: () => ({
            count:0
        }),
        mutations: {
            inc: state => state.count++
        },
        actions: {
            inc: ({ commit }) => commit('inc')
        }
    }
    export default all;

```
#### vuex 模块
all 单独一个全局模块
如果home有自己的数据，那么就在home下 惰性注册模块
但是记得页面销毁时，也一定要销毁模块！！！
因为当多次访问路由时，可以避免在客户端重复注册模块。
如果想只有个别几个路由共用一个模块，
可以在all里面进行模块嵌套，或者将这个几个页面归纳到一个父级路由下 ，在父级实例进行模块惰性注册。


```
// home/index.js
import all from '../store/all.js';
export default {
    name: 'home',
    computed:{
        count(){
            return this.$store.state.all.count
        }
    },
    asyncData({ store}){
        store.registerModule('all',all);
        return store.dispatch('all/inc')
    },
    data() {
        return {
            activeIndex2: '1',
            show:false,
            nav:[
                {
                    path:'/home/vue',
                    name:'vue'
                },
                {
                    path:'/home/vuex',
                    name:'vue-vuex'
                },
                {
                    path:'/home/vueCli3',
                    name:'vue-cli3'
                },
                {
                    path:'/home/vueSSR',
                    name:'vue ssr'
                }
            ]
        };
    },
    watch:{
        $route:function(){
            if(this.show){
                this.show = false;
            }
            //切换路由时，进行自增
            this.$store.dispatch('all/inc');
        }
    },
    mounted() {
        //做额外请求时,在mounted下进行

    },
    methods: {
        user_info(){
            this.http.post('/cms/i1/user_info').then(res=> {
                console.log(res.data);
            }).catch( error => {
                console.log(error)
            })
        }
    },
    destroyed(){
        this.$store.unregisterModule('all')
    }
}


````
#### 数据预取
```
// store.config.js

    //store总配置
    import Vue from 'vue';
    import Vuex from 'vuex';

    Vue.use(Vuex);


    //预请求数据
    function fetchApi(id){
        //该函数是运行在node环境 所以需要加上域名
        return axios.post('http://localhost:8080/cms/i1/user_info');
    }
    //返回 Vuex实例 避免在服务端运行时的单利状态
    export function createStore() {
        return new Vuex.Store({
            state:{
                items:{}
            },
            actions: {
                fetchItem ({commit}, id) {
                    return fetchApi(id).then(item => {
                        commit('setItem',{id, item})
                    })
                }
            },
            mutations: {
                setItem(state, {id, item}){
                    Vue.set(state.items, id, item.data)
                }
            }
        })
    }


```
#### mixin相关

 在src下新建个methods 文件夹，这里存放写vue的全局代码以及配置
  获取当前实例
```
// src/methods/index.js

import './mixin';
import Vue from 'vue';
import axios from 'axios';
Vue.prototype.http = axios;

// src/methods/mixin/index.js
    import Vue from 'vue';
    Vue.mixin({
        beforeMount() {
            const {
                asyncData
            } = this.$options;//这里 自己打印下就知道了。就不过多解释了
            //当前实例是否有该函数
            if (asyncData) {
               // 有就执行，并传入相应的参数。
                asyncData({
                    store: this.$store,
                    route: this.$route
                })
            }
        }
    })


````
main.js 新增代码

```
    import Vue from 'vue';
    Vue.config.productionTip = false;
    import VueRouter from 'vue-router';

    import App from './App.vue';
+   import './methods';
    //同步路由状态
+   import { sync } from 'vuex-router-sync';


    import {
        createRouter
    } from './router.config.js';

+   import {
        createStore
    } from './store.config.js';

    export function createApp() {

        const router = createRouter()
        const store = createStore()
        //同步路由状态(route state)到 store
        sync(store, router)
        const app = new Vue({
            router,
+           store,
            render: h => h(App)
        })
        return {
            app,
            router,
+           store
        };
    }

```


下面更新，开发环境部署相关
