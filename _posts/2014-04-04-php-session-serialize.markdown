---
layout: post
title:  "php的session存储对象时需要序列化"
date:   2014-04-04 16:04:41 +0800
categories: php
---

前两天做东西，把对象存到session后，取出来怎么都读不出来，后来发现是session存储对象时，无法分别对象类型，所以将对象序列化成字符串，这有利于存储或传递 PHP的值，同时不丢失其类型和结构，然后取出来反序列化才可以，再此mark一下