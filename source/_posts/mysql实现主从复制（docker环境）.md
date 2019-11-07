
---
title: mysql实现主从复制（docker环境）
date:   2019.01.18 14:15
categories: 'mysql'
tags: [mysql, docker]
---

原文链接：https://blog.csdn.net/fyihdg/article/details/78951357 

本文是学习了以上链接文章之后，自己总结了一遍，便于以后查看。

### 一、拉取镜像、创建2个容器
##### 1.拉取镜像centos7，作为安装运行mysql的环境。

    docker pull registry.cn-hangzhou.aliyuncs.com/moensun/centos7

##### 2.查询镜像ID

    docker images

![查询镜像ID](/images/mysql实现主从复制（docker环境）-img/1.png)


##### 3.创建两个空容器
如果是centos7需要加上--privileged 和 /usr/sbin/init，解决systemctl报错不能使用的问题

    docker run --privileged -itd --name 容器名称 镜像ID /usr/sbin/init

非centos7，使用命令如下

    docker run -tid 镜像ID /bin/bash

![a.png](/images/mysql实现主从复制（docker环境）-img/2.png)


### 二、在容器中安装mysql
两个容器分别执行以下操作

##### 1.进入容器

    docker ps //查询容器信息
    docker exec -it 容器ID或容器名称 /bin/bash  //进入容器命令
    
##### 2.安装mysql
依此执行以下命令

    yum -y install wget
    wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
    rpm -ivh mysql-community-release-el7-5.noarch.rpm
    yum install mysql-community-server

如遇到以下提示，输入y
![b.png](/images/mysql实现主从复制（docker环境）-img/3.png)

### 三、搭建主从复制
![架构图](/images/mysql实现主从复制（docker环境）-img/4.png)

##### 1.启动mysql、登录设置mysql
这一个步骤，两个容器都需要执行

启动mysql、登录

    systemctl start mysql
    mysql -uroot

设置mysql密码，不想设置可以跳过这一步

    mysql> set password = password(‘你的密码’)

远程登陆授权

    mysql> grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;
    mysql> flush privileges;

##### 2. 修改配置，实现主从复制

###### 1）修改配置文件
两个容器都需要修改，但**`server-id两个容器不能设置相同数字`**

    vi  /etc/my.cnf

添加以下配置

    server-id=1
    port=3306
    log-bin=mysql-bin

![/etc/my.cnf文件配置](/images/mysql实现主从复制（docker环境）-img/5.png)

重启mysql

    systemctl restart mysql


###### 2)查询两个容器ip

![g.png](/images/mysql实现主从复制（docker环境）-img/6.png)


###### 3) 主机配置
主从复制过程（在主机上操作）：

本例子中以容器名为mysql-master作为主机

        docker exec -it mysql-master /bin/bash //进入主机容器
        mysql //登录mysql

1.	创建同步复制的用户

        mysql> create user 'hdg'@'172.17.0.%' identified by 'root';

2.	给同步复制用户赋权

        mysql> grant replication slave on *.* to 'hdg'@'172.17.0.%' identified by 'root';

3.	开启binlog

        mysql> flush privileges;

    配置时候注意几个坑：
    Replication-do-db的坑，如果多个库则使用多行Replication-do-db进行配置
    Replication-ignore-db的坑，如果忽略多个库则使用多行Replication-ignore-db进行配置
4.	重启mysql

        systemctl restart mysql

![e.png](/images/mysql实现主从复制（docker环境）-img/7.png)

登陆主机mysql
这个时候就产生binlog了，离成功不远了！
![f.png](/images/mysql实现主从复制（docker环境）-img/8.png)

###### 4)从机配置

本例子中以容器名为mysql-slave作为主机

        docker exec -it mysql-slave /bin/bash //进入主机容器
        mysql //登录mysql

从机操作
主从复制的最关键语句:

    Stop slave；
    Change master to
         Master_host='172.17.0.13',	//主机的IP地址
         Master_user='hdg',
         Master_password='root',
         Master_log_file='mysql-bin.000001',
         Master_log_pos=120;
    Start slave;

![查询主机状态](/images/mysql实现主从复制（docker环境）-img/9.png)

在主机中查看主机状态：

    show master status;

![i.png](/images/mysql实现主从复制（docker环境）-img/10.png)



在从机中查看从机状态

    show slave status\G;

![查看从机状态](/images/mysql实现主从复制（docker环境）-img/11.png)

##### 3.检测主从复制
验证：如果在从机上创建库，进行增删改操作，会同步到从机

在主机上创建新库hdg,可以看到从机上也自动创建了库hdg
![验证主从查询](/images/mysql实现主从复制（docker环境）-img/12.png)

在主机库hdg中添加表hdg,同时添加一条数据，可以看到从机上也自动更新
![验证主从复制](/images/mysql实现主从复制（docker环境）-img/13.png)

### 四、搭建主主复制
##### 1.在之前的从机上进行主机操作
![在从机上进行主机操作](/images/mysql实现主从复制（docker环境）-img/14.png)

![查询新主机信息](/images/mysql实现主从复制（docker环境）-img/15.png)


##### 2.在之前的主机上进行从机操作

![在之前的主机上进行从机操作](/images/mysql实现主从复制（docker环境）-img/16.png)

##### 3.检测如主从检测
### 五、 排错

   可能有些小伙伴没有成功。这里写个检查错误的方法：

在主库，执行：

```
show processlist;
```

```
[root@ed434a1db697 mysql]# netstat -natp
```

![p.png](/images/mysql实现主从复制（docker环境）-img/17.png)



