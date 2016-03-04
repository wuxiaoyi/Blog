---
layout: post
title:  "使用devise时，登录失败devise只抛出401"
date:   2015-03-07 16:04:41 +0800
categories: ruby
---


接触ruby后，第一次使用devise，在处理登录失败错误的时候，发现devise并没有抛出相关的error_message，只是抛出了401错误，所以前端使用devise的resource.errors.full_messages方法是并没有错误输出。

小研究了一下，发现登录错误是通过rails抛出的。输出falsh的时候发现包含了登录失败的message。而例如注册时用户名被占用是在devise的error_message中，因为注册时devise会先查询一下该用户名，如果被占用了，就会返回devise的错误提示。

解决方法：

在applicationHelper里加了个方法处理flash中的错误，方法只处理了alert，如果有需要的话也可以加上notice：

{% highlight ruby %}
def notice_message
  flash_messages = []

  flash.each do |type, message|
    if type.to_sym == :alert
      text = content_tag(:small, message, :class =&gt; "error")
      flash_messages &lt;&lt; text if message
    end
  end

  flash_messages.join("\n").html_safe
end
{% endhighlight %}

而devise的错误直接用resource.errors.full_messages输出就可以了