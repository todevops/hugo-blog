---
title: Ubuntu 网络配置
date: 2018-11-23
categories:
  - linux
tags:
  - linux
  - ubuntu
---

Ubuntu Server 网络配置
<!--more-->

## 1. 查看所有网卡信息
```
ifconfig -a
```
## 2. 修改网卡配置文件
```
vim /etc/network/interfaces
```
+ DHCP

```bash
source /etc/network/interfaces.d/*

auto lo
iface lo inet loopback

auto ens33
iface ens33 inet dhcp
```

+ STATIC

```bash
source /etc/network/interfaces.d/*

auto lo
iface lo inet loopback

auto ens33
iface ens33 inet static
# 克隆 mac 地址
# pre-up ifconfig ens33 hw ether <MACADDR> 
address <IPADDR>
gateway <GATEWAY>
netmask <NETMASK>
# network <NETWORK>
# broadcast <BROADCAST>
dns-nameservers <DNS>
```