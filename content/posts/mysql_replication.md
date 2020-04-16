---
title: 基于 GTID 的 MySQL 双主安装及配置
date: 2019-09-15 01:31:26
categories: ["database"]
tags:
  - mysql
---

基于 GTID 的 MySQL 双主安装及配置
<!--more-->


### 环境
+ CentOS 7
+ MySQL 5.7

hostname | ipaddress
---|---
mysql01 | 192.168.33.31
mysql02 | 192.168.33.32



### 1. 下载MySQL安装包
[mysql-community-server-5.7.28](https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-community-server-5.7.28-1.el7.x86_64.rpm)  
[mysql-community-client-5.7.28](https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-community-client-5.7.28-1.el7.x86_64.rpm)  
[mysql-community-common-5.7.28](https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-community-common-5.7.28-1.el7.x86_64.rpm)  
[mysql-community-libs-5.7.28](https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-community-libs-5.7.28-1.el7.x86_64.rpm)  
[mysql-community-libs-compat-5.7.28](https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm)  
[mysql-community-devel-5.7.28](https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-community-devel-5.7.28-1.el7.x86_64.rpm)


### 2. 安装MySQL

```
[root@mysql01 ~]# yum -y remove mariadb-*
[root@mysql01 ~]# rpm -ivh mysql-community-*
```

```
[root@mysql02 ~]# yum -y remove mariadb-*
[root@mysql02 ~]# rpm -ivh mysql-community-*
```

### 3. 配置/etc/my.cnf

```
[root@mysql01 ~]# cat /etc/my.cnf | grep -Ev "#|^$"
[mysqld]
server-id                       =       1
log-bin                         =       mysql-bin
binlog_format                   =       row
gtid_mode                       =       on
enforce_gtid_consistency        =       on
log-slave-updates               =       on
auto_increment_offset           =       1
auto_increment_increment        =       2
datadir                         =       /var/lib/mysql
socket                          =       /var/lib/mysql/mysql.sock
symbolic-links                  =       0
log-error                       =       /var/log/mysqld.log
pid-file                        =       /var/run/mysqld/mysqld.pid
```

```
[root@mysql02 ~]# cat /etc/my.cnf | grep -Ev "#|^$"
server-id                       =       2
log-bin                         =       mysql-bin
binlog_format                   =       row
gtid_mode                       =       on
enforce_gtid_consistency        =       on
auto_increment_offset           =       2
auto_increment_increment        =       2
datadir                         =       /var/lib/mysql
socket                          =       /var/lib/mysql/mysql.sock
symbolic-links                  =       0
log-error                       =       /var/log/mysqld.log
pid-file                        =       /var/run/mysqld/mysqld.pid
```

### 3. 配置replication

+ mysql01、mysql02创建同步用户

```
[root@mysql01 ~]# systemctl enable mysqld
[root@mysql01 ~]# systemctl start mysqld
[root@mysql01 ~]# mysql -uroot -pP@ssw0rd
mysql> create user replication@'%' identified by 'P@ssw0rd';
mysql> grant replication slave on *.* to replication@'%';
```

```
[root@mysql02 ~]# systemctl enable mysqld
[root@mysql02 ~]# systemctl start mysqld
[root@mysql02 ~]# mysql -uroot -pP@ssw0rd
mysql> create user replication@'%' identified by 'P@ssw0rd';
mysql> grant replication slave on *.* to replication@'%';
```

+ 备库配置slave

```
[root@mysql02 ~]# mysql -uroot -pP@ssw0rd
mysql> CHANGE MASTER TO 
mysql> MASTER_HOST='192.168.33.31', 
mysql> MASTER_USER='replication', 
mysql> MASTER_PASSWORD='P@ssw0rd', 
mysql> MASTER_AUTO_POSITION=1;
mysql> start slave;
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.33.31
                  Master_User: replication
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000005
          Read_Master_Log_Pos: 1146
               Relay_Log_File: mysql02-relay-bin.000003
                Relay_Log_Pos: 1359
        Relay_Master_Log_File: mysql-bin.000005
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 1146
              Relay_Log_Space: 1568
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: be182906-ef69-11e9-b8e6-5254008afee6
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set: be182906-ef69-11e9-b8e6-5254008afee6:1-5
            Executed_Gtid_Set: be182906-ef69-11e9-b8e6-5254008afee6:1-5
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)
```

```
[root@mysql01 ~]# mysql -uroot -pP@ssw0rd
mysql> CHANGE MASTER TO 
mysql> MASTER_HOST='192.168.33.32', 
mysql> MASTER_USER='replication', 
mysql> MASTER_PASSWORD='P@ssw0rd', 
mysql> MASTER_AUTO_POSITION=1;
mysql> start slave;
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.33.32
                  Master_User: replication
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 859
               Relay_Log_File: mysql01-relay-bin.000007
                Relay_Log_Pos: 1072
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 859
              Relay_Log_Space: 1581
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 2
                  Master_UUID: dc67074d-ef6a-11e9-be75-5254008afee6
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set: dc67074d-ef6a-11e9-be75-5254008afee6:1-4
            Executed_Gtid_Set: be182906-ef69-11e9-b8e6-5254008afee6:1-31944,
dc67074d-ef6a-11e9-be75-5254008afee6:1-4
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)
```
