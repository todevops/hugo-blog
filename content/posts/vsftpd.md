---
title: FTP 教程
date: 2018-11-20
categories:
  - linux
tags:
  - linux
  - ftp
---

一份简易的 ftp 安装使用教程
<!--more-->

## 1. 安装 vsftpd

```bash
# 查看 selinux 状态
getenforce
# 查看防火墙状态
service firewalld status
# 安装 vsftpd
yum -y install vsftpd
# 设置开机启动
systemctl enable vsftpd
# 启动 vsftpd 服务
systemctl start vsftpd
# 防火墙开放 ftp 服务
firewall-cmd --add-service=ftp --permanent
# 查看和更改 ftp 的 sebool 值
getsebool -a | grep ftp
setsebool ftpd_full_access on
```

## 2. 配置 vsftpd

```bash
vim /etc/vsftpd/vsftpd.conf
```
vsftpd.conf
```conf
# 不允许匿名访问 ftp
anonymous_enable=NO
# 用户不允许切换上级目录
chroot_local_user=YES
allow_writeable_chroot=YES
# 设置时区
use_localtime=YES
```

```bash
# 修改配置后重启 vsftpd
systemctl restart vsftpd
```

## 3. 创建用户

```bash
# 创建用户 [username]，不允许登陆，家目录为 [directory]
useradd -s /sbin/nologin -m -d [directory] [username]
# 设置密码
echo "[password]" | passwd --stdin [username]
```

## 4. 验证

```bash
# 安装 ftp 客户端
yum -y install ftp
# 连接 ftp 服务器
ftp [ip]
```