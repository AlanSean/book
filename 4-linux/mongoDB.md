#mongoDB安装


```
    下载
    wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.5.tgz
    解压
    tar -zxvf mongodb-linux-x86_64-4.0.5.tgz
    改名
    mv mongodb-linux-x86_64-4.0.5 你安装的目录/mongodb
    环境变量
    export PATH=/usr/local/mongodb/bin:$PATH
    ln -s /你安装的目录/mongodb/bin/mongod /usr/local/bin/mongod
    mkdir  data
    cd data
    mkdir  db
    mkdir  logs
    vi  logs/mongodb.log
    按esc 输入:wq 回车
    vi  mongodb.conf
        #设置端口
        port  = 27017
        #数据目录
        dbpath = /你安装的目录/mongodb/data
        #日志目录
        logpath = /你安装的目录/mongodb/data/logs/mongodb.log
        #后台运行
        fork = true
        #日志输出方式
        logappend = true
        #修改ip
        bind_ip = 0.0.0.0
        #开启验证
        auth = true
    按esc 输入:wq 回车
    进入mongo shell
    ./bin/mongo
        use admin
        db.createUser({user:'admin',pwd:"密码你自己设置嘛",roles:["root"]})
    成功之后Ctrl+c 退出shell
    mongod -f /你安装的目录/mongodb/mongodb.conf

    然后就可以在Robo里面访问拉，记住你得密码！！！

```
