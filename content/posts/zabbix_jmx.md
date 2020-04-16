---
title: Zabbix 通过 jmx 监控 Tomcat
date: 2018-11-16
categories:
  - monitor
tags:
  - linux
  - zabbix
  - tomcat
---

Zabbix 通过 jmx 监控 Tomcat
<!--more-->

## 1. 安装 jdk 和 zabbix-java-gateway
```bash
# 安装 openjdk 或者下载 tar.gz
apt install openjdk-8-jdk
# 安装 zabbix-java-gateway
apt install zabbix-java-gateway
```
## 2. 修改服务器端配置
```
vim /etc/zabbix/zabbix_server.conf
```

```conf
JavaGateway=192.168.6.30
JavaGatewayPort=10052
StartJavaPollers=50
```
```bash
systemctl restart zabbix-server
systemctl start zabbix-java-gateway
systemctl enable zabbix-java-gateway
```

## 3. 配置被监控 tomcat
```bash
vim /tomcat/bin/catalina.sh
```
```sh
CATALINA_OPTS="-Djava.rmi.server.hostname=< 被监控 tomcat 主机 IP 地址 > -Djavax.management.builder.initial= -Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.port=12345 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false"
```
更改完成之后重启 tomcat 应用

## 4. zabbix web 配置
- 添加被监控 tomcat