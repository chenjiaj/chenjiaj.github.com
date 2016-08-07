---
title: 使用hexo搭建githubPage博客
date: 2016-08-07 15:51:46
categories: 'hexo'
tags:
---

## 一、相关资料

1.hexo官网：https://hexo.io/zh-cn/

2.github page: https://pages.github.com/

## 二、搭建hexo本地环境，并上传到githubPage

1.新建hexo项目,并启动

	$ hexo init <folder>
	$ cd <folder>
	$ npm install
	$ hexo server

2.打开浏览器，默认访问http://localhost:4000/ 查看首页便可以看到效果

3.配置github路径,修改_config.yml

	deploy:
  		type: git
 		repo: git@github.com:chenjiaj/chenjiaj.github.com.git
  		branch: master
  

4.生成静态文件，并上传部署
	
	$ hexo generate
	$ hexo deploy

5.访问你的githubpage，便可以查看效果 ,例如：http://chenjiaj.github.io/

## 三、更换主题

1.主题列表：https://hexo.io/themes/

2.下载主题后，将主题放到themes文件夹中，然后修改_config.yml中theme为要使用的主题名称

3.有些主题需要把主题中的一些内容拷贝到source文件夹中，具体情况看具体的主题文档

## 四、hexo 和 jekyll 对比

之前使用的jekyll搭建github Page，个人觉得用起来很麻烦，后来果断改用hexo了

1.hexo用 node.js，jekyll 用 Ruby。根据个人喜好选择，个人觉得hexo搭建本地环境更方便，特别是对于前端开发来说。并且hexo生成速度更快

2.jekyll可以把原文上传到 github， 可以直接生成博客，也可以用在编辑器处理。相比而言，hexo这一点稍微麻烦一点，需要先生成html静态文件再上传，因为有现成的hexo generate，hexo deploy命令，所以操作起来也不麻烦。hexo需要把原文件传到另一个分支或者项目，便于对原文件的版本管理。







