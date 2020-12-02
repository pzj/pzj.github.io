---
layout: post
title: 简化条件表达式
category: 技术
tags: 
description: 
---

    程序中经常要写条件式，有的条件逻辑十分复杂，使用如下技巧可以简化条件逻辑：

### 分解条件表达式
    - 将if段落提炼出来，各自构成一个独立函数
    - 将then和else段落提炼出来各自构成一个独立函数

### 回调
    - 根据回调函数不同采取不同的行为，移除if-else

### 利用多态代替条件表达式
    - 根据对象的不同类型而采取不同的行为

### case-switch语句
    - 预计后继会要增加条件判断，提前使用switch语句，避免深层if-else