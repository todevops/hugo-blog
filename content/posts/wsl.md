---
title: Windows LinuxSubSystem
date: 2018-11-19
categories:
  - tools
tags:
  - windows
  - wsl
---

Windows Linux 子系统安装指南
<!--more-->

## 安装 [oh-my-zsh](https://ohmyz.sh/)
+ curl
```shell
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

+ wget
```shell
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

## 更改默认 shell
```shell
chsh -s $(which zsh)
```

## 更改主题
```shell
vim ~/.zshrc
```
![](images/zshrc.png)
```conf
ZSH_THEME="agnoster"
DEFAULT_USER="jeremy"
```

## 问题
>字体问题导致显示异常

![](images/zshlostfonts.png)

解决办法：Windows 环境安装 powerline-fonts 并设置控制台字体
https://github.com/powerline/fonts.git

![](images/setpowerlinefonts.png)


>启动vim的时候会出现字体变回原来的新宋体的情况：

win + R -> 运行 -> ```regedit```
```
HKEY_CURRENT_USER\Console\C:_Program Files_WindowsApps_CanonicalGroupLimited.Ubuntu18.04onWindows_1804.2018.817.0_x64__79rhkp1fndgsc_ubuntu1804.exe
```
添加：CodePage (DWORD (32) 类型、值 0x01b5)

![](images/regedit.png)