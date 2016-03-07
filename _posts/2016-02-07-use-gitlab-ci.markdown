---
layout: post
title:  "配置 gitlab-ci 进行持续集成"
date:   2016-02-07 16:04:41 +0800
categories: ruby
---

在日常工作中，我们经常会写一些单元测试来确保程序可以正确执行，但是写完单元测试后常常会面临这些问题：

有一些小改动懒得跑单元测试
别人改动相关代码影响了你的代码逻辑，但他们不知道你写过这些单元测试或者没有跑单元测试的习惯
项目大了，单元测试很多，跑一遍要花很长时间
这时候可以选择进行持续集成

进行持续集成有很多种方式，介绍一下使用gitlab-ci进行持续集成的配置方式。

准备

gitlab版本升级的很快，公司的gitlab是在去年5月份安装的，当时的版本还是7.10，现在已然发布到8.3了。
建议把gitlab升级到8.0以上，8.0以上的版本自动集成了gitlab-ci的功能，无需再自己配置一个gitlab-ci-server了。google gitlabci相关的内容很多都是基于8.0以上介绍的。并且gitlab7.12以上版本支持使用.gitlab-ci.yml进行ci配置，自己搭建gitlab-ci的话很容易出现ci版本和gitlab不一致导致找不到.gitlab-ci.yml的问题。
升级方式就不详细说了，详见https://about.gitlab.com/update/
公司之前用的Omnibus gitlab，升级很方便，直接yum install就ok了，升级前记得备份数据库。

gitlab开启build功能

gitlab项目下-》Project-Settings-》Features里的Builds选项勾上-》保存-》完事儿
开启后会发现菜单多了几个功能，打开runners，里面有token和url，下面配置runner时会用到。

配置ci-runner

首先安装gitlab-ci-multi-runner，之前有一个runner是gitlab-ci-runner，这个runner的代码已经不再维护了，不要装错。
安装完成后启动：gitlab-ci-multi-runner start
注册一个新的runner： gitlab-cimulti-runner register ，提示输入url和token，输入上一步runners里的内容就行了，runner的名字随便输入。然后提示选择executer，个人选的shell，还可以选择docker和ssh
完成后刷新gitlab的runner页面，可以看到刚才注册的runner
配置gitlab-workhouse

之前公司没有使用gitlab内置的nginx，而是自己起的nginx做配置。这种情况可能导致build的时候git clone不下代码，会报这个错误：
fatal: reference is not a tree
google了一下，建议的解决办法就是使用gitlab内置的一个gitlab_git_http_server，好像是8.2之后的版本改名成了gitlab-workhouse。
/etc/gitlab/gitlab.rb中增加配置：
{% highlight ruby %}
  gitlab_workhorse['listen_network'] = "tcp"
  gitlab_workhorse['listen_addr'] = "127.0.0.1:8181"
{% endhighlight %}
nginx增加配置：
{% highlight ruby %}
  upstream gitlab_repo {
     server 127.0.0.1:8181 fail_timeout=0;
     server 127.0.0.1:8181 fail_timeout=0;
  }
  #gitlab的server配置中增加
  location ~* \.(git) {
      proxy_read_timeout      300;
      proxy_connect_timeout   300;
      proxy_redirect          off;

      proxy_set_header   Host              $http_host;
      proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
      proxy_pass http://gitlab_repo;
  }
{% endhighlight %}
###.gitlab-ci.yml配置
在项目的根目录下创建.gitlab-ci.yml，下面是跑单元测试的配置，当然还可以干很多别的。
{% highlight ruby %}
before_script:
    - cp config/database.yml.sample config/database.yml
    - bundle install --path vendor/bundle --without development
    - "bundle exec rake db:create RAILS_ENV=test"
    - "bundle exec rake db:migrate RAILS_ENV=test"
  job1:
    script: 'RAILS_ENV=test rake test:units'
{% endhighlight %}
进阶配置请参考文档

到此配置完成，从此之后每一个push都会触发ci进行build，提交Merge quest后也会有相关build结果提示，再也不用担心小伙伴们忘跑单元测试了。

##TODO
其实不需要每个push都触发build的，希望只在提交MR后触发一次build，在和代码之前看到build结果就可以了。还在研究，希望有了解的同学能指点一下。