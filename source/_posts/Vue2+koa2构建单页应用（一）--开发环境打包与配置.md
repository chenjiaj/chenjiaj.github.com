
---
layout: vue2
title: Vue2+koa2构建单页应用（一）- 开发环境打包与配置
date:  2017.08.18 16:26
categories: 'vue'
tags: vue
---

之前用Vue2+webpack+express构建单页应用，发现node端不能用es6的语法，为了前后端都用上es6的语法，node框架决定尝试koa2

代码地址：https://github.com/chenjiaj/Vue2-koa2-demo

## 一、需要环境
node版本在7及以上
选择新版本的环境是为了node端不用引入babel对es6做处理，新的node版本对es6有良好的支持。

## 二、创建基本项目

### 1.1 启动koa小项目demo

1.新建文件夹koa2+vue2-demo，然后进入文件夹，打开cmd进入文件夹内的目录或者webstorm的命令面板。

2.执行npm init命令生成package.json文件,然后执行npm install koa koa-onerror --save 下载koa和 koa-onerror

3.在项目根目录下新建config文件夹,然后进入文件夹新建config.js，输入以下内容，配置端口号

    module.exports = {
        node: {
		port: 3011
	}
  };    

4.在项目根目录新建app.js文件，然后输入以下内容，创建一个基本的项目

    const Koa = require('koa');
    const app = new Koa();
    const Config = require('./config/config');
    const onerror = require('koa-onerror');

    //错误信息处理
    onerror(app);

    //控制台打印请求信息
    app.use(async(ctx, next) => {
        const start = Date.now();
        await next();
        const ms = Date.now() - start;
        console.log(`${ctx.method} ${ctx.url} - ${ms}ms`);
    });

    app.listen(Config.node.port);

5. 修改package.json,在script对象中添加 "start": "node ./app.js"，如下图所示


![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/1.png)

6.此时在命令面板中输入npm start即可启动项目，然后在浏览器中访问 http://localhost:3011/，因为现在还没有对请求做任何处理，所以返回not found。

![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/2.png)


### 1.2 添加vue页面，实现hello world


1.在根目录下新建src,然后在src下添加index.html文件,并且添加一个id为app的div，如下。同时在命令面板输入 npm install vue --save

![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/3.png)


2.在src目录下新建views文件夹（存放所有的.vue文件），并在此文件夹下添加app.vue文件。在app.vue写入一下代码：
 
    <template>
        <div>hello world!</div>
    </template>

3.在src下新建main.js作为vue的入口文件。在main.js中添加以下内容：
      
    import Vue from 'vue';
    import App from './views/app.vue';

    new Vue({
      el: '#app',
      render: h=> h(App)
    });

现在目录结构如下：
![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/4.png)

一个hello world 的demo就写好了，但是要在浏览器看到效果，还需要引入vue包，同时也需要添加webpack打包需要的文件与配置。将vue内容打包引入index.html中，然后再将node接收到的页面请求返回打包后的index.html页面

### 1.3 添加webpack打包文件与配置

1.下载webpack打包需要的依赖包

    npm install webpack webpack-merge koa-webpack extract-text-webpack-plugin html-webpack-plugin css-loader file-loader vue-template-compiler vue-loader vue-style-loader  --save-dev

**webpack** ：webpack打包需要引入的核心包

**koa-webpack**：封装了webpack-dev-middleware和webpack-hot-middleware两个插件

webpack-dev-middleware是需要webpack打包的项目，开发时使用的中间件，主要作用是不需要将打包生成的文件放在硬盘中，而是放在内存中，这样可以提高开发效率，而且配合webpack-hot-middleware中间件使用可以实现热加载

**extract-text-webpack-plugin**：主要是为了抽离css样式,防止将样式打包在js中引起页面样式加载错乱的现象

**html-webpack-plugin**：是webpack的插件，这个插件用来简化创建服务于 webpack bundle 的 HTML 文件，尤其是对于在文件名中包含了 hash 值，而这个值在每次编译的时候都发生变化的情况。你既可以让这个插件来帮助你自动生成 HTML 文件，也可以使用 lodash 模板加载生成的 bundles，或者自己加载这些 bundles。

**vue-loader**：是webpack的loader,能够将.vue文件转换成js文件

**vue-html-loader、vue-template-compiler、css-loader、vue-style-loader**：这三个都是webpack的loader,都是将.vue文件转换成js文件的依赖

2.下载babel需要的依赖包

    npm install babel-core babel-loader babel-plugin-transform-runtime babel-polyfill babel-preset-es2015 babel-preset-stage-0 babel-runtime   --save-dev

**以babel-开头**：都是用于兼容es6写法，将es6的代码转换成es5的代码
**babel-core、babel-loader**：babel配合webpack工具使用必须要引入的
**babel-plugin-transform-runtime、babel-runtime**：解决重复出现在一些模块里，导致编译后的代码体积变大的问题。
**babel-preset-es2015**：将 ES2015 编译成 ES5
**babel-preset-stage-2**：除了覆盖stage-3的所有功能,不是对ES6功能的增加，而是为了增强代码的可读性和可修改性而提出的参考：[http://babeljs.io/docs/setup/#installation](http://babeljs.io/docs/setup/#installation)

3.在根目录下添加.babelrc文件，添加以下配置：

        {
              "presets": ["es2015","stage-0"],
              "plugins": ["transform-runtime"],
              "comments": false
         }

4.在根目录下新建build文件夹，然后添加webpack.base.conf.js、webpack.dev.conf.js、webpack.prod.conf.js
webpack.base.conf.js：开发和生成环境相同配置写字这个里边
webpack.dev.conf.js：针对开发时配置的文件
webpack.prod.conf.js：针对生产环境（正式上线）配置的文件

5.在webpack.base.conf.js中添加以下配置

     /* 引入操作路径模块和webpack */
    const path = require('path');
    const webpack = require('webpack');
    const HtmlWebpackPlugin = require('html-webpack-plugin');
    const ExtractTextPlugin = require('extract-text-webpack-plugin');

    module.exports = {
	/* 输入文件 */
	entry: {
		index: ['babel-polyfill', path.resolve(__dirname, '../src/main.js')]
	},
	output: {
		/* 输出目录，没有则新建 */
		path: path.resolve(__dirname, '../dist'),
		/* 静态目录，可以直接从这里取文件 */
		publicPath: '/',
		/* 文件名 */
		filename: 'js/[name].[hash].js',
		chunkFilename: 'js/[name].[chunkhash].js'
	},
	module: {
		rules: [{
			test: /\.vue$/,
			loader: 'vue-loader',
			options: {
				loaders: {
					css: ExtractTextPlugin.extract({
						use: 'css-loader',
						fallback: 'vue-style-loader' // <- this is a dep of vue-loader, so no need to explicitly install if using npm3
					})
				}
			}
		}, {//页面中import css文件打包需要用到
			test: /\.css/,
			loader: ExtractTextPlugin.extract({ fallback: 'style-loader', use: 'css-loader' })
		}, {
			test: /\.js$/,
			loader: 'babel-loader',
			/* 排除模块安装目录的文件 */
			exclude: /node_modules/
		},{
			test: /\.png$|\.jpg$|\.gif$|\.ico$/,
			loader: "file-loader",
			exclude: /node_modules/
		}]
	},
	plugins: [
		new HtmlWebpackPlugin({
			filename: 'index.html',
			template: path.resolve(__dirname, '../src/index.html'),
			inject: true
		}),
		new ExtractTextPlugin("style.css")
	]
};

6.我们先配置开发是需要文件，生成环境之后再近些配置。
首先在webpack.dev.conf.js中添加

    const merge = require('webpack-merge');
    const baseConfig = require('./webpack.base.conf');
    const webpack = require('webpack');

    let devConfig =  merge(baseConfig, {
	output: {
		path: '/'
	},
	plugins: [
		new webpack.HotModuleReplacementPlugin(),
		new webpack.NoEmitOnErrorsPlugin()
	]
    });

    module.exports = devConfig;

7.然后在app.js中添加webpack配置，如下


![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/5.png)


8.然后运行npm start启动项目，看到打印出一下日志，证明启动成功

![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/6.png)

9.在浏览器中访问http://localhost:3011/，效果如下，vue + koa构建一个hello world的小项目就成功了。

![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/7.png)

## 三、添加路由实现单页

### 1.1 vue添加路由

1.下载路由的依赖包 npm install vue-router --save

2.在src文件夹下添加router文件夹，然后新建index.js,存放路由主要配置

3.在views文件夹下，添加example文件，然后添加两个文件，分别是example.vue和example1.vue，作为路由中的两个页面。分别添加以下内容：

![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/8.png)

![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/9.png)

4.在router下的index.js编辑以下内容：


![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/10.png)


5.修改main.js如下，将路由：


![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/11.png)

6.修改app.vue，配置的路由中的compent将显示在router-view中

![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/12.png)

7.刷新浏览器，可以看到以下效果

![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/13.png)

点击example1将显示(点击example1跳转并未刷新页面，只是vue的路由跳转)

![.](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/14.png)

8.但是在http://localhost:3011/example1下刷新浏览器，现在会发现找不到

![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/15.png)

这是什么原因造成的呢？

因为vue的路由是在浏览器中进行管理，如果刷新http://localhost:3011/时可以访问到的，因为请求/路径，node将其指向了index.html(因为webpack打包会把index.html打包到根目录，而koa-webpack在没有传递参数的情况下也是指向的webpack配置文件中output中的publicPath，配置文件中配置的是/，所以默认/请求指向index.html),以下是koa-webpack中默认配置的源代码

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1210894-bb4f6395ba003114.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240z

而/example1请求到node，node会去寻找node端的路由处理，因为node端没有配置这个路由，所有找不到。

如果想要访问到/example1，需要请求到index.html,然后由vue的路由处理找到对应的模块

解决方法有：

（1）vue路由采用hash模式，修改router下的index.js文件，将mode的值改为hash，如果是hash，但是这样浏览器路径会变成http://localhost:3011/#/example1，有一点丑陋，http://localhost:3011/#/example1这个路径还是请求的/后面的会被当做参数处理

![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/16.png)

（2）将所有html请求转到index.html 然后让vue的浏览器处理

### 1.2 配置node端端

1.将所有的html请求转到index.html,有一个现成的插件connect-history-api-fallback替我们做了这件事，但是需要稍微封装一下才能在koa2中使用。
（1）npm install connect-history-api-fallback --save 下载插件包
（2）在根目录下添加middleware文件夹，用于存放koa的中间件
（3）在middleware文件夹中添加koa2-connect-history-api-fallback.js 文件，koa2中间件需要传入需要的方法，所以封装返回了一个方法

        const history = require('connect-history-api-fallback');
        module.exports = options=> {
	        const middleware = history(options);
	        const noop = ()  => {
	        };
	
	        return async (ctx, next)=> {
		      middleware(ctx, null, noop);
		    await next();
	        };
        };

2.在app.js添加const history = require('./middleware/koa2-connect-history-api-fallback');和
app.use(history({
	verbose: true//打出转发日志
}));
注意：app.use(history(...))要写在app.use(middleware({...}))之前，不然koa-webpack已经返回not found了，app.use(history(...))就不会生效了

11.写好之后重启服务，然后访问浏览器刷新http://localhost:3011/example1就可以访问到了
并且可以看到日志，只有请求html转到了index.html

![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/17.png)

### 1.3 优化

#### 1.添加less配置，让.vue文件支持写less文件
（1）下载less打包编译需要的依赖，npm install less less-loader --save-dev
（2）在webpack.base.conf.js中配置，添加这两个配置，将会识别.vue里边的less,也可以将less写成单独的文件

![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/18.png)

（3）在页面中引入less文件（通常便于维护，less写在单独文件中）
在src文件夹中添加static文件夹，然后在static文件夹中添加less文件，专门存放less文件，然后添加example.less文件，顺便写一些样式进去，如下

![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/19.png)


然后在example.vue文件中引入这个less文件

![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/20.png)

页面效果如下

![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/21.png)

#### 2.实现懒加载，优化初始化加载
在页面比较多的时，单页应用按照之前的方式打包成一个js会导致首页加载时很慢，为了解决这个问题，可以修改打包，让首页只加载通用代码和首页要用到的代码，跳转到其他页面再加载对应页面的js，这样可以解决项目较大首页加载缓慢的问题。

修改router中index.js,将.vue文件加载改为
const Example = r => require.ensure([], () => r(require('../views/example/example.vue')), 'Example');
const Example1 = r => require.ensure([], () => r(require('../views/example/example1.vue')), 'Example1');

因为webpack.base.conf.js已经加了chunkFilename: 'js/[name].[chunkhash].js'，所以不用再修改webpack配置

![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/22.png)

浏览页面，这个时候会发现，访问example页面只有example.js

![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/23.png)

点击example1页面，才会新加入example1.js，并且加载之后，以后返回example1页面不会重新加载，提高性能

![Paste_Image.png](/images/Vue2+koa2构建单页应用（一）--开发环境打包与配置-img/24.png)

经过以上步骤，我们已经可以在本地跑起来开发vue项目了。
