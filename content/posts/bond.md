---
title: Red Hat Enterprise Linux 7 配置网络绑定
date: 2020-01-15
categories:
  - linux
tags:
  - network
---

Red Hat Enterprise Linux 7 可让管理员将多个网络接口绑定在一起作为单一、绑定的频道。频道绑定可让两个或多个接口作为一个接口动作， 同时增加带宽，并提供冗余。
<!--more-->

## 1. 常见的网卡绑定驱动模式:

> active-backup、balance-tlb 和 balance-alb 模式不需要交换机的任何特殊配置。其他绑定模式需要配置交换机以便整合链接。  
例如：Cisco 交换机需要在模式 0、2 和 3 中使用 EtherChannel，但在模式 4 中需要 LACP 和 EtherChannel。  
有关交换机附带文档，请查看 https://www.kernel.org/doc/Documentation/networking/bonding.txt。

- **balance-rr** 或者 0 — 为容错及负载平衡设定轮询机制。从第一个可用的绑定从属接口开始按顺序接收和发送传输数据。
- **active-backup** 或者 1 — 为容错设定 active-backup 策略。 通过第一个可用的绑定从属接口接收和发送传输文件。只有在活动的绑定从属接口失败时才使用其他绑定从属接口。
- **balance-xor** 或者 2 — 只根据所选哈希策略传输数据。默认为使用源的 XOR 和目标 MAC 地址与从属接口数的余数相乘生成哈希。在这个模式中，指向具体对等接口的模式流量总是使用同一接口发送。因为目标是由 MAC 地址决定，因此这个方法最适合相同链接或本地网络的对等接口流量。如果流量必须通过单一路由器，那么这个流量平衡模式将是次选模式。
- **broadcast** 或者 3 — 为容错设定广播策略。可在所有从属接口中传输所有数据。
- **802.3ad** 或者 4 — 设定 IEEE 802.3ad 动态链接聚合策略。创建一个共享同一速度和双工设置的聚合组。在所有活跃聚合器中传输和接受数据。需要兼容 802.3ad 的交换机。
- **balance-tlb** 或者 5 — 为容错及负载平衡设定传输负载平衡（TLB）策略。传出流量会根据每个从属接口的当前负载分布。传入流量由当前从属接口接收。如果接收数据从属接口失败，另一个从属接口会接管失败从属接口的 MAC 地址。这个模式只适用于内核绑定模式了解的本地地址，因此无法在桥接后的虚拟机中使用。
- **balance-alb** 或者 6 — 为容错及负载平衡设定自适应负载平衡（ALB）策略，包括用于 IPv4 流量的传输及接收负载平衡。使用 ARP 协商获得接收负载平衡。这个模式只适用于内核 binding 模块了解的本地地址，因此无法在桥接后的虚拟机中使用。

## 2. 主接口及从属接口的默认行为

使用 NetworkManager 守护进程控制绑定的从属接口时，特别是在查找出现问题时，请记住以下几点：
- 启动主接口不会自动启动从属接口。
- 启动从属接口总是启动主接口。
- 停止主接口也可以停止从属接口。
- 没有从属接口的主接口可启动静态 IP 连接。
- 没有从属接口的主接口会在启动 DHCP 连接时等待从属接口。
- 有 DHCP 连接的主接口会在添加有载波的从属接口时等待从属接口完成。
- 有 DHCP 连接的主接口会在添加没有载波的从属接口时等待从属接口完成。

## 3. 使用 (NetworkManager) nmcli 工具配置网卡绑定

```bash
nmcli con add type bond con-name bond0 ifname bond0 mode active-backup
nmcli con add type bond-slave ifname enp0s9 master bond0
nmcli con add type bond-slave ifname enp0s10 master bond0
```

要启动绑定，则必须首先启动从属接口：

```bash
nmcli c up bond-slave-enp0s9

nmcli c up bond-slave-enp0s10
```

启动绑定：

```bash
nmcli c up bond0
```

```bash
nmcli c modify bond0 ipv4.addresses 192.168.33.32/24
nmcli c modify bond0 ipv4.gateway 192.168.33.1
nmcli c modify bond0 ipv4.dns 192.168.33.1
nmcli c modify bond0 ipv4.method manual
```

```bash
systemctl restart network
```

## 3. 使用命令行界面配置网卡绑定

### 检查是否已安装 Bonding 内核模块

> 在 Red Hat Enterprise Linux 7 中默认载入 bonding 模块。

显示 boding 模块信息

```bash
modinfo bonding
```

### Bonding 模块指令

查看所有现有绑定（包括未启动的绑定），请运行：
```bash
cat /sys/class/net/bonding_masters
```

绑定接口参数
- `ad_select=value` - 指定要使用的 802.3ad 聚合选择逻辑。
- `arp_interval=time_in_milliseconds` - 以毫秒为单位指定 ARP 监控的频繁度。默认将这个数值设定为 0，即禁用该功能。
- `arp_ip_target=ip_address[,ip_address_2,…ip_address_16]` - 启用 arp_interval 参数后，指定 ARP 请求的目标 IP 地址。在使用逗号分开的列表中最多可指定 16 个 IP 地址。
- `arp_validate=value` - 验证 ARP 探测的源/分配，默认为 none。其他值为 active、backup 和 all。
- `downdelay=time_in_milliseconds` - 以毫秒为单位指定从链接失败到禁用该链接前要等待的时间。该值必须是 miimon 参数中的多个数值。默认将其设定为 0，即禁用该功能。
- `fail_over_mac=value` - 指定 active-backup 模式是否应该将所有从属连接设定为使用同一 MAC 地址作为 enslavement（传统行为），或在启用时根据所选策略执行绑定 MAC 地址的特殊处理。
- `lacp_rate=value` - 指定链接伙伴应使用 802.3ad 模式传输 LACPDU 的速率
- `miimon=time_in_milliseconds` - 以毫秒为单位指定 MII 链接监控的频率。
- `mode=value` - 允许您指定绑定的策略。
- `primary=interface_name` - 指定主设备的接口名称，比如 eth0。
- `primary_reselect=value` - 为主从属接口指定重新选择策略。
- `resend_igmp=range` - 指定故障转移事件后要进行的 IGMP 成员报告数。故障转移后会立即提交一个报告，之后会每隔 200 毫秒发送数据包。
- `updelay=time_in_milliseconds` - 以毫秒为单位指定启用某个链接前要等待的时间。该数值必须是在 miimon 参数值指定值的倍数。默认设定为 0，即禁用该参数。
- `use_carrier=number` - 指定 miimon 是否应该使用 MII/ETHTOOL ioctls 或者 netif_carrier_ok() 来决定该链接状态。
- `xmit_hash_policy=value` - 选择 balance-xor 和 802.3ad 模式中用来选择从属接口的传输哈希策略。

### 创建频道绑定接口

> 必须在 `ifcfg-bondN` 接口文件的 `BONDING_OPTS="bonding parameters separated by spaces"` 指令中，使用以空格分开的列表指定 bonding 内核模块。  
请不要在 `/etc/modprobe.d/bonding.conf` 文件或弃用的 `/etc/modprobe.conf` 文件中为绑定设备指定选项。  
`max_bonds` 参数不是具体接口的参数，且不应在使用 `BONDING_OPTS` 指令的 `ifcfg-bondN` 文件中设定，因为这个指令会让网络脚本根据需要创建绑定接口。

在 /etc/sysconfig/network-scripts/ 目录中创建名为 ifcfg-bondN 的文件，使用接口号码替换 N，比如 0。

```bash
cat << EOF > /etc/sysconfig/network-scripts/ifcfg-bond0
DEVICE=bond0
NAME=bond0
TYPE=Bond
BONDING_MASTER=yes
IPADDR=192.168.33.32
PREFIX=24
GATEWAY=192.168.33.1
DNS1=192.168.33.1
ONBOOT=yes
BOOTPROTO=none
BONDING_OPTS="miimon=1000 mode=1 primary=enp0s9"
EOF
```

### 创建从属接口

在 /etc/sysconfig/network-scripts/ 目录中创建名为 ifcfg-ethN 的文件，使用接口号码替换 N，比如 0。

```bash
cat << EOF > /etc/sysconfig/network-scripts/ifcfg-enp0s9
DEVICE=enp0s9
NAME=bond0-slave-enp0s9
TYPE=Ethernet
BOOTPROTO=none
ONBOOT=yes
MASTER=bond0
SLAVE=yes
EOF
```

```bash
cat << EOF > /etc/sysconfig/network-scripts/ifcfg-enp0s10
DEVICE=enp0s10
NAME=bond0-slave-enp0s10
TYPE=Ethernet
BOOTPROTO=none
ONBOOT=yes
MASTER=bond0
SLAVE=yes
EOF
```

### 激活频道绑定

```bash
ifup bond0
ifup ifcfg-enp0s9
ifup ifcfg-enp0s10

# 生效更改
nmcli con load /etc/sysconfig/network-script/ifcfg-device
nmcli con reload
# 查看网卡绑定接口状态
ip link show
```
