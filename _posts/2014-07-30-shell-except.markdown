---
layout: post
title:  "linux下shell脚本自动输入--expect"
date:   2014-07-30 16:04:41 +0800
categories: linux
---


最近日常开发中遇到了这个问题，程序生成的一个文件需要同步到多台服务器，而跑phpfpm进程的用户跟欲同步目录的所有者不一样，所以用sshkeygen也没法做到。于是写了这个脚本。
{% highlight shell %}
  #!/usr/bin/expect
  set timeout 30
  spawn scp /www/file1 user1@xx.xx.xx.xx:/home/user1/files
  expect "*password*"
  send "123123\r"
  expect EOF
{% endhighlight %}

1.#!/usr/bin/expect，首页要引入expect，否则无法使用

2.设置超时时间

3.必须使用spawn进行命令操作，否则expect无法捕捉到

4.最爽的一步，用expect捕捉接下来出现的命令提示，一般都是请输入密码之类的文字，根据实际情况来定义

5.输入你的密码，\r是回车的意思

6.注意要使用expect EOF结束，否则你会发现命令不会执行成功

<strong>更新：发现其实在目标服务器用户下加个authorized_keys就可以了，比这个安全也简单。</strong>