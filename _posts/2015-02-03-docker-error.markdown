---
layout: post
title:  "使用docker时出现“Could not resolve 'archive.ubuntu.com'” 解决办法"
date:   2015-02-03 16:04:41 +0800
categories: docker
---

我在使用docker的时候遇到了这个问题，问题出现在dns无法解析这个域名。

google了一下，使用命令export DOCKER_OPTS="--dns 8.8.8.8 --dns 8.8.4.4"可以解决这个问题。

但实际上最好先把dns缓存清除，并且重启docker，把之前build到一半的image删除再重新build比较好，因为我一开始没有进行这些操作还是出现了无法解析的问题。

build的时候也可以使用--no-cache来从新build image。

另外，删除images的时候，先用docker images查看当前image，使用docker rmi imageid删除image，可能出现无法删除成功的情况，因为该image正在使用，需要先停止它的container，docker ps -a 查看运行的container，docker stop containerid