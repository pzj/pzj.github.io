---
layout: post
title: Shell技巧
category: 技术
tags: archlinux
description: Shell技巧
---

sudo -s <shell>：切换到root用户的shell
可以不加 <shell>，会使用默认 shell
用户必须有相应shell命令的sudo权限，例如 /usr/bin/bash
一旦切换成功，用户可以以root身份执行任何命令

普通用户使用 sudo -s /usr/bin/bash 命令切换到 root 的shell，然后修改root用户的密码

sudo -- sh -c 'whoami; whoami'

source命令执行脚本是在当前Shell执行，不会像`bash http://script.sh`或`./script.sh`那样新建一个子Shell执行，所以不必export输出变量。因此source常用来重新加载修改过的配置文件。