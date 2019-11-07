
---
title: 用户代理字符串检测技术
date:  2019. 01.18 10:15
categories: 'node'
tags: node
---

### 一、mysql库
文档：[https://github.com/mysqljs/mysql](https://github.com/mysqljs/mysql)

mysql有三种创建连接方式
#### 1.createConnection
使用时需要对连接的创建、断开进行管理
#### 2.createPool
创建资源池，使用时不需要对连接的创建、断开进行管理，每次使用完调用一次release进行释放连接到资源池，至于连接是否断开交给资源池去管理。每次建立连接时非常消耗资源的，影响性能，因此对连接创建合理的管理，有利于提高性能。
#### 3.createPoolCluster
创建连接池集群，允许与多个host连接

### 二、sequelize库

中文文档：[https://github.com/demopark/sequelize-docs-Zh-CN](https://github.com/demopark/sequelize-docs-Zh-CN)

此库依赖mysql2

与mysql库相比，不需要写sql语句，增删查改都封装成对应的方法。
mysql库入门比较简单，有利于学习sql语句
sequelize封装了一些简单sql语句，掌握封装的方法及对应的参数即可，但学习成本稍微高一些，需要创建模式，模式需要与数据库中的表对应起来。在项目实际开发过程中，使用sequelize开发效率更高，代码可以更加简短。也有query方法，支持使用sql语句。

sequelize提供了一个方法sequelize.sync({ force: true });强制数据库中的表与模式定义的表进行同步，如果数据库中存在与模式定义同名的表，此表会被删除，重新定义。如果数据库中存在模式未定义的表，不会对其进行操作。
也可以对单个模式设置强制同步
如：

    // 注意:如果表已经存在,使用`force:true`将删除该表
    User.sync({ force: true }).then(() => {
      // 现在数据库中的 `users` 表对应于模型定义
      return User.create({
        firstName: 'John',
        lastName: 'Hancock'
      });
    });

强制同步也有风险点：
1.适合在项目初始化，需要创建数据表的时候使用
2.如果数据库中已有部分数据，当服务重启时，数据库中模式定义的表会被删除，数据会丢失，因此不适合在非初始化时使用，因此在使用时需要判断是否时初始化的情况













