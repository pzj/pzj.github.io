添加--force参数就好了
grub-install --force /dev/sda


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