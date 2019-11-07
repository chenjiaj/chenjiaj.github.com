
---
title: docker中mysql备份
date:  2019.01.18 10:15
categories: 'mysql'
tags: ['docker', mysql]
---

以下示例均在镜像mysql:5.7.22基础上应用

## 一、备份mysql
#### 1.创建容器testmysql

    docker run -p 3307:3306 --name testmysql -e MYSQL_ROOT_PASSWORD=12345 -d mysql:5.7.22

#### 2.创建数据库test、数据表test1

    docker exec -it testmysql /bin/bash //进入容器

    mysql -uroot -p //进入数据库

    show databases; //查询数据库
    create database test; // 创建名为test数据库
    use test;//切换到数据库 test
    create table test (id int);//创建名为test的数据表
    insert into test (id) values (1); //插入一条数据

![添加数据](/images/docker-中mysql备份-img/1.png)


#### 2.mysqldump 备份

导出命令1（建议使用此命令）

docker exec -it mysql_server【docker容器名称/ID】 mysqldump --defaults-extra-file=/etc/mysql/my.cnf  test_db【数据库名称】 > /opt/sql_bak/test_db.sql【导出表格路径】



（1）修改/etc/mysql/my.cnf文件

    vim /etc/mysql/my.cnf

    [mysqldump]
    user=your_backup_user_name
    password=your_backup_password

![image.png](/images/docker-中mysql备份-img/2.png)

进入容器后执行vim /etc/mysql/my.cnf，如果报bash: vim: command not found
在容器内执行以下代码下载vim即可

    apt-get update
    apt-get install vim

修改后，exit退出容器，重启以下容器 docker restart testmysql【容器名称/id】

执行导出命令

    docker exec -it testmysql mysqldump --defaults-extra-file=/etc/mysql/my.cnf test > /home/iotdev/cjj/test.sql

导出命令2
docker exec -it  mysql_server【docker容器名称/ID】 mysqldump -uroot -p12345【数据库密码】 test_db【数据库名称】 > /opt/sql_bak/test_db.sql【导出表格路径】

    docker exec -it testmysql mysqldump -p12345 test > /home/iotdev/cjj/test.sql

如果使用这个命令，导出的文件会有警告 ，在导入的时候报这个错误,需要把警告删除再导入
![导入](/images/docker-中mysql备份-img/3.png)


## 二、导入备份

#### 1.创建容器testmysql1

    docker run -p 3308:3306 --name testmysql1 -e MYSQL_ROOT_PASSWORD=1234 -d mysql:5.7.22

#### 2.导入命令

(1)先将文件导入到容器

    docker cp **.sql 【容器名】:/root/

(2)进入容器

    docker exec -ti 【容器名/ID】/bin/bash

(3)创建数据库

    mysql -uroot -p
    create database test

(4)将文件导入数据库

     mysql -uroot -p 【数据库名】 < ***.sql


三、加入定时任务
用crontab添加定时器，参考详情 [https://www.cnblogs.com/peida/archive/2013/01/08/2850483.html](https://www.cnblogs.com/peida/archive/2013/01/08/2850483.html)

#### 1.添加myql_test.sh文件

    #!/bin/bash
data_name="rap2-mysql-new"
data_dir="/mysql_data/rap2-new"
database="RAP2_DELOS_APP"
time="`date +%H:%M`"
docker exec $data_name sh -c "rm -rf /$database"
docker exec $data_name sh -c "mkdir $database"
docker exec $data_name sh -c "mysqldump -uroot -pfewsdkjk13_ --databases $database > /$database/data_`date +%Y%m%d_%H:%M`.txt"
docker cp $data_name:$database/ $data_dir
# mv $data_dir/$database/data_`date +%Y%m%d_%H:%M`.sql $data_dir/$database/data_`date +%Y%m%d_%H:%M`.txt
# 将备份文件发送到邮箱 数据量小的情况下 服务器需要安装 yum install mailx 并配置 yum install sharutils
# 修改 vi /etc/mail.rc
# 添加以下配置
# set from=shenjianyu@thinktrader.net smtp=smtp.exmail.qq.com
# set smtp-auth-user=shenjianyu@thinktrader.net smtp-auth-password=邮箱密码
# set smtp-auth=login
# echo -e "rap2备份文件" | mail -s 'rap2备份' -a $data_dir/$database/data_`date +%Y%m%d_%H:%M`.txt 845142388@qq.com
if [ $time = "12:00" ]
then
    echo -e "rap2备份文件" | mail -s 'rap2备份' -a $data_dir/$database/data_`date +%Y%m%d_%H:%M`.txt 845142388@qq.com
fi
if [ $time = "20:00" ]
then
    echo -e "rap2备份文件" | mail -s 'rap2备份' -a $data_dir/$database/data_`date +%Y%m%d_%H:%M`.txt chenjiajun@cmiot.chinamobile.com
fi
find $data_dir -mtime +7 -name 'data_*.txt' -exec rm -rf {} \;

以上代码和之前的命令有些不一样，之前的命令是直接生成到宿主机，但不知为什么，直接执行.sh文件是可以正常生成文件的，但使用定时器生成的文件内容是空的，求大神答疑解惑。

问题代码如下：

    #!/bin/bash
    data_dir="/home/iotdev/cjj/"
    docker exec -it testmysql mysqldump -uroot -p12345 test > "$data_dir/data_`date +%Y%m%d_%H:%M`.sql"

##### 2.添加定时任务

  以下添加的是每分钟执行一次，可根据自己情况配置

    crontab -e
    * * * * * sh /home/cjj/mysql_test.sh 【.sh文件的位置】


四、添加脚本，删除时间太久的文件

    // 删除60分钟前的文件
    find $data_dir$container_dir -amin +60 -name 'data_*.sql' -exec rm -rf {} \;

    //删除7天前的数据
    find $data_dir$container_dir -mtime +7 -name 'data_*.sql' -exec rm -rf {} \;






