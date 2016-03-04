---
layout: post
title:  "validates_numericality_of的使用细节"
date:   2015-09-21 16:04:41 +0800
categories: ruby
---

validates_numericality_of是activerecord中校验数字相关的方法。
如:equal_to,:less_than,:less_than_or_equal_to等，但是在使用中发现了一些细节：

例如现在有一个Product类，其price是整型，校验其价格必须小于或等于1
{% highlight ruby %}
# encoding: utf-8
  class Product < ActiveRecord::Base
    validates_numericality_of :price, less_than_or_equal_to=1
  end
{% endhighlight %}

{% highlight ruby %}
  product = Product.new
  product.price = 1.5
  p product.price #输出1
  product.save
{% endhighlight %}

由于price字段是整型，所以ActiveRecord会自动帮我们把1.5这个浮点转换成整型1，那么现在save会怎样呢？结果是校验不通过，为什么？看看源码：
源码在~/.rvm/gems/ruby-1.9.3-p125/gems/activemodel-3.2.21/lib/active_model/validations/numericality.rb

{% highlight ruby %}
def validate_each(record, attr_name, value)
  before_type_cast = "#{attr_name}_before_type_cast"
  raw_value = record.send(before_type_cast) if record.respond_to?(before_type_cast.to_sym)
  .....
end
{% endhighlight %}

原因是validate的时候会使用类型转换前的的值来比较，即一开始赋值的1.5，所以校验会不通过