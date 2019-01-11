# vue-cli3+node中间件遇到的坑 - 主cli3的坑

```
    设置别名 只能在  chainWebpack 里面设置
    config.resolve.alias
        .set('assets', resolve('src/assets'))
        .set('components', resolve('src/components'))
        .set('methods', resolve('src/methods'));
    在configureWebpack 里面设置别名 用node运行webpack函数时，不会合并。
```
