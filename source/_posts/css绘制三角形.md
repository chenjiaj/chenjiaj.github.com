
---
title: css绘制三角形
date: 2018.09.12 17:43
categories: 'css'
tags: css
---

#### 一、绘制固定宽高三角形
绘制三角形，主要通过border来实现，首先我们看一下四边border分配的效果。

```
<div class="triangle"></div>
```

```
.triangle{
    width: 0;
    height: 0;
    border-left: 100px solid red;
    border-top: 100px solid green;
    border-right: 100px solid blue;
    border-bottom: 100px solid pink;
}
```

效果如下：
![image.png](/images/css绘制三角形-img/1.png)

可以看到，但宽高为0，四边border的宽度相等的情况下会均分为4个三角形。

通过这个效果，我们可以通过控制四边border的透明、宽度来显示一个三角形。

例如：绘制一个直角三角形
顶部border为0，左右border设置宽度相等且透明的即可实现。

```
.triangle{
     width: 0;
     height: 0;
     border-left: 100px solid transparent;
     border-right: 100px solid transparent;
     border-bottom: 100px solid pink;
}
```
效果如下：

![image.png](/images/css绘制三角形-img/2.png)


#### 二、绘制宽高自适应三角形

通过clip-path和svg实现宽高自适应的三角形。
但这个方式兼容性不太好。
![image.png](/images/css绘制三角形-img/3.png)


```
<div class="t">
   dsfsdfs
   <br/>
   dsfsdfs
   <br/>
   dsfsdfs
   <br/>
</div>
```

```
.t{
     width: 100%;
     position: relative;
     text-align: center;
}

.t:after{
     position: absolute;
     content: '';
     top: 0;
     left: 0;
     right: 0;
     bottom:0;
     background: yellow;
     z-index: -1;
     clip-path: polygon(50% 0, 0% 100%, 100% 100%);
}
```
![image.png](/images/css绘制三角形-img/4.png)

效果如下：


#### 三、绘制固定宽高比三角形
思路：绘制一个足够大的三角形，外层三角形宽高比与三角形一致，且为百分比，根据屏幕改变，显示里边的内容，超出范围隐藏
```
<div class="t"></div>
```

```
      .t {
            width: 40%;
            padding-top: 8%;
            padding-left: 40%;
            overflow: hidden;
            border: 1px solid red;

        }

        .t:after {
            content: '';
            display: block;
            width: 0;
            height: 0;
            border-left: 1500px solid transparent;
            border-right: 1500px solid transparent;
            border-top: 300px solid #249ff1;
            margin-top: -300px;
            margin-left: -1500px;
        }
```

效果如下：（会根据屏幕宽度变化）
![image.png](/images/css绘制三角形-img/5.png)
