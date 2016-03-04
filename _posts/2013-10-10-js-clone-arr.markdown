---
layout: post
title:  "js数组克隆问题"
date:   2013-10-10 16:04:41 +0800
categories: javascript
---

前两天工作中遇到这么一个问题，写了一个js图片插件，把页面的图片实体数组传到这个插件中进行预览，在使用插件中的删除功能后，回到页面总是发现页面的实体数组也被删除了。
于是知道了js的浅克隆和深克隆问题。

浅克隆：
concat()，slice()

就可以实现数组的浅克隆，但是面对数组中含有对象时，浅克隆依旧不好使，对象仍然是引用传递。
深克隆：
{% highlight javascript %}
function cloneObject(obj){
  var o = obj.constructor === Array ? [] : {};
  for(var i in obj){
    if(obj.hasOwnProperty(i)){
      o[i] = typeof obj[i] === "object" ? cloneObject(obj[i]) : obj[i];
    }
  }
  return o;
}
{% endhighlight %}