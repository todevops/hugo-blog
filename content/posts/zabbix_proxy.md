---
title: Zabbix Proxy 分布式监控
date: 2018-11-18
categories:
  - monitor
tags:
  - linux
  - zabbix
---

使用 Zabbix Proxy 搭建分布式监控
<!--more-->

## 1. 安装 MySQL、Zabbix Prorxy
```bash
# 安装 MySQL
apt install mysql-server
# 安装 zabbix proxy
wget https://repo.zabbix.com/zabbix/3.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.4-1+xenial_all.deb
dpkg -i zabbix-release_3.4-1+xenial_all.deb
apt update
apt install zabbix-server-mysql zabbix-agent zabbix-proxy-mysql zabbix-get zabbix-sender
```

## 2. 导入数据
```
mysql -uroot -p
```
```sql
--创建 zabbix_proxy 数据库
create database zabbix_proxy character set utf8 collate utf8_bin;
--授权
grant all privileges on zabbix_proxy.* to zabbix@localhost identified by 'password';
```

```bash
zcat /usr/share/doc/zabbix-proxy-mysql/schema.sql.gz | mysql -uzabbix -pzabbix zabbix
```

## 3. 更改配置
```bash
vim /etc/zabbix/zabbix_proxy.conf
```

```conf
Server=192.168.6.30
Hostname=Zabbix proxy
LogFile=/var/log/zabbix/zabbix_proxy.log
LogFileSize=0
PidFile=/var/run/zabbix/zabbix_proxy.pid
SocketDir=/var/run/zabbix
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix
ConfigFrequency=120
DataSenderFrequency=1
SNMPTrapperFile=/var/log/snmptrap/snmptrap.log
Timeout=30
ExternalScripts=/usr/lib/zabbix/externalscripts
FpingLocation=/usr/bin/fping
Fping6Location=/usr/bin/fping6
LogSlowQueries=3000
```
## 4. 启动服务并设置开机启动
```bash
systemctl enable zabbix-proxy zabbix-agent mysql
```
