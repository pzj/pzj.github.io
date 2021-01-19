---
layout: post
title: shell技巧
category: 
tags: 
description: 
---

linux后台运行 
& 要是关闭终端那么脚本也停了，
加nohup  既使把终端关了，脚本也会跑，是在服务器那运行的。 

脚本名称叫test.sh 入参三个: 1 2 3
运行test.sh 1 2 3后
$*为"1 2 3"（一起被引号包住）
$@为"1" "2" "3"（分别被包住）
$#为3（参数数量）

Makefile有三个非常有用的变量。分别是$@，$^，$<代表的意义分别是：
$@--目标文件，$^--所有的依赖文件，$<--第一个依赖文件

sudo -s <shell>：切换到root用户的shell
可以不加 <shell>，会使用默认 shell
用户必须有相应shell命令的sudo权限，例如 /usr/bin/bash
一旦切换成功，用户可以以root身份执行任何命令

普通用户使用 sudo -s /usr/bin/bash 命令切换到 root 的shell，然后修改root用户的密码

sudo -- sh -c 'whoami; whoami'

source命令执行脚本是在当前Shell执行，不会像`bash http://script.sh`或`./script.sh`那样新建一个子Shell执行，所以不必export输出变量。因此source常用来重新加载修改过的配置文件
