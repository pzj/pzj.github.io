---
layout: post
title: Linux性能分析
category: 资源
tags: 性能
description: Linux性能分析
---

### 1. top

- 指定显示进程 9836，每隔 5 秒的进程的资源的占用情况

```
# top –d 5 –p 9836 -c
```