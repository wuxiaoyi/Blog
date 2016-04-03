---
layout: post
title:  "关于乐观锁的重试"
date:   2016-03-20 16:04:41 +0800
categories: ruby
---


### 关于乐观锁的重试

锁相关的知识可以参考 @zamia 的这篇文章 <https://ruby-china.org/topics/28963>

这篇文章算是一点小小的补充

### 如何本地复现乐观锁冲突

在访问量大的线上环境，使用了乐观锁的地方是有可能出现StaleObjectError的异常的，本地如何复现呢？这个有个小方法（可能比较low）

1. 使用byebug这个gem，在锁的事务中加上byebug调试，例：
 
  假设这是一个库存的对象，库存比较容易出现抢锁的情况
  
  {% highlight ruby %}
    def unforzen
      self.with_lock do
        byebug
        self.frozen_num -= 1
        self.save!
      end
    end
  {% endhighlight %}
2. 开两个console，获取两个对象（stock_a和stock_b）
  
 - 调用stock_a.unforzen，会进入到如下调试模式，可以看到已经进入到事务中了：

  ![](https://ruby-china-files.b0.upaiyun.com/photo/2016/116ce53845c1853ffd5b0901b3098f96.png)

 - 然后调用stock_b.unforzen，跟a相同，也进入到了事务，这时将a的代码执行完，next是单步执行，continue是执行到下一个断点。
    
 - 执行b的代码，这时你会发现b的unforzen方法抛出了 StaleObjectError的异常


###准备工作结束，如何重试


值得注意的一点是，reload需要给参数lock: true，否则lock_version是不会更新的

{% highlight ruby %}
def unforzen
  optimistic_lock_retry_times = 3
  begin
    self.with_lock do
      byebug
      self.frozen_num -= 1
      self.save!
    end
  rescue ActiveRecord::StaleObjectError => e
        optimistic_lock_retry_times -= 1
        if optimistic_lock_retry_times > 0
          self.reload(lock: true)
          retry
        else
          raise e
        end
    rescue => e
      raise e
    end
end

{% endhighlight %}

    
### 一点疑问

其实实际的代码要比例子复杂一点，这个释放库存的功能是通过一个service调用的，异常统一在service中处理，但是service不能直接获取到出现锁异常的record，我尝试过error.record.reload(lock: true)，如下：

{% highlight ruby %}
rescue ActiveRecord::StaleObjectError => e
  p e.record.lock_version
  e.record.reload(lock: true)
  p e.record.lock_version
  retry
end
{% endhighlight %}

发现lock_version是变了的，但是重新retry的代码中的record的lock_version还是原来的。这块还没仔细研究原理，懂得同学也可以分享一下~