---
title: Linux 网络管理
date: 2018-12-05
categories:
  - linux
tags:
  - linux
  - network
---

Linux 网络管理
<!--more-->

## 1. ```arp```：管理系统中的 ARP 高速缓存
```
Usage:
  arp [-vn]  [<HW>] [-i <if>] [-a] [<hostname>]             <-Display ARP cache
  arp [-v]          [-i <if>] -d  <host> [pub]               <-Delete ARP entry
  arp [-vnD] [<HW>] [-i <if>] -f  [<filename>]            <-Add entry from file
  arp [-v]   [<HW>] [-i <if>] -s  <host> <hwaddr> [temp]            <-Add entry
  arp [-v]   [<HW>] [-i <if>] -Ds <host> <if> [netmask <nm>] pub          <-''-

        -a                       display (all) hosts in alternative (BSD) style
        -e                       display (all) hosts in default (Linux) style
        -s, --set                set a new ARP entry
        -d, --delete             delete a specified entry
        -v, --verbose            be verbose
        -n, --numeric            don't resolve names
        -i, --device             specify network interface (e.g. eth0)
        -D, --use-device         read <hwaddr> from given device
        -A, -p, --protocol       specify protocol family
        -f, --file               read new entries from file or from /etc/ethers

  <HW>=Use '-H <hw>' to specify hardware address type. Default: ether
  List of possible hardware types (which support ARP):
    ash (Ash) ether (Ethernet) ax25 (AMPR AX.25)
    netrom (AMPR NET/ROM) rose (AMPR ROSE) arcnet (ARCnet)
    dlci (Frame Relay DLCI) fddi (Fiber Distributed Data Interface) hippi (HIPPI)
    irda (IrLAP) x25 (generic X.25) infiniband (InfiniBand)
    eui64 (Generic EUI-64)
```
## 2. ```arpwatch```：监听 ARP 记录
## 3. ```arping```：发送 ARP 请求到一个相邻主机
## 4. ```finger```：查找并显示用户信息
## 5. ```ifconfig```：设置网络接口
## 6. ```iwconfig```：设置无线网卡
## 7. ```hostname```：显示主机名
## 8. ```ifup```：激活设备
## 9. ```ifdown```：禁用网络设备
## 10. ```mii-tool```：调整网卡模式
## 11. ```route```：设置路由表
## 12. ```netstat```：查看网络连接
## 13. ```ping```：检测主机的连通性
## 14. ```traceroute```：检查数据包所经过的路由器
## 15. ```wget```：下载文件
## 16. ```telnet```：远程登陆
## 17. ```ethtool```：查询及设置网卡参数
## 18. ```tc```：显示和维护流量控制设置