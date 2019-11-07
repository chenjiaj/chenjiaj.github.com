---
layout: vue2
title: express4构建单页应用二
date: 2016-12-28 16:46:20
categories: 'vue'
tags: vue
---

根据[Vue2 + webpack + express4构建单页应用(一)](http://www.jianshu.com/p/2e65ac138df6)已经构建了一个基本的小应用，但是还没有解决jquery的ajax请求、style模块中使用less等问题

## 一、实现异步请求及转发

### 1.客户端发起请求

一直实现异步请求都是用ajax（XMLHttpRequest）来实现的  ,最近兴起了ajax的替代技术fetch,XMLHttpRequest 是一个设计粗糙的 API，不符合关注分离（Separation of Concerns）的原则，配置和调用方式非常混乱，而且基于事件的异步模型写起来也没有现代的 Promise，generator/yield，async/await 友好。

Fetch 的出现就是为了解决 XHR 的问题。

我在github上选择了一个支持前后端同构的fetch插件：https://github.com/matthew-andrews/isomorphic-fetch

在项目中执行 npm install --save isomorphic-fetch es6-promise 下载插件，可以在需要的地方按照下面方式使用：

    import es6Promise from 'es6-promise';
    import fetch from 'isomorphic-fetch';
    es6Promise.polyfill();

    fetch('//offline-news-api.herokuapp.com/stories')
        .then(function(response) {
            if (response.status >= 400) {
                throw new Error("Bad response from server"); } return response.json(); }
         )
        .then(function(stories) { console.log(stories); });

一般在项目中会将以上代码抽离出来写成一个方法单独抽离出成一个文件，使用时引入文件调用该方法

### 2.node端转发异步请求

node有一个http转发的中间件http-proxy-middleware

npm install http-proxy-middleware --save 下载中间件

然后在app.js中引入中间件并注册

    var proxy = require('http-proxy-middleware');
    app.use('/api', proxy({target: 'http://www.example.org', changeOrigin: true}));

详细使用，参考 https://github.com/chimurai/http-proxy-middleware#options

## 二、使用less预编译语言

现在写样式一般可以使用预编译语言less或者sass

根据个人习惯我选用的less，使用less需要有一下配置：

1.在.vue文件中的<style>需要加上lang="less"属性，如：

    <style lang="less">

2.下载less-loader、less插件，npm install --save-dev less-loader less

3.在webpack.base.conf.js中加上postcss: [require('autoprefixer')()]，如下：

    vue: {
        loaders: {  js: 'babel' },
        postcss: [require('autoprefixer')()]
    }

这样就可以在.vue中的style里边写less语法了

## 三、将.vue中的css单独提出来成一个css文件

需要使用webpack的extract-text-webpack-plugin插件

参考文档：https://vue-loader.vuejs.org/en/configurations/extract-css.html

npm install extract-text-webpack-plugin --save-dev

一般只需要在生成环境提取出来，于是在webpack.prod.conf.js中添加

    vue: {
        loaders: {
            css: ExtractTextPlugin.extract('vue-style-loader', 'css'),
            // you can also include <style lang="less"> or other langauges
            less: ExtractTextPlugin.extract('vue-style-loader', 'css!less')
        }
    }

在plugins中加入new ExtractTextPlugin("static/css/style.css")，这样css就是在output/static/css中生成style.css文件

## 四、引用图片

如果template中或style中引用了图片，需要添加file-loader

参考：https://vue-loader.vuejs.org/en/configurations/asset-url.html

1.下载file-loader    npm install --save-dev file-loader
2.在webpack.base.conf.js中的loaders里边添加

     {
        test: /\.png$|\.jpg$|\.gif$|\.ico$/,
        loader: "file?name=static/img/[name].[hash].[ext]",
        exclude: /node_modules/
    }

## 五、实现懒加载（按需加载）

在页面比较多的时，单页应用按照之前的方式打包成一个js会导致首页加载时很慢，为了解决这个问题，可以修改打包，让首页只加载通用代码和首页要用到的代码，跳转到其他页面再加载对应页面的js，这样可以解决项目较大首页加载缓慢的问题。

[官方文档地址](http://router.vuejs.org/zh-cn/advanced/lazy-loading.html)

### 1.修改路由

将要通过懒加载的路由的compenent改写成

    const Plugin = r => require.ensure([], () => r(require('../views/Plugins/plugin')), 'Plugin');

具体路由页面如下：


![Paste_Image.png](/images/Vue2-+-webpack-+-express4构建单页应用二-img/1.png)

### 2.修改webpack配置

在webpack.prod.conf.js中的output中添加

    chunkFilename: 'static/js/[id].[name]_[chunkhash:7].js'

运行 npm run build 这个时候就可以看到每一个require.ensure引入的模块都单独生成了一个js

![Paste_Image.png](/images/Vue2-+-webpack-+-express4构建单页应用二-img/2.png)

运行npm run prod启动生产环节，访问页面http://localhost:3001/
因为我没有定义首页路由，所以会跳转到下面模块中：

![Paste_Image.png](/images/Vue2-+-webpack-+-express4构建单页应用二-img/3.png)

查看请求，只请求了index.js和notFoundComponent.js。

![Paste_Image.png](/images/Vue2-+-webpack-+-express4构建单页应用二-img/4.png)

访问我定义的路由的http://localhost:3001/plugin（根据你自己定义的路由查看），可以看到除了公用的index.js以为，其他js并没有加载，如果通过路由跳转的话也不用重新加载index.js

![Paste_Image.png](/images/Vue2-+-webpack-+-express4构建单页应用二-img/5.png)

细心的你如果仔细看的话会发现一个问题，除了app.vue里边的css打包到style.css外，其他vue里的js并没有打包进去

于是还需要在webpack.prod.conf.js的plugins中将

    new ExtractTextPlugin("static/css/style.css")

修改为

    //实现css分块，讲所有vue文件中的css打包到一个一个入口css中
    new ExtractTextPlugin('static/css/[id].[name]_[chunkhash:7].css', {
        allChunks: true
    })

    //加上这个插件，可以将通用的css和js单独打包成一个vendors.css和vendors.js
    new webpack.optimize.CommonsChunkPlugin({
        name:'vendors',
        filename:'static/js/[id].[name].[hash].js',
        minChunks: function () {  return true }
    })
minChunks:num 表示require了num此才放在CommonChunk里边

再次执行npm run build会生成一下目录

![Paste_Image.png](/images/Vue2-+-webpack-+-express4构建单页应用二-img/6.png)

会发现app.vue里边的css被打包到了vendor.css中了，app.vue里的js和vue/vue-router被打包到了vendor.js中。

这样就基本完成了懒加载。

但是还没有实现如何将css安装vue文件打包成单独的文件，请大神多多指教！

备忘：生命周期（[参考](https://segmentfault.com/a/1190000008010666)）
![Paste_Image.png](/images/Vue2-+-webpack-+-express4构建单页应用二-img/7.png)
