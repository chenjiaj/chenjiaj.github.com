
---
title: 解决docker 创建mysql，navcat无法连接问题
date:  2019. 01.18 10:15
categories: 'mysql'
tags: [mysql, docker]
---

#### 一、创建容器
    docker run -p 3306:3306 --name mymysql -v $PWD/data:/var/lib/mysql -d -e MYSQL_ROOT_PASSWORD=123456 mysql

#### 二、navicat连接
配置好后无法连接

1,容器中登录mysql,查看mysql的版本

    mysql> status;
    mysql  Ver 8.0.11 for Linux on x86_64 (MySQL Community Server - GPL)

2,进行授权远程连接(注意mysql 8.0跟之前的授权方式不同)

    // 授权
    GRANT ALL ON *.* TO 'root'@'%';

    //刷新权限
    flush privileges
　
此时,还不能远程访问,因为Navicat只支持旧版本的加密,需要更改mysql的加密规则

 
    //更改加密规则
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;
    //更新root用户密码
    ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
    //刷新权限
    flush privileges;

OK，设置完成，再次使用 Navicat 连接数据库
