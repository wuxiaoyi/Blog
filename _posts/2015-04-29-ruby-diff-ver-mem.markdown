---
layout: post
title:  "ruby版本不同占用的内存变化"
date:   2015-04-29 16:04:41 +0800
categories: ruby
---

今天领导分享了一个gem：<a href="https://github.com/schneems/derailed_benchmarks">derailed_benchmarks</a>

安装之后能够清晰的看到gem，字符串对象的内存分配和占用。根据gem的内存占用情况，就可以考虑是否还要用这个gem，有没有其他的方案来代替。

ruby会把内存大量消耗在对象创建上面，看到了一个<a href="https://github.com/rails/rails/pull/17658/files">rails改进</a>可以很好地说明这一点

activeRecord每次调用会为主键id创建一个字符串，ruby皆对象，所以每次创建一个字符串都是创建了一个对象，这个方法又是大量调用，所以这里用freeze作了改进，这个方法使得这个字符串不会被修改，类似于设置了一个常量，避免了每次去创建对象。

在实际项目中我们也可以注意这些问题，大量调用的方法可以使用类似的优化，比如字符串的downcase!这种，可能效果不会太明显，但是毕竟聊胜于无。