
---
layout: vue2
title: express4构建单页应用一
date: 2016-12-28 16:45:31
categories: 'vue'
tags: vue
---

想要构建一个vue2的单页应用，发现vue-cli生成的项目虽然可以直接用，但是生成的项目是用的vue1，而且直接使用也不能完全掌握和了解项目用了哪些插件和打包工具，为了自己更好的了解和学习开发vue2需要用到的打包工具和相关插件，自己看了许多博客，总结了以下一个构建基本vue2单页项目的流程，对于vue2和webpack小白可以跟着步骤试着自己构建一个小项目，如有大神请多指教。

完整源码链接：https://github.com/chenjiaj/vue2-express4-webpack

## 一、环境
需要安装node v4.4.4及以上

## 二、创建基本项目
1.打开cmd,运行 npm install express-generator -g （express-generator是express的应用生成器,相关介绍:http://www.expressjs.com.cn/starter/generator.html
用生成器生成主要是为了使用bin文件夹下的www文件，为了少写一点打印的日志而已，对应不需要的，可以不用生成器生成，自己创建server就行，可以用http模块或者直接调用app.listen都可以

2.切换到你要生成项目的目录，然后运行 express myapp，生成新的express项目，目录如下：

![Paste_Image.png](/images/Vue2-+-webpack-+-express4构建单页应用一-img/1.png)

3.删除public、routes、views文件夹，现在项目目录如下

![Paste_Image.png](/images/Vue2-+-webpack-+-express4构建单页应用一-img/2.png)

4.在根目录下创建index.html，内容如下：

    <!DOCTYPE html>
    <html lang="en">
      <head>   
         <meta charset="UTF-8"> <title>Title</title>
      </head>
      <body>  
         <div id="app"></div>
      </body>
     </html>

5.在根目录下添加src文件，然后创建app.vue和main.js

app.vue的内容如下：

    <template>
       <div>hello , this app vue !</div>
    </template>
    <style>  
         body{color:red;}
    </style>

main.js的内容如下：

    import Vue from "vue";
    import App from "./app";
    new Vue({
        el:'#app', 
        render:h=> h(App)
    })

如果是用的webStorm,请将language设置为ECMAScript6,这样才不会报错误提示
![Paste_Image.png](/images/Vue2-+-webpack-+-express4构建单页应用一-img/3.png)

6.需要安装以下依赖，将package.json文件替换如下：

      {  "name": "myapp",  
          "version": "0.0.0", 
          "private": true,  
          "scripts": {    "start": "node ./bin/www"  },     
          "devDependencies": {
            "babel-core": "^6.20.0",    
            "babel-loader": "^6.2.9",    
            "babel-plugin-transform-runtime": "^6.15.0",    
            "babel-preset-es2015": "^6.18.0",   
            "babel-runtime": "^5.8.38",    
            "babel-preset-stage-2": "^6.18.0",
            "html-webpack-plugin": "^2.24.1",   
            "vue-html-loader": "^1.2.3",   
            "css-loader": "^0.26.1",
            "vue-loader": "^10.0.2",    
            "vue-template-compiler": "^2.1.6",    
            "webpack": "^1.14.0",   
            "webpack-dev-middleware": "^1.6.1" 
           }, 
           "dependencies": {    
              "express": "^4.14.0",   
              "vue": "^2.1.6",   
              "vue-router": "^2.1.1" 
           }
      }

**以babel-开头**：都是用于兼容es6写法，将es6的代码转换成es5的代码
**babel-core、babel-loader**：babel配合webpack工具使用必须要引入的
**babel-plugin-transform-runtime、babel-runtime**：解决重复出现在一些模块里，导致编译后的代码体积变大的问题。
**babel-preset-es2015**：将 ES2015 编译成 ES5
**babel-preset-stage-2**：除了覆盖stage-3的所有功能,不是对ES6功能的增加，而是为了增强代码的可读性和可修改性而提出的
参考：http://babeljs.io/docs/setup/#installation

**webpack** ：webpack打包需要引入的核心包

**html-webpack-plugin**：是webpack的插件，这个插件用来简化创建服务于 webpack bundle 的 HTML 文件，尤其是对于在文件名中包含了 hash 值，而这个值在每次编译的时候都发生变化的情况。你既可以让这个插件来帮助你自动生成 HTML 文件，也可以使用 lodash 模板加载生成的 bundles，或者自己加载这些 bundles。

**vue-loader**：是webpack的loader,能够将.vue文件转换成js文件

**vue-html-loader**、**vue-template-compiler**、**css-loader**：这三个都是webpack的loader,都是将.vue文件转换成js文件的依赖

**webpack-dev-middleware** ：需要webpack打包的项目，开发时使用的中间件，主要作用是不需要将打包生成的文件放在硬盘中，而是放在内存中，这样可以提高开发效率，而且配合webpack-hot-middleware中间件使用可以实现热加载（热加载插件后面会介绍）

7.在根目录下添加build文件夹，在文件夹内创建webpack.base.conf.js，内容如下：

    // nodejs 中的path模块
    var path = require('path');
    var webpack = require('webpack');
    var HtmlWebpackPlugin = require('html-webpack-plugin');
    var  webpackConf = {
        // 入口文件，path.resolve()方法，可以结合我们给定的两个参数最后生成绝对路径，最终指向的就是我们的index.js文件 
        entry: {  index: [   path.resolve(__dirname, '../src/main.js')  ] },
        // 输出配置 
        output: {  // 输出路径是 myProject/output/static  
            path: path.resolve(__dirname, '../output'), 
            publicPath:'/',  filename: '[name].[hash].js' 
        },
         resolve: {  
            extensions: ['', '.js', '.vue'], 
            alias: {   'vue': 'vue/dist/vue.js'  } // 设置别名vue1不需要设置，vue2必须设置 否则会报错 
        },
        module: {  
            loaders: [   
            // 使用vue-loader 加载 .vue 结尾的文件   
                {    test: /\.vue$/,    loader: 'vue'   },  
                 {    test: /\.js$/,    loader: 'babel',    exclude: /node_modules/   }  ]
        }, 
        vue: {  loaders: {   js: 'babel'  } }, 
        plugins:[  
            new HtmlWebpackPlugin({   
                  filename: 'index.html',  
                  template: path.resolve(__dirname, '../index.html'),  
                  inject: true  
              }) ]
     };
     module.exports  = webpackConf;


`path`：用来存放打包后文件的输出目录 
`publicPath`：指定资源文件引用的目录 

8.将根目录下app.js内容替换如下：

    var express = require('express');
    var path = require('path');
    var webpack = require('webpack');
    var webpackConfig = require('./build/webpack.base.conf');
    var app = express();// 创建一个express实例
    var compiler = webpack(webpackConfig);// 调用webpack并把配置传递过去
    // 使用 webpack-dev-middleware 中间件
    var devMiddleware = require('webpack-dev-middleware')(compiler, { 
       publicPath: '/', 
       stats: {   
         colors: true,    
        chunks: false  
      }});
    app.use(devMiddleware);
    module.exports = app;

10.在根目录下添加.babelrc文件，内容如下：

    {  "presets": ["es2015","stage-2"],  "plugins": ["transform-runtime"],  "comments": false}

`presets ` 字段设定转码规则，官方提供以下的规则集，你可以根据需要安装。
`comments`是否输出输出中的注释

9.在根目录下运行 npm install ,然后运行npm start,webpack编译完之后如下图所示，就可以通过http://localhost:3000 访问了，效果如图所示：

![Paste_Image.png](/images/Vue2-+-webpack-+-express4构建单页应用一-img/4.png)

![Paste_Image.png](/images/Vue2-+-webpack-+-express4构建单页应用一-img/5.png)

这样就生成了一个简单的应用。

## 三、添加热加载

开发是需要热加载，即修改完代码后重新编译并更新浏览器，提高开发效率
热加载需要webpack-hot-middleware中间件（https://www.npmjs.com/package/webpack-hot-middleware）
1.在项目根目录下运行npm install --save-dev  webpack-hot-middleware
2.打开app.js,添加app.use(require("webpack-hot-middleware")(compiler));
3.在webpack.base.conf.js中的plugins中添加

      // Webpack 1.0
      new webpack.optimize.OccurenceOrderPlugin(),
      // Webpack 2.0 fixed this mispelling
      // new webpack.optimize.OccurrenceOrderPlugin(),
      new webpack.HotModuleReplacementPlugin(),
      new webpack.NoErrorsPlugin()

**方法一:只完成以下步骤4**

4.在webpack.base.conf.js中添加以下代码：

    Object.keys(webpackConf.entry).forEach(function (name) { 
        var extras = ['webpack-hot-middleware/client?reload=1'];
         webpackConf.entry[name] = extras.concat(webpackConf.entry[name]);
    });

**方法二:以下步骤4-6**

4.在build文件夹中新建hot-client.js文件，内容如下：
  
    // 动态向入口配置中注入 webpack-hot-middleware/client
    var hotClient = require('webpack-hot-middleware/client')
    // 订阅事件，当 event.action === 'reload' 时执行页面刷新
    hotClient.subscribe(function (event) {
         if (event.action === 'reload') {  window.location.reload() }
    })

5. 修改webpack.base.conf.js中的enter,改为以下代码：


      entry: { 
        index: [ 
            'webpack-hot-middleware/client',  
            path.resolve(__dirname, '../src/main.js') ]
      }

6.在webpack.base.conf.js中添加以下代码：

    var hotClient = './build/hot-client';
    Object.keys(webpackConf.entry).forEach(function (name) { 
        var extras = [hotClient];
         webpackConf.entry[name] = extras.concat(webpackConf.entry[name]);
    });

重新启动服务，修改vue或者js可以看到浏览器自动更新了

** 注意如果编辑器是webstorm的要将settings=>appearance=>system settings中synchronization中的最后一个选项不勾选，否则没有效果 **

![Paste_Image.png](/images/Vue2-+-webpack-+-express4构建单页应用一-img/6.png)

## 四、将开发的webpack配置提取出来

因为开发时需要webpack-dev-middleware、webpack-hot-middleware这两个中间件，但是正真发布的时候并不需要，因此需要将这两个的配置提取出来。

webpack-dev-middleware 这个中间件的作用是 开发的时候不生成代码，编译后的代码放在内存中，不会写入硬盘，因此不用每次编译都生成新的文件，提高开发效率

webpack-hot-middleware用于热加载，修改代码后立即更新

1.首先运行npm install --save-dev webpack-merge 下载 webpack-merge 工具，用于合并webpack配置使用

2.运行 npm install --save-dev cross-env 下载cross-env工具，用户在命令中设置环境变量

3.在build文件夹中新建webpack.dev.conf.js，写入以下代码：
    
    var path = require('path');
    var webpack = require('webpack');
    var webPackBaseConf = require('./webpack.base.conf');
    var HtmlWebpackPlugin = require('html-webpack-plugin');
    var merge = require('webpack-merge');
    var hotClient = './build/hot-client';

    var webpackDevConf = merge(webPackBaseConf,{ 
        entry: { index: [  hotClient,  path.resolve(__dirname, '../src/main.js') ]},
        plugins:[  
        // Webpack 1.0  new webpack.optimize.OccurenceOrderPlugin(), 
        // Webpack 2.0 fixed this mispelling  
        // new webpack.optimize.OccurrenceOrderPlugin(),  
        new webpack.HotModuleReplacementPlugin(),  
        new webpack.NoErrorsPlugin()
        ]});
  
    Object.keys(webpackDevConf.entry).forEach(function (name) { 
        var extras = [hotClient]; 
        webpackDevConf.entry[name] = extras.concat(webpackDevConf.entry[name]);
    });
    module.exports = webpackDevConf;

然后删除webpack.base.conf.js中对应的代码

注意删除的时候小心，不要将webpack.base.conf.js中的下面代码删掉了

        plugins:[new HtmlWebpackPlugin({   
            filename: 'index.html',   
            template: path.resolve(__dirname, '../index.html'),   
            inject: true  }) 
        ]})]

4.修改package.json中script中start改为如下：

    "start": "cross-env NODE_ENV=development node bin/www"

运行这个命令会将环境变量设置为development（默认也是development ，但为了避免环境配置被修改的情况下可以手动设置一次）

5.修改app.js,添加环境变量判断，将开发使用的中间件添加到条件成立时执行，改为如下：

    var express = require('express');
    var path = require('path');
    // 创建一个express实例
    var app = express();
    if(app.get('env') == 'development'){  
          var webpack = require('webpack');
          var webpackConfig = require('./build/webpack.dev.conf');
         // 调用webpack并把配置传递过去 
         var compiler = webpack(webpackConfig);  
        // 使用 webpack-dev-middleware 中间件  
        var devMiddleware = require('webpack-dev-middleware')(compiler, {   
           publicPath: '/',   
           stats: {      colors: true,      chunks: false    }  
        });    
        app.use(devMiddleware);  
        app.use(require("webpack-hot-middleware")(compiler));  

    }
    module.exports = app;

重新启动服务查看效果

五、添加路由实现单页应用

1.按照下图添加文件夹和文件

![Paste_Image.png](/images/Vue2-+-webpack-+-express4构建单页应用一-img/7.png)

说明：
components文件夹中放公用组件
router放置路由相关配置的js
views下放置单页的vue，按照模块放置

现在模拟一个场景：
有两个页面首页和用户页

1.修改main.js

    import Vue from "vue";
    import router from './router/index';//引入路由配置
    import App from "./app";
    import VueRouter from "vue-router";

    Vue.config.debug = true;//开启错误提示

    Vue.use(VueRouter);

    const app = new Vue({ 
        el:'#app', 
        router:router,//添加路由配置
        render: h => h(App)
    });

2.修改app.vue

添加<router-view></router-view>（路由更新的地方）
**注意<template></template>中的代码必须有一个包裹元素**

3.在/src/router/index.js中添加：

    import VueRouter from "vue-router";
    import Index from "../views/Index/index";
    import User from "../views/User/user";
    const routes = [
    {
         path:'/', component:Index
    },
    { 
        path:'/user', component:User
    }];
    const router = new VueRouter({ mode: 'history', routes});
    module.exports = router;

4.在index.vue和user.vue中分别加上<template><div>this index page</div></template>和<template><div>this user page</div></template>

5.运行npm install --save connect-history-api-fallback，然后在app.js中加入
var history = require('connect-history-api-fallback');
app.use(history());//放在app.use(devMiddleware);之前才有效(放在    if(app.get('env') == 'development'){  之前)

单页请求，需要加上connect-history-api-fallback中间件否则会报404

** 注意:一定要放在app.use(devMiddleware);之前才有效 **

参考：https://github.com/bripkens/connect-history-api-fallback

重启服务，就可以访问http://localhost:3000看到如下


![Paste_Image.png](/images/Vue2-+-webpack-+-express4构建单页应用一-img/8.png)


访问http://localhost:3000/user 效果如下

![Paste_Image.png](/images/Vue2-+-webpack-+-express4构建单页应用一-img/9.png)

完成以上步骤就可以开发用vue开发一个简单的单页应用了

## 五、生成环境打包

1.运行 npm install shelljs  ora --save-dev

2.在build文件下创建一个webpack.prod.conf.js.内容如下：
    
    var path = require('path');
    var webpack = require('webpack');
    var merge = require('webpack-merge');
    // 引入基本配置
    var webpackConf = require('./webpack.base.conf');
    var prodWebpackConf = merge(webpackConf,{ 
        output:{  publicPath:'/',  filename: 'static/js/[name].[hash].js' },
        plugins:[ 
             new webpack.optimize.UglifyJsPlugin({  
                compress: {    warnings: false   } 
            })
        ]});
    module.exports = prodWebpackConf;

3.在build文件下创建一个build.js文件，内容如下：

    // https://github.com/shelljs/shelljs
    require('shelljs/global');
    var path = require('path');
    var ora = require('ora');
    var webpack = require('webpack');
    var webpackConf = require('./webpack.prod.conf');
    console.log( '  Tip:\n' + '  Built files are meant to be served over an HTTP server.\n' + '  Opening index.html over file:// won\'t work.\n');
    var spinner = ora('building for production...');
    spinner.start();
    var assetsPath = path.join('/', 'static');
    rm('-rf', assetsPath);
    mkdir('-p', assetsPath);
    cp('-R', 'static/', assetsPath);
    webpack(webpackConf, function (err, stats) { 
        spinner.stop();
        if (err) throw err 
        process.stdout.write(stats.toString({   
            colors: true,   
            modules: false,   
            children: false,   
            chunks: false,   
            chunkModules: false  
        }) + '\n')
     });
    
4.在package.json的script中添加 "build": "node build/build.js"

5.运行npm run build 之后会生成一下文件
  
![Paste_Image.png](/images/Vue2-+-webpack-+-express4构建单页应用一-img/10.png)

    
6.修改app.js,在if(app.get('env') == 'development'){......}条件后添加
    else{
        app.use(express.static('output'));
    }

7.在package.json的script中添加 "prod":"cross-env NODE_ENV=production node bin/www"

8.运行npm run prod 就可以查看成产环境了

9.一般生成环境用pm2来起服务，于是在根目录下创建pm2.json 内容如下：

    {  "apps": [    {      "script": "bin/www"    }  ]}
connect-history-api-fallback中间件注册一定要在app.use(express.static(path.join(__dirname, 'output')));之前
10.在package.json的script中添加 "pm2_start":"cross-env NODE_ENV=production pm2 start pm2.json"

运行 npm run pm2_start就可用pm2启动服务 （运行前需要安装过全局的pm2 ，   npm install -g pm2）

完成以上步骤就可以基本完成一个简单的vue单页应用了


**注意**

1.connect-history-api-fallback中间件注册一定要在webpack-dev-middleware中间件注册之前
2.connect-history-api-fallback中间件注册一定要在app.use(express.static(path.join(__dirname, 'output')));之前


参考链接：

1.vue2官网：http://cn.vuejs.org/v2/guide/
2.vue-loader文档：https://vue-loader.vuejs.org/en/
3.vue-cli创建项目介绍：https://vuejs-templates.github.io/webpack/
4.[Vue作者尤雨溪：Vue 2.0，渐进式前端解决方案](http://mp.weixin.qq.com/s?__biz=MzIwNjQwMzUwMQ==&mid=2247484393&idx=1&sn=142b8e37dfc94de07be211607e468030&chksm=9723612ba054e83db6622a891287af119bb63708f1b7a09aed9149d846c9428ad5abbb822294&mpshare=1&scene=1&srcid=1026oUz3521V74ua0uwTcIWa&from=groupmessage&isappinstalled=0#wechat_redirect&utm_source=tuicool&utm_medium=referral)
