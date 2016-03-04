---
layout: post
title:  "二级域名操作顶级域名的cookie"
date:   2015-04-16 16:04:41 +0800
categories: ruby
---

开发中遇到一个二级域名需要删除顶级域名下cookie的问题。

正常删除cookie的方式是：
{% highlight ruby %}
  cookies.delete :key
{% endhighlight %}

这样操作只针对当前域名的cookie，想要删除顶级域名的cookie需要这样：

{% highlight ruby %}
  //假设当前域名是abc.qwe.com
  cookies.delete(:key, :domain => 'qwe.com')
{% endhighlight %}