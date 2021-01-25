# http2升级
[学习地址](https://yq.aliyun.com/articles/117130?t=t1)

##### 下载最新版的OpenSSL 库

```
    wget https://www.openssl.org/source/openssl-1.1.0f.tar.gz
    tar xzf openssl-1.1.0f.tar.gz
    cd openssl-1.1.0f
    ./config --prefix=/usr/local/openssl
    make && make install
```
##### 替换openssl版本库

```
    mv /usr/bin/openssl  /usr/bin/openssl.old \
    mv /usr/include/openssl /usr/include/openssl.old \
    ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl \
    ln -s /usr/local/openssl/include/openssl /usr/include/openssl \
    #链接新库文件
    ln -s /usr/local/openssl/lib/libssl.so /usr/local/lib64/libssl.so \
    ln -s /usr/local/openssl/lib/libcrypto.so /usr/local/lib64/libcrypto.so \
    #检查更新后的openssl依赖库是否是1.1.0f
    strings /usr/local/lib64/libssl.so | grep OpenSSL
    #显示结果表明已升级到最新版本链接库
    OpenSSL 1.1.0f  25 May 2017
    #配置openssl库文件的搜索路径
    echo '/usr/local/openssl/lib' >> /etc/ld.so.conf
    #使修改后的搜索路径生效
    ldconfig -v
    #查看openssl版本，结果显示升级成功
    openssl version
    OpenSSL 1.1.0f  25 May 2017
```
##### linux把nginx添加到全局环境变量
```
    ln -s /usr/local/nginx/sbin/nginx /usr/local/bin/
    /usr/local/bin/就是环境变量目录
```
##### nginx 编译
```
    wget http://nginx.org/download/nginx1.13.7.tar.gz
    tar zxvf nginx1.13.7.tar.gz
    cd nginx1.13.7
    ./configure --prefix=/usr/local/nginx \
    --conf-path=/usr/local/nginx/conf/nginx.conf \
    --sbin-path=/usr/local/nginx/sbin/nginx \
    #注意改成自己的pid文件位置
    --pid-path=/usr/local/nginx/logs/nginx.pid \  
    #注意改成自己的log文件位置
    --error-log-path=/usr/local/nginx/logs/error.log \    
    --http-log-path=/usr/local/nginx/logs/access.log \  
    --user=nginx \
    --group=nginx \
    --with-http_ssl_module \
    --with-http_realip_module \
    --with-http_flv_module \
    --with-http_mp4_module \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_secure_link_module \
    --with-http_v2_module \
    --with-http_stub_status_module \
    --with-http_sub_module \
    #注意改成自己的openssl位置
    --with-openssl=/usr/local/openssl-1.1.0f

    make
    mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.old
    cp objs/nginx /usr/local/nginx/sbin/nginx
    make upgrade
    nginx -V

    conf 文件  
    listen 443 ssl http2;
```
