## 安装

cd ~/lnmp1.7

./install db

## 开启远程访问

1. 登录
    
    mysql -u root -p
2. 查看用户表

    use mysql
    select Host,User from user;

    grant all privileges on 库名.表名 to '用户名'@'IP地址' identified by '密码' with grant option;

    flush privileges;

    库名:要远程访问的数据库名称,所有的数据库使用“*” 
    表名:要远程访问的数据库下的表的名称，所有的表使用“*” 
    用户名:要赋给远程访问权限的用户名称 
    IP地址:可以远程访问的电脑的IP地址，所有的地址使用“%” 
    密码:要赋给远程访问权限的用户对应使用的密码

3. 增加新用户。（注意：mysql环境中的命令后面都带一个分号作为命令结束符）

    grant select on 数据库.* to 用户名@登录主机 identified by "密码"
    如增加一个用户test密码为123，让他可以在任何主机上登录， 并对所有数据库有查询、插入、修改、删除的权限。首先用以root用户连入mysql，然后键入以下命令：
    grant select,insert,update,delete on . to " Identified by "123";

# 查询时区
 show variables like '%time_zone%';  

    



