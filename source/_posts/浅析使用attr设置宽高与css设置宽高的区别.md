
---
title: 浅析使用attr设置宽高与css设置宽高的区别
date: 2016-08-31 13:59:14
categories: 'canvas'
tags: canvas
---

### 一、主要区别
使用attr设置宽高，attr对于的是实际宽高，如果没设置css就是显示的实际宽高，如果设置了css就会跟随css变成对应的显示宽高，这个时候实际宽高就可能跟显示宽高不一样，但是你所有操作对应的还是实际宽高而不是显示宽高

### 二、应用场景

主要是针对图片、视频、canvas
以例子来说明更有说服力，如下：
想要实现画一个800*800的灰色canvas画布，然后清空坐标点在（40,40）,且宽高为140的矩形。如下图所示：
![Paste_Image.png](/images/浅析使用attr设置宽高与css设置宽高的区别-img/1.png)

先看一下第一段代码，通过css设置宽高：

    <!DOCTYPE html>
    <html lang="en">
    <head>
	<meta charset="UTF-8">
	<title>截图练习</title>
    </head>
    <body>
	<script src="js/jquery-1.11.1.min.js"></script>
	<script type="text/javascript">
		var width = 800;
		var height = 800;
		var $blankCanvas = $('<canvas></canvas>')
			.css({//通过css设置的宽高
				width: width,
				height: height,
				position: 'absolute',
				top: 0,
				left: 0,
				cursor: 'crosshair'
			});

		var cxt = $blankCanvas[0].getContext('2d');
		cxt.fillStyle = "rgba(0,0,0,.2)";
		cxt.fillRect(0, 0, width, height);
		cxt.clearRect(40,40,140,140);
		$('body').append($blankCanvas);
	</script>
    </body>
    </html>

发现效果如下：
![Paste_Image.png](/images/浅析使用attr设置宽高与css设置宽高的区别-img/2.png)

很明显宽高远大于140，并且左上角的坐标也不是（40,40），并且四边还有点模糊，像是被拉伸一样

原因是：
通过css设置的是显示宽高，没有通过attr设置canvas的实际宽高，但canvas默认的实际宽高是300*150，但操作还是针对canvas的实际宽高操作的，因此看到的效果是缩放后的效果

如何达到要求的效果呢，需要修改canvas的实际宽高，如下

    <!DOCTYPE html>
    <html lang="en">
    <head>
	<meta charset="UTF-8">
	<title>截图练习</title>
    </head>
    <body>
	<script src="js/jquery-1.11.1.min.js"></script>
	<script type="text/javascript">
		var width = 800;
		var height = 800;
		var $blankCanvas = $('<canvas></canvas>')
			.attr({
				width: width,
				height: height
			})
			.css({
				position: 'absolute',
				top: 0,
				left: 0,
				cursor: 'crosshair'
			});

		var cxt = $blankCanvas[0].getContext('2d');
		cxt.fillStyle = "rgba(0,0,0,.2)";
		cxt.fillRect(0, 0, width, height);
		cxt.clearRect(40,40,140,140);
		$('body').append($blankCanvas);
	</script>
    </body>
    </html>

补充：更改CSS的宽高只会导致canvas的缩放，但是更改属性的宽高会导致canvas的图形丢失（canvas被重置了，所有数据都没了）
