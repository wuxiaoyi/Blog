---
layout: post
title:  "ie下，动态改变图片src问题"
date:   2013-09-25 16:04:41 +0800
categories: javascript
---
昨天在工作中遇到了个问题。

在ie下点击验证码换一张的时候发现没法更换，查了查，原来是需要在src后面加上random参数。
而且看文章发现如果用a标签的话，加random参数也是不好使的，需要以href="xxxxx();return 
false;"来解决这个问题。