---
layout: post
title:  "mysql和monogodb的dump"
date:   2013-05-07 16:04:41 +0800
categories: database
---


备份mongo
mongodump -d public_link -o bak 备份public_link数据库到bak文件夹下

恢复mongo
mongorestore -d public_link bak/public_link 恢复public_link数据库

备份mysql

mysqldump -uroot -d testdb testdb.sql  加-d只导表结构，不加导全部数据

恢复mysql

进入mysql终端，选择或创建欲恢复数据库，执行source /xxx/xx/testdb.sql

另外，在mysql的数据迁移过程中，如果所有的表类型都是myisam的并且mysql版本不冲突的话，是直接可以copy使用的，mysql的数据文件通常存在/var/lib/mysql中