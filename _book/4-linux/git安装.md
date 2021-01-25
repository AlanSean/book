# linux 7.2 下 git安装

` yum -y install git`


安装目录在 `/usr/libexec/git-core`


但是这个方法安装的git版本比较低


# 源码安装 


1.查询最新包版本

git 包地址 `https://github.com/git/git/releases
`
想下载哪个版本只需要替换后面的版本号 

`https://codeload.github.com/git/git/tar.gz/v2.30.0`

2.下载

`wget  https://codeload.github.com/git/git/tar.gz/v2.30.0`

3.解压

`tar -zxvf git-2.30.0.tar.gz`

4.拿到解压后的源码以后我们需要编译源码了，不过在此之前需要安装编译所需要的依赖。

`yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker`
卸载安装的git
`yun -y remove git`
5.编译git源码，进入 cd  git-2.30.0 目录

make prefix=/usr/local/git all

make prefix=/usr/local/git install

6. 添加环境变量
```
vi /etc/profile 
最后一行添加
export PATH=$PATH:/usr/local/git/bin
:wq //保存并退出
刷新变量
source /etc/profile
```
