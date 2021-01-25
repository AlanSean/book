## node安装
    ```
        wget https://npm.taobao.org/mirrors/node/v8.9.3/node-v8.9.3-linux-x64.tar.xz
        yum search xz   检查是否有xz命令
        yum install xz
        xz -d  node-v8.9.3-linux-x64.tar.xz
        tar -xf node-v8.9.3-linux-x64.tar
        ln -s 安装的位置/node/bin/npm /usr/local/bin/npm
        ln -s 安装的位置/node/bin/node /usr/local/bin/node
    ```
