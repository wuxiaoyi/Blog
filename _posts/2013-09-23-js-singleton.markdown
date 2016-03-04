---
layout: post
title:  "javascript的单例实现"
date:   2013-09-23 16:04:41 +0800
categories: javascript
---
最近工作之余，打算做一个html5的rpg游戏，开发中遇到了一点问题，就是主角和地图实例化的问题，因为有多个js文件都需要主角，地图对象来进行操作，于是乎就要使用单例模式了，之前没用过js写单例，研究了一下，发现跟java的思想是差不多的。

上代码：

{% highlight javascript %}
var Player = (function(){
  function playerSingleton(){
    //欲实现的方法，属性
  }
  var instance;
  var _static = {
    getInstance : function(){
      if(instance === undefined){
        instance = new playerSingleton();
      }
      return instance;
    }
  }
  return _static;
})();
{% endhighlight %}

使用Player.getInstance()来获取player对象，就可以保证单例了