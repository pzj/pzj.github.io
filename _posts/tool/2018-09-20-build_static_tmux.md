---
title: 编译静态链接tmux
category: 工具
tags: [tmux]
---

### 起因


## 过程

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
