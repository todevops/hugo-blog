---
title: SELinux
date: 2018-11-25
categories:
  - linux
tags:
  - linux
  - selinux
---

SELinux
<!--more-->

## 1. 上下文（以 apache 为例）
### 1）服务器端安装 httpd
```
[root@server ~]# yum -y install httpd
[root@server ~]# systemctl start httpd
[root@server ~]# systemctl enable httpd
```

### 2）查看 httpd 默认目录的上下文
```
[root@server ~]# ls -ldZ /var/www/
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 /var/www/
```

### 3）设置临时上下文
```
[root@server ~]# mkdir /html

[root@server ~]# ls -ldZ /html/
drwxr-xr-x. root root unconfined_u:object_r:default_t:s0 /html/

[root@server ~]# chcon -R -t httpd_sys_content_t /html/

[root@server ~]# ls -ldZ /html/
drwxr-xr-x. root root unconfined_u:object_r:httpd_sys_content_t:s0 /html/

[root@server ~]# restorecon -R /html/

[root@server ~]# ls -ldZ /html/
drwxr-xr-x. root root unconfined_u:object_r:default_t:s0 /html/

[root@server ~]# # --reference 设置和目录相同的上下文
[root@server ~]# chcon -R --reference=/var/www/html /www
```

### 4）设置永久上下文

```
[root@server ~]# semanage fcontext -a -t httpd_sys_content_t '/html(/.*)?'

[root@server ~]# ls -ldZ /html
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 /html

[root@server ~]# restorecon -R /html/

[root@server ~]# ls -ldZ /html
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 /html
```


## 2. 布尔值（以 vsftpd 为例）
+ 如果搭建了一个服务，需要在客户端往服务里写东西，但是无法写入，按照以下不走排查
>1）检查配置文件是否允许写  
>2）检查文件系统是否允许写  
>3）检查 selinux （上下文|布尔值）  

### 1)服务端安装 vsftpd
```
[root@server ~]# yum -y install vsftpd
[root@server ~]# systemctl start vsftpd
[root@server ~]# systemctl enable vsftpd
```
### 2）检查配置文件是否允许写
```
[root@server ~]# vim /etc/vsftpd/vsftpd.conf
```
```conf
# 允许匿名上传
anon_upload_enable=YES
anon_mkdir_write_enable=YES
```
### 3）检查文件系统是否允许写
```
[root@server ~]# ls -ld /var/ftp/
drwxr-xr-x. 4 root root 27 8月  17 14:29 /var/ftp/

[root@server ~]# cd /var/ftp/

[root@server ftp]# mkdir test

[root@server ftp]# chown -R ftp.ftp test

[root@server ftp]# ls -ld test
drwxr-xr-x. 2 ftp ftp 6 8月  17 14:48 test
```

### 4）检查 selinux （上下文|布尔值）

```
[root@server ~]# getsebool -a | grep ftp
ftpd_anon_write --> off
ftpd_connect_all_unreserved --> off
ftpd_connect_db --> off
ftpd_full_access --> off
ftpd_use_cifs --> off
ftpd_use_fusefs --> off
ftpd_use_nfs --> off
ftpd_use_passive_mode --> off
httpd_can_connect_ftp --> off
httpd_enable_ftp_server --> off
tftp_anon_write --> off
tftp_home_dir --> off

[root@server ftp]# setsebool -P ftpd_anon_write on

[root@server ftp]# setsebool -P ftpd_full_access on

[root@server ~]# getsebool -a | grep ftp
ftpd_anon_write --> on
ftpd_connect_all_unreserved --> off
ftpd_connect_db --> off
ftpd_full_access --> on
ftpd_use_cifs --> off
ftpd_use_fusefs --> off
ftpd_use_nfs --> off
ftpd_use_passive_mode --> off
httpd_can_connect_ftp --> off
httpd_enable_ftp_server --> off
tftp_anon_write --> off
tftp_home_dir --> off
```
