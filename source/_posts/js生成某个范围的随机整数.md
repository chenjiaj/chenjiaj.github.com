
---
title: js生成某个范围的随机整数
date: 2017.12.18 14:14
categories: 'js'
tags: js
---

 >js没有提供一个现成的函数直接生成某个范围的随机数。
    js只有一个Math.random() 函数返回一个浮点,  伪随机数在范围[0，1)。

我们只有利用Math.random() 函数，自己封装一些函数，实现生成某个范围的随机数。
实现生成某个范围色随机数也需要与一下函数配合使用：

>Math.ceil()    向上取整 
>Math.floor()   向下取整
>Math.round()   四舍五入

## 一、以0~10为例理解生成某个范围的随机数

首先我们以0~10为例，对生成某一返回有一个简单的理解：
Math.random() * 10 会随机生成 [0,10)，但是浮点数；

生成[0,10]的随机整数，Math.round(Math.random() * 10) ，通过四舍五入可以将大于9.5的数值转换为10；

生成[0,10)的随机整数，Math.floor(Math.random() * 10 )；

生成(0,10]的随机整数，Math.ceil(Math.random() * 10 )；

或者Math.round(Math.random() * 9 ) + 1  ，相当于[1,10]，Math.round(Math.random() * 9 )相当于Math.round(Math.random() * 10 * (9/10) )生成的[0,9]范围的随机值，再加一个1，就是[1,10]；
或者 var rand = Math.random(); Math.round(rand  * 10 ) === 0 ? Math.round(rand  * 10 ) + 1 : Math.round(rand  * 10 ) ;Math.round(rand  * 10 )生成[0,10]的随机数，但是做了一个判断，如果生成0，就转换成1；

生成(0,10)的随机整数,Math.round(Math.random() * 8) + 1,相当于[1,9],理解同上
或者var rand = Math.random(); Math.floor(rand  * 10 ) === 0 ？(Math.floor(rand  * 10 ) +1：(Math.floor(rand  * 10 ) ;

## 二、生成随机整数的四种情况

归纳总结为一下四种情况：

### 1.min ≤ r ≤ max 
实现函数如下：

    function Random(min, max) {
        return Math.round(Math.random() * (max - min)) + min;
    }

### 2.min﹤r ≦ max

    function Random(min, max) {
        return Math.ceil(Math.random() * (max - min)) + min;
    }

    function Random(min, max) {
        return Math.round(Math.random() * (max - min)) === 0? (min+1):Math.round(Math.random() * (max - min)) + min;
    }

### 3.min≦ r ﹤ max

    function Random(min, max) {
        return Math.floor(Math.random() * (max - min)) + min;
    }

    function Random(min, max) {
        return Math.round(Math.random() * (max - min)) === max ? (max-1):Math.round(Math.random() * (max - min)) + min;
    }

### 4.min﹤ r ﹤ max

    function Random(min, max) {
        return Math.floor(Math.random() * (max - min)) === min  ? (min + 1) : Math.floor(Math.random() * (max - min)) + min;
    }

    function Random(min, max) {
        return Math.floor(Math.random() * ((max-1) - (min+1))) + (min+1);相当于 min+1 ≤ r ≤ max - 1
    }
