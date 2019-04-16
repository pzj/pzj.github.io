---
layout: post
title: ArchLinux安装
category: 技术
tags: archlinux
description: ArchLinux安装
---

Archlinux是最好的Linx发行版，下面是我在mac上Parallels Desktop安装ArchLinux的过程。

# 准备

1. [下载iso镜像文件](https://www.archlinux.org/download/)
2. [官方安装文档(中文)](https://wiki.archlinux.org/index.php/Installation_guide_%28%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%29)
3. 安装Vmware

# 安装

打开vwamre加载镜像，创建虚拟机
![cover](image/1.jpg)

添加--force参数就好了
grub-install --force /dev/sda

查看目前分区情况
# fdisk -l

# cfdisk /dev/sda
将硬盘分为三个区 sda1 sda2 sda3

格式化分区
mkfs.ext4 /dev/sda1
mkfs.ext4 /dev/sda2
mkfs.ext4 /dev/sda3

# grub-install /dev/sda
/usr/sbin/grub-setup: warn: This GPT partition label has no BIOS Boot Partition; embedding won't be possible!.
/usr/sbin/grub-setup: warn: Embedding is not possible.  GRUB can only be installed in this setup by using blocklists.  However, blocklists are UNRELIABLE and their use is discouraged..
/usr/sbin/grub-setup: error: will not proceed with blocklists.

The problem is because I set the wrong flags for bios_grub partition:


#  parted /dev/sda unit s print
Model: ATA QEMU HARDDISK (scsi)
Disk /dev/sda: 20971520s
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End        Size       File system  Name       Flags
 1      34s     97656s     97623s     ext2         bios_grub  boot
 2      98304s  20969471s  20871168s  ext4         rootfs

Flags need set to bios_grub instead of boot:

# parted /dev/sda set 1 bios_grub on
Information: You may need to update /etc/fstab.                           

# partprobe

#  parted /dev/sda unit s print
Model: ATA QEMU HARDDISK (scsi)
Disk /dev/sda: 20971520s
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End        Size       File system  Name       Flags
 1      34s     97656s     97623s     ext2         bios_grub  bios_grub
 2      98304s  20969471s  20871168s  ext4         rootfs

# grub-install /dev/sda
Installation finished. No error reported.

http://edward-zhu.github.io/special/os_exp/2015/01/02/exp-1.2.html

https://blog.yoitsu.moe/arch-linux/installing_arch_linux_for_complete_newbies.html

root@archiso ~ # genfstab -U /mnt >> /mnt/etc/fstab
root@archiso ~ # arch-chroot /mnt /bin/bash

# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 要


# mkinitcpio -p linux 不要

# UEFI 用户这么做：

# grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub --recheck

pacstrap /mnt base base-devel iw dialog wpa_supplicant wpa_actiond
# pacman -S networkmanager
当然还有 NetworkManager：
# systemctl enable NetworkManager

添加--force参数就好了
grub-install --force /dev/sda

新建一个用户

-m 为新用户创建一个文件夹，-s 设置用户的登录 Shell

记得最后是用户名就好 😂

# useradd -m -s /bin/bash horo

然后设置密码

# passwd horo

net-tools openssh binutils

18878
17610

visudo

yaourt 只能非root用户执行

git clone https://aur.archlinux.org/package-query.git

git clone https://aur.archlinux.org/yaourt.git

makepkg -si

yaourt unsupported package potentially dangerous
/Users/pengzhangjie/Documents


pacman -Q archlinux-keyring
pacman -Syu haveged
systemctl start haveged
systemctl enable havegedsudo 

rm -fr /etc/pacman.d/gnupg
pacman-key --init
pacman-key --populate archlinux
https://wiki.archlinux.org/index.php/Pacman/Package_signing_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)

# 总结

# 附:pacman命令
genpac -c ~/.pac/config.ini

命令 | 解释
---- | ---
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


sudo pacman -Syy && sudo pacman -S archlinuxcn-keyring

sudo pacman -Sy net-tools
sudo pacman -Sy sysstat