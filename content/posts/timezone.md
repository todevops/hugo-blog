---
title: Linux 时区设置
date: 2018-12-04
categories:
  - linux
tags:
  - linux
  - shell
---

Linux 时区设置
<!--more-->

## 1. 时间设置
### 查看、设置硬件时间
```bash
hwclock --show
clock --show
## 月/日/年时:分:秒
hwclock --set --date="06/18/14 14:55"
clock --set --date="06/18/14 14:55"
```

### 硬件时钟与系统时钟同步
```bash
## hc代表硬件时间，sys代表系统时间
## 硬件时钟同步系统时钟
hwclock --hctosys
clock --hctosys
## 系统时钟同步硬件时钟
hwclock --systohc
clock --systohc
```

### 同步 ntp 服务器
```bash
## 安装 ntp、ntpdate
yum -y install ntp
ntpdate <IP>
## 添加定时任务
crontab -e
0 0 * * * /usr/sbin/ntpdate <IP>
```

## 2. 时区设置
### tzselect
```bash
TZ='Asia/Shanghai'; export TZ
```

### 修改配置文件
```
vim etc/sysconfig/clock
ZONE=Asia/Shanghai
rm /etc/localtime
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
```shell
timedatectl
timedatectl list-timezones
timedatectl set-timezone Asia/Shanghai
yum -y install ntp
vim  /etc/ntp.conf
```