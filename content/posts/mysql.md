---
title: MySQL 安装指南
date: 2019-02-18
categories:
  - database
tags:
  - mysql
  - replication
---

MySQL 安装指南
<!--more-->

# 1. 安装并初始化数据库

```bash
[root@master ~]# ls
mysql-8.0.15-linux-glibc2.12-x86_64.tar.xz
[root@master ~]# tar -Jxf mysql-8.0.15-linux-glibc2.12-x86_64.tar.xz
[root@master ~]# ls
mysql-8.0.15-linux-glibc2.12-x86_64  mysql-8.0.15-linux-glibc2.12-x86_64.tar.xz
[root@master ~]# mkdir /appdata 
[root@master ~]# mv mysql-8.0.15-linux-glibc2.12-x86_64 /appdata/mysql
[root@master ~]# cd /appdata/mysql/bin
[root@master bin]# ./mysqld --verbose --help
[root@master bin]# ./mysqld --initialize --user=mysql
2019-02-17T13:24:47.271942Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
2019-02-17T13:24:47.272022Z 0 [System] [MY-013169] [Server] /appdata/mysql/bin/mysqld (mysqld 8.0.15) initializing of server in progress as process 9517
2019-02-17T13:24:49.230753Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: fYd00.<Ucx:0
2019-02-17T13:24:50.553995Z 0 [System] [MY-013170] [Server] /appdata/mysql/bin/mysqld (mysqld 8.0.15) initializing of server has completed
```

# 2. 编辑自定义配置文件

```shell
[root@master ~]# vim /etc/my.cnf
```
```conf
[mysqld]
port=3306
basedir=/appdata/mysql
datadir=/appdata/mysql/data
socket=/appdata/mysql/data/mysql.sock
log-bin=mysql-bin
server-id=1
binlog_format=MIXED
gtid_mode = on
enforce_gtid_consistency = 1
log_slave_updates = 1
max_allowed_packet = 128M
interactive_timeout = 28800000
wait_timeout = 28800000

[client]
socket=/appdata/mysql/data/mysql.sock
```
# 3. 启动MySQL
```shell
[root@master ~]# cp /appdata/mysql/support-files/mysql.server /etc/init.d/
[root@master ~]# vim /etc/init.d/mysql.server
```

```
basedir=/appdata/mysql
datadir=/appdata/mysql/data
```

```shell
[root@master ~]# /etc/init.d/mysql.server start
```

# 4. 设置主从同步

```shell
[root@master ~]# mysql -uroot -pfYd00.<Ucx:0
```
```sql
mysql> ALTER USER root@'localhost' IDENTIFIED BY 'Abc123';
Query OK, 0 rows affected (0.04 sec)

mysql> CREATE USER replication@'%' IDENTIFIED WITH 'mysql_native_password' BY 'replication';
Query OK, 0 rows affected (0.04 sec)

mysql> grant replication slave on *.* to replication@'%';
Query OK, 0 rows affected (0.00 sec)

mysql> ALTER TABLE mysql.slave_master_info ENGINE=InnoDB;
Query OK, 0 rows affected (0.07 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> ALTER TABLE mysql.slave_relay_log_info ENGINE=InnoDB;
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> reset slave;
Query OK, 0 rows affected (0.01 sec)

mysql> CHANGE MASTER TO MASTER_HOST="slave.example.com",MASTER_USER='replication',MASTER_PASSWORD="replication",MASTER_AUTO_POSITION=1;
Query OK, 0 rows affected, 2 warnings (0.01 sec)

mysql> start slave;
Query OK, 0 rows affected (0.01 sec)

mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: slave.example.com
                  Master_User: replication
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000015
          Read_Master_Log_Pos: 1567
               Relay_Log_File: master-relay-bin.000003
                Relay_Log_Pos: 456
        Relay_Master_Log_File: mysql-bin.000015
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes


```

# 5. 问题处理

>Error_code: MY-002061

+ 不兼容的变化： 在caching_sha2_password和 sha256_password认证插件提供比更安全的密码加密 mysql_native_password插件，并 caching_sha2_password提供了比更好的性能sha256_password。由于这些优越的安全性和性能特性 caching_sha2_password，它现在是首选的身份验证插件，而且也是默认的身份验证插件而不是 mysql_native_password。此更改会影响服务器和libmysqlclient客户端库：
    + 对于服务器，default_authentication_plugin 系统变量的默认值 从更改 mysql_native_password为 caching_sha2_password。
    + 该libmysqlclient库将其 caching_sha2_password视为默认的身份验证插件而不是 mysql_native_password。
    + 此更改仅影响用于创建新MySQL帐户的身份验证插件。对于已升级安装中已存在的帐户，其身份验证插件保持不变。
    + 使用经过身份验证的帐户的客户端 caching_sha2_password必须使用安全连接（使用TCP使用TLS / SSL凭据，Unix套接字文件或共享内存制作），或使用RSA密钥对支持密码交换的未加密连接。此安全要求不适用 mysql_native_passsword，因此交换机 caching_sha2_password可能需要其他配置（请参阅 缓存SHA-2可插入身份验证）。但是，默认情况下，MySQL 8.0中的客户端连接更喜欢使用TLS / SSL，因此已满足该首选项的客户端可能不需要其他配置。
    + 因为caching_sha2_password现在也是libmysqlclient客户端库中的默认身份验证插件，所以 身份验证需要在客户端/服务器协议中进行额外的往返，以便从MySQL 8.0客户端连接到使用mysql_native_password（以前的默认身份验证插件）的帐户 ，除非调用客户端程序一个 --default-auth=mysql_native_password 选择。
    + 不兼容：尚未更新的客户端和连接器 caching_sha2_password 无法连接到通过身份验证的帐户，caching_sha2_password 因为他们无法将此插件识别为有效。要解决此问题，请libmysqlclient从MySQL 8.0.4或更高版本重新链接客户端 ，或获取可识别的更新连接器 caching_sha2_password。
    + 不兼容：尚未更新的客户端和连接器 caching_sha2_password可能无法连接到配置caching_sha2_password为默认身份验证插件的MySQL 8.0服务器 ，甚至使用未通过身份验证的帐户caching_sha2_password。出现此问题的原因是服务器为客户端指定其默认身份验证插件的名称。如果客户端或连接器基于未正常处理无法识别的默认身份验证插件的客户端/服务器协议实现，则可能会失败并显示错误。

>ERROR 1872 (HY000): Slave failed to initialize relay log info structure from the repository
