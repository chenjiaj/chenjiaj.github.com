
---
title: request中间件的gzip
date: 2018.02.13 16:37
categories: 'node'
tags: node
---

今天遇到一个问题，向文件服务器上传文件，上传成功后文件解析响应内容报错，由于英文不好，绕了不少弯路，记录一下，避免下次遇到忘记。

将返回的body打出发现是乱码，最后发现是gzip的问题。node端request中间件向服务器发送的accept-encoding="gzip, deflate, br",服务器返回的content-encoding:gzip。但是request默认是没有将响应内容解压的，导致运行JSON.parse(body)的时候就会报错。

代码如下：

    router.post('/fileupload', function (req, res, next) {
	  req.setTimeout(1800000);
	  req.pipe(request.post(fileUpload.url, function (e, r, body) {
		if (!e) {
			if (r.statusCode == 200) {
				try {
					var importFileUrl = JSON.parse(body).url;
					var privateUrl = JSON.parse(body).privateUrl;
					var tr069Url = JSON.parse(body).tr069Url;
					var internetUrl = JSON.parse(body).internetUrl;
					req.body = {
						serviceName: 'xxx',
						methodName: 'xxx',
						parameters: {
							filePath: importFileUrl,
							privateUrl: privateUrl,
							tr069Url: tr069Url,
							internetUrl: internetUrl
						}
					};
					console.log('--req.body-',req.body);
					postRequest(req, res, 1800000);
				} catch (err) {
					console.log(err);
					res.send({
						resultMsg: '文件服务器返回错误',
						resultCode: -2
					});
				}
			}
		  } else {
			console.log(e);
			res.send({
				resultMsg: '服务器内部错误，请重试',
				resultCode: -2
			});
		  }
	  }));
    });

服务器返回如下：

![image.png](/images/request中间件的gzip-img/1.png)

请求头部：

![image.png](/images/request中间件的gzip-img/2.png)



发现最终原因是：

请求头部accept-encoding="gzip, deflate, br"，表示是可以接受gzip编码的响应，但实际上服务器返回gzip编码的响应式却没有处理，导致报错。

查阅了一下request的相关文档，发现需要设置gzip:true，响应后才会对相应的内容进行解压处理。

文档内容：
![image.png](/images/request中间件的gzip-img/3.png)

![image.png](/images/request中间件的gzip-img/4.png)


代码修改如下：

    router.post('/fileupload', function (req, res, next) {
	  req.setTimeout(1800000);
        req.pipe(request.post(fileUpload.url,{
              gzip: true,//是设置为true
              /*proxy: 'http://127.0.0.1:8888' // 为了用fiddler抓包，将返回内容替换成相应为gzip的,方便测试*/
          } function (e, r, body) {
                ...
          })
    })

