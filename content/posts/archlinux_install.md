---
title: Arch Linux 入坑指南
date: 2018-12-16
categories:
  - linux
tags:
  - archlinux
---

Arch Linux 安装指南
<!--more-->

## 系统安装
### 连接网络
```
wifi-menu
```
### 磁盘分区
```bash
fdisk -l
fdisk /dev/sda
# 1. 先将硬盘格式转换为 GPT
# 2. 创建 sda1 EFI 分区 200M
# 3. 创建 ada2 boot 分区 1G
# 4. 剩下的 sda3 Linux LVM 分区
```
```bash
# 格式化三个分区
mkfs.vfat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
mkfs.xfs /dev/sda3
```
### 查看硬盘的分区情况
```
lsblk -l
```
### 创建并挂载目录
```bash
# 一定要先挂载 /mnt
mount /dev/sda3 /mnt
# 创建 boot 文件夹并挂载
mkdir /mnt/boot
mount /dev/sda2 /mnt/boot
# 创建 efi 文件夹并挂载
mkdir /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi

```
### 安装基本系统
```
pacstrap /mnt base
```
### 生成 fstab
```
genfstab -U /mnt >> /mnt/etc/fstab
```
### Change root 到新安装的系统
```
arch-chroot /mnt
pacman -S vim net-tools
```
### Locale
```bash
# 编辑地区信息生成文件
vim /etc/locale.gen
```
```bash
# 生成地区信息
locale-gen
# 写进配置文件
echo LANG=zh_CN.UTF-8 > /etc/locale.conf
```
### 设置时区
```
ln -S /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --localtime 
```
### 主机名
```
echo myhostname > /etc/hostname
vim /etc/hosts
```
### Root 密码
```
passwd
```
### 安装 GRUB
```bash
pacman -S grub-efi-x86_64
pacman -S efibootmgr
grub-install --efi-directory=/boot/efi --bootloader-id=arch --recheck
grub-mkconfig -o /boot/grub/grub.cfg
```
### 重启
```
exit
umount /mnt/boot/efi
umount /mnt/boot
umount /mnt
reboot
```

## 安装图形化界面
### 设置 archlinux 源
```
vim /etc/pacman.d/mirrorlist
```
### 更新源并且安装yaourt:
```bash
pacman -Syu yaourt
```
### 安装驱动
+ 触摸板驱动

```bash
pacman -S xf86-input-libinput xf86-input-synaptics
```

+ 显示驱动

```
pacman -S xf86-video-intel
pacman -S mesa-libgl libva-intel-driver libvdpau-va-glmesa-demos
pacman -S nvidia nvidia-settings
```

+ 声音

```
pacman -S alsa-utils
```

+ 无线网卡安装BCM4322

```
pacman -S linux-firmware
pacman -S linux-headers
yaourt -S broadcom-wl-dkms
```

+ 安装Deepin桌面环境

1.安装Xorg和其工具包
```
pacman -S xorg-server xorg-xinit
```

2.DDE基本环境安装
```
pacman -S deepin deepin-extra lightdm
pacman -S file-roller evince gedit thunderbird gpicview
pacman -S unrar unzip p7zip
```

3.修改lightdm.conf使用登录管理器
```
/etc/lightdm/lightdm.conf
greeter-session=lightdm-deepin-greeter
systemctl enable lightdm
```

4.配置网络连接
```
pacman -S networkmanager
systemctl enable NetworkManager
```

+ 安装常用软件
1.搜狗输入法安装
```
pacman -S fcitx fcitx-configtool
pacman -S fcitx-sougoupinyin
```
在用户目录下添加个配置文件：
```
nano ~/.xprofile
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```
```
pacman -S opencc  （简体和繁体互相转换的库）
```

2.浏览器
```
pacman -S chromium
pacman -S google-chrome
pacman -S firefox
pacman -S git
```
