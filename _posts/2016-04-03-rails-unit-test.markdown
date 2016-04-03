---
layout: post
title:  "Rails如何写测试"
date:   2016-04-03 11:04:41 +0800
categories: ruby
---

本帖只是个人总结的一些实践经验，适用于一些不太会写测试的同学，高手请指教

### 为什么要写单元测试？

个人总结为以下两点：

  - 现在的测试是为了避免以后的麻烦。当一个功能比较复杂，关系到比较重要的业务或者难以进行黑盒测试的时候，或许在你开发的时候你很有信心，相信这里不会出现bug。但过了几个月甚至一年，你需要扩展功能或者别人需要接手你的代码时，这就会令人头疼了（尤其是公司没有测试工程师！）。

  - 依赖于第三方服务的功能，比如支付回调，支付平台退款。在本地环境无法以正确的数据调用第三方服务，这时候可以用测试来mock返回值，以便进行后续开发。

  - 节约回归测试的时间，每次回归测试都要跑的case可以写成单元测试

个人认为，如果开发时间充裕，尽可能的写测试覆盖全部功能。如果时间不够，比如在创业公司，测试最好也能够覆盖service和api


### miniTest+fixtures+mocha的测试框架

  - minTest: rails的测试框架
  - fixtures: 生成测试数据
  - mocha: 各种mock方法的补充

### 常用的测试方法

  详细可见 <http://ruby-doc.org/stdlib-2.1.0/libdoc/minitest/rdoc/MiniTest/Expectations.html>

  - 判断真假must_equal
    这是最常用的一个测试方法了。
    {% highlight ruby %}
      it 'model save must be true' do
        model.save.must_equal true
      end
      it 'service must return true' do
        service.execute.must_equal true
      end
    {% endhighlight %} 

  - 判断一个对象的类型must_be_kind_of
    {% highlight ruby %}
      it 'man must be kind of Human' do
        man = Human.new
        man.must_be_kind_of Human
      end
    {% endhighlight %} 

  - 判断一定会抛出异常must_raise
    {% highlight ruby %}
      it 'model must raise error if name is nil' do
        model.name = nil
        model.save.must_raise ValidationError
      end
    {% endhighlight %} 

### 常用的模拟方法

  - 模拟调用stubs

    在写测试的过程中，我们常常会希望某个方法返回我们希望的值，不管它如何执行的，这时可以用stubs。在下面这段代码中，我们需要测试可以成功申请支付宝退款，而实际代码中，申请支付宝退款是一个http请求，没有真实的订单号我们一定会申请失败，所以我们模拟一下它的返回。

    {% highlight ruby %}
      class AliPayDrawbackService
        
        def do_drawback(order)
          if apply(order)
            return order.update_attribute(:state, 1)
          else
            return false
          end
        end

        def apply(order)
          RestClient.post(url, order.id)
        end
      end
      it 'apply alipay drawback success' do
        #any_instance表示该service的任意实例对象
        AliPayDrawbackService.any_instance.stubs(:apply).returns(true)

        order = Order.first
        service = AliPayDrawbackService.new()
        service.do_drawback
        order.state.must_equal 1 #pass
      end
    {% endhighlight %} 

  - 期待调用expects

    某个功能在执行过程中会调用一个其他系统服务，或者某个功能会插入一个任务到异步队列。这是我们需要秉承一个原则：自己的功能自己测。即我不关心其他服务的功能是否正确，我认为只要我成功调用了就是正确的。

    {% highlight ruby %}
      #一个消息队列的pusher
      class Mq::Publisher
        def self.publish(key, msg = {})
          # push 消息体到队列
        end
      end
      class Order
        def submit
          # ...业务逻辑
          Mq::Publisher.puhlish('order.submit.success', {xxx})
        end
      end

      it 'send a message if order submit success' do
        #注意，期待调用的方法一定要写在实际调用前
        #这段代码表示期待Mq::Publisher的publish方法在本测试中至少调用一次，并且第一个参数是"order.submit.success"，any_parameters表示后面的可以是任意参数
        Mq::Publisher.expects(:publish)
        .with("order.submit.success", any_parameters)
        .at_least_once.returns(true)

        order = Order.new
        #xxx
        order.submit
        #如果Mq::Publisher没有调用publish，测试结果会是失败
      end
    {% endhighlight %} 

  - mock对象

    有时某个方法可能会需要一个很复杂的参数，或者某个方法返回的一个结果对象会影响剩余方法的执行，这时我们可以使用mock

    个人总结了两个方法来mock

  1.使用Minitest::Mock

    {% highlight ruby %}
      def execute
        service_a = ServiceA.new
        ret = service_a.do_something
        if ret.success?
          #xxxx
          return true
        else
          #xxxx
          return false
        end
      end

      it 'execute must return true if do_something' do
        #创建一个Mock对象，设置它的success?方法返回true
        mock = Minitest::Mock.new
        mock.expect :success?, true
        #设置ServiceA查到任意实例对象调用do_smoething方法返回mock
        ServiceA.any_instance.stubs(:do_smoething).returns(mock)

        execute.must_equal true
      end
    {% endhighlight %} 

  2.使用Class.new

    {% highlight ruby %}
      def execute
        service_a = ServiceA.new
        ret = service_a.do_something
        if ret.success?
          #xxxx
          return true
        else
          #xxxx
          return false
        end
      end
      it 'execute must return true if do_something' do
        mock = Class.new{
                define_method(:success?) { true }
              }.new
        ServiceA.any_instance.stubs(:do_smoething).returns(mock)

        execute.must_equal true
      end
    {% endhighlight %}     


##先总结这么多


