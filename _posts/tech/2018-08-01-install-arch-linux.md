---
layout: post
title: ArchLinux安装
category: 技术
tags: archlinux
description: ArchLinux安装
---

Archlinux是最好的Linx发行版，下面是我在mac上Parallels Desktop安装ArchLinux的过程。

## 准备

1. [下载iso镜像文件](https://www.archlinux.org/download/)
2. [官方安装文档(中文)](https://wiki.archlinux.org/index.php/Installation_guide_%28%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%29)
3. 安装Vmware

## 启动虚拟机

打开vwamre加载镜像，创建虚拟机，注意配置硬盘大小。

查看目前分区情况
```
# fdisk -l
```

使用cfdisk进行分区
```
# cfdisk /dev/sda
```
将硬盘分为三个区 sda1 sda2 sda3
其中sda1为系统安装分区，sda2为用户home分区(建议设置至少占总容量50%以上)，sda3为交换分区

格式化分区
```
# mkfs.ext4 /dev/sda1
# mkfs.ext4 /dev/sda2
# mkswap /dev/sda3
```

挂载分区
共分3个区：

| 类型 | 大小 | 挂载点
|---|---|---|
|ext4|4GB|/|
|ext4|5GB|/home|
|swap|1GB|-|
```
# mount /dev/sda1 /mnt
# mkdir /mnt/home
# mount /dev/sda2 /mnt/home
# swapon /dev/sda3
```

下载archlinux基础包

先配置国内的镜像源
```
用 vim 打开 /etc/pacman.d/mirrorlist

https://www.archlinux.org/mirrorlist/all/
## China
#Server = http://mirrors.163.com/archlinux/$repo/os/$arch
#Server = http://mirror.lzu.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.neusoft.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.neusoft.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.shu.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.shu.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.shu6.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.sjtug.sjtu.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.xjtu.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.zju.edu.cn/archlinux/$repo/os/$arch
然后用 pacman -Syy 刷新一下软件包数据库
```
```
# pacstrap /mnt base base-devel  
```
生成fstab表
```
# genfstab -p /mnt >> /mnt/etc/fstab
```
切换根目录至新系统
```
# arch-chroot /mnt
```

## 初始设置
选择安装常用软件包
```
 pacman -S sudo coreutils dnsutils  vim file sed awk sysstat net-tools openssh binutils git networkmanager
 自动启动网络服务与ssh
 systemctl enable NetworkManager
 systemctl enable sshd
  
 vim /etc/ssh/sshd_config 
 在最后一行增加
 PermitRootLogin yes
 关机
 systemctl poweroff
 重启
 systemctl reboot
```
设置主机名
```
# echo pzhjie-arch > /etc/hostname
```
设置时区
```
# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

设置时间标准 为 UTC，并调整 时间漂移
```
# hwclock --systohc --utc
```


在/etc/locale.gen中反注释需要的locales，然后运行命令生成locales
```
# locale-gen
```
设置locale偏好
```
# echo LANG=en_US.UTF-8 > /etc/locale.conf
```
创建初始RAM disk
```
# mkinitcpio -p linux
```
创建root密码
```
# passwd
```

##  安装引导器

下载并安装grub
```
# pacman -S grub
```

安装grub至硬盘
```
# grub-install --recheck /dev/sda
```
报错:/usr/sbin/grub-setup: error: will not proceed with blocklists. 则添加"--force"参数就好了
```
# grub-install --force /dev/sda
```
创建配置文件
```
# grub-mkconfig -o /boot/grub/grub.cfg
```

## 重启至新系统
返回安装环境
```
# exit
```
重新启动
```
# reboot
```

# 参考链接

1.  [OS Experiment Special](http://edward-zhu.github.io/special/os_exp/2015/01/02/exp-1.2.html)

2.  [给 GNU/Linux 萌新的 Arch Linux 安装指南](https://blog.yoitsu.moe/arch-linux/installing_arch_linux_for_complete_newbies.html)

# 总结

……

# 附:pacman命令

命令 | 解释
---- | ---
pacman -Syy  | 仅更新源
pacman -Sy abc | 和源同步后安装名为abc的包
pacman -S abc |  从本地数据库中得到abc的信息，下载安装abc包
pacman -Sf abc |  #强制安装包abc
pacman -Ss abc |搜索有关abc信息的包
pacman -Si abc |从数据库中搜索包abc的信息
pacman -Q | 列出已经安装的软件包
pacman -Q abc | 检查 abc 软件包是否已经安装
pacman -Qi abc |列出已安装的包abc的详细信息
pacman -Ql abc | 列出abc软件包的所有文件
pacman -Qo /path/to/abc | 列出abc文件所属的软件包
pacman -Syu |同步源，并更新系统
pacman -Sy |仅同步源
pacman -Su |更新系统
pacman -R abc |删除abc包
pacman -Rd abc |强制删除被依赖的包
pacman -Rc abc |删除abc包和依赖abc的包
pacman -Rsc abc |删除abc包和abc依赖的包
pacman -Sc |清理/var/cache/pacman/pkg目录下的旧包
pacman -Scc |清除所有下载的包和数据库
pacman -U abc |安装下载的abs包，或新编译的abc包
pacman -Sd abc |忽略依赖性问题，安装包abc
pacman -Su --ignore foo |升级时不升级包foo
pacman -Sg abc |查询abc这个包组包含的软件包
pacman -R $(pacman -Qdtq) |清除无用的包

downgrade 