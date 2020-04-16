---
title: Red Hat Enterprise Linux 7 配置网络成组
date: 2020-01-15
categories:
  - linux
tags:
  - network
---

网络成组旨在通过提供小内核驱动程序，以便使用不同的方法应用这个概念，实现数据包流的快速处理，并让各种用户空间应用程序在用户空间执行各种任务。
<!--more-->


## 网络成组和绑定对比

![](/resources/linuxbond/bond&team.png)

## 安装 teamd
默认不会安装网络成组守护进程 teamd。要安装 teamd
```bash
yum -y install teamd
```

## 将绑定转换为成组

```bash
bond2team --examples
```

```
The following commands will deliver the ifcfg files into a temporary
directory. You can review the files and copy to the right location.

Add the following argument to the commands below to print the output
to the screen instead of writing to files.
  --stdout

Add the following arguments to the commands below to set the
destination directory for the output files.
  --outputdir </path/to/dir>

Add the following argument to the commands below to output the
files in teamd format (JSON) instead of the default ifcfg format.
  --json

To convert the current "bond0" ifcfg configuration to team ifcfg:
# /usr/bin/bond2team --master bond0

To convert the current "bond0" ifcfg configuration out of the
standard ifcfg-:
# /usr/bin/bond2team --master bond0 --configdir </path/to/ifcfg>

To convert the current "bond0" ifcfg configuration to team ifcfg
renaming the interface name to "team0". (carefull: firewall rules,
aliases interfaces, etc., will break after the renaming because the
tool will only change the ifcfg file, nothing else)
# /usr/bin/bond2team --master bond0 --rename team0

To convert given bonding parameters without any ifcfg:
# /usr/bin/bond2team --bonding_opts "mode=1 miimon=500"

To convert given bonding parameters without any ifcfg with ports:
# /usr/bin/bond2team --bonding_opts "mode=1 miimon=500 primary=eth1 primary_reselect-0" \
     --port eth1 --port eth2 --port eth3 --port eth4
```

```bash
# 保留名称
/usr/bin/bond2team --master bond0

Resulted files:
  /tmp/bond2team.j8PiKA/ifcfg-bond0
  /tmp/bond2team.j8PiKA/ifcfg-enp0s10
  /tmp/bond2team.j8PiKA/ifcfg-enp0s9
```

```bash
# 使用新名称保存该配置
# 添加 --json 选项输出 JSON 格式文件
/usr/bin/bond2team --master bond0 --rename team0

Resulted files:
  /tmp/bond2team.vwCjgt/ifcfg-team0
  /tmp/bond2team.vwCjgt/ifcfg-enp0s10
  /tmp/bond2team.vwCjgt/ifcfg-enp0s9
```

```bash
# 将绑定转换为成组并指定文件路径
# 添加 --json 选项输出 JSON 格式文件
/usr/bin/bond2team --master bond0 --configdir /etc/sysconfig/network-scripts/
```
