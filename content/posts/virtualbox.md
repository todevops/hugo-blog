---
title: VVirtualBox EFI 引导问题
date: 2018-11-21
categories:
  - tools
tags:
  - virtualbox
---

VirtualBox EFI 引导问题
<!--more-->

## 1、引导界面修改
```
Shell> FS0:
FS0:\> cd EFI
FS0:\EFI> cp centos\grubx64.efi BOOT\grubx64.efi
```

## 2、系统界面修改
```
cp /boot/efi/EFI/centos/grubx64.efi /boot/efi/EFI/BOOT/grubx64.efi
```