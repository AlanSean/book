## 简览
* [警告总览](#警告总览)
* [修改md内容时服务报错并中断](#修改md内容时服务报错并中断)
* [gitbook官网打不开](#gitbook官网打不开)





### 修改md内容时服务报错并中断

    解决方案:  
        先启动 gitbook serve服务，然后删除本地 "book"文件夹（或者你指定的源码生成目录）
        具体为啥这样会好，我也不清楚。不过这样好用就是了

### 警告总览
```
    warn: "options" property is deprecated, use config.get(key) instead

    机翻：
        警告:不推荐使用“options”属性，而是使用config.get(key)
    原因:
        猜测为某个插件版本过老，毕竟gitbook更新还是蛮快的
```

### gitbook官网打不开

```
    把ssr代理模式修改为全局代理即可。
```
