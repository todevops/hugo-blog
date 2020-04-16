---
title: MySQL Group Replication 安装及配置
date: 2019-10-15 01:31:26
categories: ["database"]
tags:
  - mysql
---

MySQL 组复制
<!--more-->

### 安装Docker
```
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum -y install docker-ce
systemctl enable docker
systemctl start docker
docker ps
```
### 创建网络
```
docker pull mysql:5.7
docker network create --subnet=192.168.100.0/24 mysqlsubnet
docker network ls
```

### 编辑配置文件

```
mkdir /mysqldata && cd /mysqldata
mkdir s{1..3}

./conf.d/
├── s1
│   └── my.cnf
├── s2
│   └── my.cnf
└── s3
    └── my.cnf
```

> /mysqldata/conf.d/s1/my.cnf

```
[mysqld]
port=3306
datadir=/var/lib/mysql
pid-file=/var/run/mysqld/mysqld.pid
socket=/var/run/mysqld/mysqld.sock
server_id=1
gtid_mode=ON
enforce_gtid_consistency=ON
master_info_repository=TABLE
relay_log_info_repository=TABLE
binlog-format=ROW
binlog_checksum=NONE
log-slave-updates=1
log_bin=binlog
relay-log=bogon-relay-bin

transaction_write_set_extraction = XXHASH64
loose-group_replication_group_name="81263447-5a2b-11e9-94c6-0242c0a86465"
loose-group_replication_start_on_boot = off
loose-group_replication_local_address = '192.168.100.101:33061'
loose-group_replication_group_seeds ='192.168.100.101:33061,192.168.100.103:33061,192.168.100.105:33061'
loose-group_replication_bootstrap_group = off
```

> /mysqldata/conf.d/s2/my.cnf

```
[mysqld]
port=3306
datadir=/var/lib/mysql
pid-file=/var/run/mysqld/mysqld.pid
socket=/var/run/mysqld/mysqld.sock
server_id=3
gtid_mode=ON
enforce_gtid_consistency=ON
master_info_repository=TABLE
relay_log_info_repository=TABLE
binlog-format=ROW
binlog_checksum=NONE
log-slave-updates=1
log_bin=binlog
relay-log=bogon-relay-bin

transaction_write_set_extraction = XXHASH64
loose-group_replication_group_name="81263447-5a2b-11e9-94c6-0242c0a86465"
loose-group_replication_start_on_boot = off
loose-group_replication_local_address = '192.168.100.103:33061'
loose-group_replication_group_seeds ='192.168.100.101:33061,192.168.100.103:33061,192.168.100.105:33061'
loose-group_replication_bootstrap_group = off
```

> /mysqldata/conf.d/s2/my.cnf

```
[mysqld]
port=3306
datadir=/var/lib/mysql
pid-file=/var/run/mysqld/mysqld.pid
socket=/var/run/mysqld/mysqld.sock
server_id=5
gtid_mode=ON
enforce_gtid_consistency=ON
master_info_repository=TABLE
relay_log_info_repository=TABLE
binlog-format=ROW
binlog_checksum=NONE
log-slave-updates=1
log_bin=binlog
relay-log=bogon-relay-bin

transaction_write_set_extraction = XXHASH64
loose-group_replication_group_name="81263447-5a2b-11e9-94c6-0242c0a86465"
loose-group_replication_start_on_boot = off
loose-group_replication_local_address = '192.168.100.105:33061'
loose-group_replication_group_seeds ='192.168.100.101:33061,192.168.100.103:33061,192.168.100.105:33061'
loose-group_replication_bootstrap_group = off
```

### 启动mysql
```
docker run --detach --memory=512m --memory-swap=1g --hostname=mgr-s1 --net=mysqlsubnet --ip=192.168.100.101 --add-host mgr-s2:192.168.100.103 --add-host mgr-s3:192.168.100.105 --publish 3306:3306 --volume=/mysqldata/conf.d/s1/:/etc/mysql/conf.d --volume=/mysqldata/s1:/var/lib/mysql --name=mgr-s1 -e MYSQL_ROOT_PASSWORD=password -d mysql:5.7
docker run --detach --memory=512m --memory-swap=1g --hostname=mgr-s2 --net=mysqlsubnet --ip=192.168.100.103 --add-host mgr-s1:192.168.100.101 --add-host mgr-s3:192.168.100.105 --publish 3307:3306 --volume=/mysqldata/conf.d/s2/:/etc/mysql/conf.d --volume=/mysqldata/s2:/var/lib/mysql --name=mgr-s2 -e MYSQL_ROOT_PASSWORD=password -d mysql:5.7
docker run --detach --memory=512m --memory-swap=1g --hostname=mgr-s3 --net=mysqlsubnet --ip=192.168.100.105 --add-host mgr-s1:192.168.100.101 --add-host mgr-s2:192.168.100.103 --publish 3308:3306 --volume=/mysqldata/conf.d/s3/:/etc/mysql/conf.d --volume=/mysqldata/s3:/var/lib/mysql --name=mgr-s3 -e MYSQL_ROOT_PASSWORD=password -d mysql:5.7
```

### 设置复制组
```

docker exec -it mgr-s1 /bin/bash

mysql -uroot -ppassword

SET SQL_LOG_BIN=0;
CREATE USER rpl_user@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO rpl_user@'%';
FLUSH PRIVILEGES;
SET SQL_LOG_BIN=1;
CHANGE MASTER TO MASTER_USER='rpl_user', MASTER_PASSWORD='password' FOR CHANNEL 'group_replication_recovery';
INSTALL PLUGIN group_replication SONAME 'group_replication.so';
SHOW PLUGINS;
SET GLOBAL group_replication_bootstrap_group=ON;
START GROUP_REPLICATION;
SET GLOBAL group_replication_bootstrap_group=OFF;
SELECT * FROM performance_schema.replication_group_members;
```

### 向复制组中增加成员1
```
docker exec -it mgr-s2 /bin/bash

mysql -uroot -ppassword

SET SQL_LOG_BIN=0;
CREATE USER rpl_user@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO rpl_user@'%';
FLUSH PRIVILEGES;
SET SQL_LOG_BIN=1;
CHANGE MASTER TO MASTER_USER='rpl_user', MASTER_PASSWORD='password' FOR CHANNEL 'group_replication_recovery';
INSTALL PLUGIN group_replication SONAME 'group_replication.so';
SHOW PLUGINS;
set global group_replication_allow_local_disjoint_gtids_join=ON;
START GROUP_REPLICATION;
SELECT * FROM performance_schema.replication_group_members;
```

### 向复制组中增加成员2
```
docker exec -it mgr-s3 /bin/bash

mysql -uroot -ppassword

SET SQL_LOG_BIN=0;
CREATE USER rpl_user@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO rpl_user@'%';
FLUSH PRIVILEGES;
SET SQL_LOG_BIN=1;
CHANGE MASTER TO MASTER_USER='rpl_user', MASTER_PASSWORD='password' FOR CHANNEL 'group_replication_recovery';
INSTALL PLUGIN group_replication SONAME 'group_replication.so';
SHOW PLUGINS;
set global group_replication_allow_local_disjoint_gtids_join=ON;
START GROUP_REPLICATION;
SELECT * FROM performance_schema.replication_group_members;
```
