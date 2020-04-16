---
title: 树莓派安装教程
date: 2018-11-26
categories:
  - linux
tags:
  - linux
  - raspberrypi
---

树莓派安装教程
<!--more-->

## rasbian 网络配置
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

auto wlan0
allow-hotplug wlan0
iface wlan0 inet dhcp
wpa-ssid "360WiFi"
wpa-psk "hellworld"
```

## 树莓派安装 docker-ce
```
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

echo "deb [arch=armhf] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list

apt update

apt install docker-ce
```