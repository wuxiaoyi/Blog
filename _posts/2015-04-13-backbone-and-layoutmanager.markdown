---
layout: post
title:  "backbone+backboneLayoutManager使用心得"
date:   2015-04-13 16:04:41 +0800
categories: javascript
---

backbone+backboneLayoutManager使用了有一段时间了，backbone的基础就不说了，主要写一写自己使用时遇到的问题。
众所周知，backbone是一个偏轻量级的前端mvc框架，虽然提供了不少灵活的特性，但是在实际开发中有时会希望它能够提供更多的api来避免重复的工作。
backboneLayoutManager就是这样的一个插件。

layoutManager文档：https://github.com/tbranyen/backbone.layoutmanager/wiki

项目引入backboneLayoutManager后，在声明View的class时，加上manage:true就可以使用layoutmanager了，如下

{% highlight javascript %}
class @xxxxx extends Backbone.View
  manage: true
  el: '#test'
  template: JST['templates/test']
{% endhighlight %}

1.render
在仅有backbone的情况下，每个view都必须重写render方法来控制元素或子view插入到哪里，但根据backbone的思想来说，当前view的元素或者子view都是应该在该容器内来操作的，所以layoutManager会默认把元素append到当前容器内，只需要直接调用render方法即可，不必重写render。
但是实际开发中也不可避免的遇到不想把子元素插入到当前容器内的情况，这时可以重写layoutManager的insert方法。

{% highlight javascript %}
insert: ($root, $el) ->
  if $root.selector.indexOf(".wrapper") > 0
    $(".wrapper").before($el)
  else
    $root.append($el)
{% endhighlight %}

这是重写的insert方法，第一个参数时父元素，new子view的时候不传默认为父容器的el，第二个为插入的元素。这个重写就是判断了传进来的$root如果超出了父容器，就放到$(".wrapper")前，没有的话默认append。如果在没重写insert并且传入的root超过了父容器，元素是添加不上的。
2.template
View在指定了template后，新建该View的对象会自动render配置的模板，用代码举个例子。

{% highlight javascript %}
//子view
class @ChildrenView extends Backbone.View
  manage: true
  template: JST['templates/test']

//父view
class @ParentView extends Backbone.View
  el: '.photo-stream'
  manage: true
  initialize: ->
    @render()
  beforeRender: ->
    @insertView new ChildrenView
//Backbone.LayoutManager 会在每次 render 时清掉所有的子view。因此这里需要在每次 render 之前,加上对应的子 view
{% endhighlight %}

父view初始化时调用render，在beforeRender时会插入子view，子view自动渲染template。

3.sync
这个与layoutmanager无关，是backbone的model获取数据的问题。

{% highlight javascript %}
class @Notify extends Backbone.Model
  defaults:
    body: ""

class @NotifyList extends Backbone.Collection
  model: Notify
  url: "#{notifies_path}.json"
{% endhighlight %}

backbone支持在collection中配置url来获取数据，但是只能通过get方式来获取，其实获取数据用get方式是符合规范的。但是不免有时需要使用post方法，或者需要在httpHeader中加入auth信息等。
这时可以重写backbone中collection的sync方法，如：

{% highlight javascript %}
sync: (method, model, options) ->
    data = {
      xx: 'xx',  
      yy: 'yy',
    }
    xhr = options.xhr = $.ajax
      url: "/xxxxxx/yyyyy.json"
      type: POST
      headers:
        'X-Authorize-Uid': 11
        'X-Authorize-Token': 22
        'Client-Id': '33'
      contentType: 'application/x-www-form-urlencoded; charset=UTF-8'
      data: data
    .done (response) =>
      //回调
    model.trigger('request', model, xhr, options);
    return xhr
{% endhighlight %}

这样在collection对象使用fetch方法时，就会调用你重写的方法了。跟源码中一样返回了一个ajax对象，为了在更上层的方法中可以继续做回调相关的操作。

4.getView
个人感觉layoutManager提供了一个非常好用的方法getView({model: xxxx})，在前端交互时，经常会遇到用过model来操作数据，然后再反馈给view做页面上的变化，以前往往需要得到model在collection的下标，再去view中查找该子view进行操作，有了个这个getView方法后就不用了，这个方法可以通过你操作的model直接获取到该model所属的子view，方便了很多。

就先这么多，以后再更新