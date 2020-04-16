---
title: Zabbix 安装指南
date: 2018-11-17
categories:
  - monitor
tags:
  - linux
  - zabbix
---
Zabbix 3.4 安装指南
<!--more-->

## CentOS 安装 zabbix3.4
### 1. 准备
```shell
rpm -i http://repo.zabbix.com/zabbix/3.4/rhel/7/x86_64/zabbix-release-3.4-2.el7.noarch.rpm
yum install zabbix-server-mysql zabbix-web-mysql zabbix-agent
yum -y install php-fpm php php-mysql php-gd php-bcmath php-mbstring php-xml php-ldap
```
### 2. 设置数据库
```mysql
/usr/bin/mysqladmin -u root password 123456
mysql -uroot -p
password
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> grant all privileges on zabbix.* to zabbix@localhost identified by 'zabbix';
mysql> quit;
```
### 3. 数据库导入数据
```shell
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```
### 4. 修改 zabbix 配置文件增加 DBPassword=zabbix
```shell
vim /etc/zabbix/zabbix_server.conf
```
```conf
DBPassword=zabbix
```
### 5. 将时区设置为：亚洲/上海
```shell
vim /etc/httpd/conf.d/zabbix.conf
php_value date.timezone Asia/Shanghai
```
### 6. 启动服务，将服务设为开机启动
```shell
systemctl restart zabbix-server zabbix-agent httpd
systemctl enable zabbix-server zabbix-agent httpd
```
### 7. 修改默认语言，文字配置
```
vim /usr/share/zabbix/include/locales.inc.php
cp simkai.ttf /usr/share/zabbix/fonts/
vim /usr/share/zabbix/include/defines.inc.php
```