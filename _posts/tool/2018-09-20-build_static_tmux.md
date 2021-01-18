---
layout: post
title: 编译静态tmux
category: 工具
tags: [tmux, Linux]
description: 编译静态tmux
---

### 起因

发现我们公司的跳板机上没有tmux，但是跳板机上的权限很受限，没有root权限，使用其它机机器编译的binary，发现各种依赖的动态库问题，选择静态编译一个tmux程序。

## 过程
tmux依赖libevent,先下载最新版的libevent

[libevent](http://libevent.org/)

```
cd libevent-2.0.22
./configure --prefix=/home/pengzhangjie/temp/
make
make install
```

下载tmux

[tmux](https://github.com/tmux/tmux)


```
cd tmux-2.0
./configure --prefix=/home/iceway/temp/ --enable-static CFLAGS=-I/home/iceway/temp/include LDFLAGS=-L/home/pengzhangjie/temp/lib64
make
make install
```

## 结束
