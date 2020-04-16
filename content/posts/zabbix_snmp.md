---
title: Zabbix 通过 SNMP 进行监控
date: 2018-11-15
categories:
  - linux
tags:
  - zabbix
---

Zabbix 通过 SNMP 进行监控
<!--more-->

# SNMP 监控 Linux
## 1. 被监控主机安装 net-snmp
```
[root@client ~]# yum install -y net-snmp
```
## 2. 修改配置文件
```
[root@client ~]# vim /etc/snmp/snmpd.conf
```
```
# sec.name  source          community
com2sec notConfigUser  default       public

view    systemview    included   .1
view    systemview    included   .1.3.6.1.2.1.1
view    systemview    included   .1.3.6.1.2.1.25.1.1
```
## 3. 启动

```
[root@client ~]# systemctl start snmpd.service
[root@client ~]# netstat -nlp | grep 161
```

## 4. 在 zabbix server 上测试
```
[root@zabbix ~]# yum install -y net-snmp
[root@zabbix ~]# snmpwalk -v 2c -c zabbix 192.168.1.51 | wc -l
```

## 5. zabbix的web界面添加主机
+ 添加模板

![]({{ "/assets/images/linux-snmp/2.png" | absolute_url }})

+ 设置 communities

![]({{ "/assets/images/linux-snmp/1.png" | absolute_url }})

# SNMP 监控 ESXI
## 1. 开启 ESXI 的 SNMP 服务（允许所有主机访问）
+ 设置 communities

```
[root@esxi:~] esxcli system snmp set --communities public
```
+ 开启 SNMP 服务

```
[root@esxi:~] esxcli system snmp set --enable true
```
+ 允许所有主机访问 SNMP

```
[root@esxi:~] esxcli network firewall ruleset set --ruleset-id snmp --allowed-all true
Already allowed all ip
```
+ 设置防火墙

```
[root@esxi:~] esxcli network firewall ruleset set --ruleset-id snmp --enabled true
```
+ 重启 SNMP 服务

```
[root@esxi:~] /etc/init.d/snmpd restart
root: snmpd Running from interactive shell, running command: esxcli system snmp set -e false.
root: snmpd setting up resource reservations.
root: snmpd opening firewall port(s) for notifications.
root: snmpd watchdog for snmpd started.
```

## 2. 开启 ESXI 主机的 SNMP 服务（允许特定主机访问）

+ 禁止所有主机访问 SNMP

```
[root@esxi:~] esxcli network firewall ruleset set --ruleset-id snmp --allowed-all false
```
+ 设置防火墙

```
[root@esxi:~] esxcli network firewall ruleset allowedip add --ruleset-id snmp --ip-address 10.0.101.0/24
[root@esxi:~] esxcli network firewall ruleset set --ruleset-id snmp --enabled true
```
+ 重启 SNMP 服务

```
[root@esxi:~] /etc/init.d/snmpd restart
```

## 3. 测试是否能获取 SNMP 数据
+ 在其他服务器上安装 SNMP

```
[root@zabbix ~]# yum -y install net-snmp net-snmp-utils net-snmp-devel
```

+ 测试获取信息

```
[root@zabbix ~]# snmpwalk -v 2c -c sunwoda 192.168.9.24:161 | wc -l
4594
```

## 4. Zabbix 添加主机
+ 添加模板

![]({{ "/assets/images/esxi-snmp/1.png" | absolute_url }})

+ 设置 communities

![]({{ "/assets/images/esxi-snmp/2.png" | absolute_url }})


