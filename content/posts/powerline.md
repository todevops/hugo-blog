---
title: 使用 Powerline 美化终端
date: 2019-01-29
categories:
  - tools
tags:
  - linux
  - powerline
---

使用 Powerline 美化终端
<!--more-->

## 安装 python3
Ubuntu
```bash
[root@ubuntu ~]# sudo apt install python3 python3-pip
```

## 安装 powerline
```
[root@ubuntu ~]# python3 -m pip install powerline-status
[root@ubuntu ~]# python3 -m pip show powerline-status
Name: powerline-status
Version: 2.7
Summary: The ultimate statusline/prompt utility.
Home-page: https://github.com/powerline/powerline
Author: Kim Silkebaekken
Author-email: kim.silkebaekken+vim@gmail.com
License: MIT
Location: /usr/local/lib/python3.6/dist-packages
Requires:
```

## 设置 bash
```bash
[root@ubuntu ~]# vim ~/.bashrc
```
```conf
powerline-daemon -q
POWERLINE_BASH_CONTINUATION=1
POWERLINE_BASH_SELECT=1
. /usr/local/lib/python3.6/dist-packages/powerline/bindings/bash/powerline.sh
```

+ 效果图
{% asset_img powerline-status-bash.png powerline-status-bash %}

## 设置 vim
```bash
[root@ubuntu ~]# vim ~/.vimrc
```
```conf
set rtp+=/usr/local/lib/python3.6/dist-packages/powerline/bindings/vim/
set laststatus=2
set t_Co=256
```

+ 效果图
{% asset_img powerline-status-vim.png powerline-status-vim %}

