---
title: CentOS Oracle 12c 安装指南
date: 2018-12-14
categories: ["database"]
tags:
  - oracle
---

CentOS Oracle 12c 安装指南
<!--more-->

## 1. 优化、安装必要的工具
```
[root@oracle12c ~]$ yum -y install unzip vim lrzsz
[root@oracle12c ~]$ setenforce 0
[root@oracle12c ~]$ sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
[root@oracle12c ~]$ reboot
[root@oracle12c ~]$ sed -i 's/^GSSAPIAuthentication yes$/GSSAPIAuthentication no/' /etc/ssh/sshd_config
[root@oracle12c ~]$ sed -i 's/#UseDNS yes/UseDNS no/' /etc/ssh/sshd_config
[root@oracle12c ~]$ service sshd restart
```

## 2. 安装依赖
```
[root@oracle12c ~]$ yum -y install binutils compat-libcap1 compat-libstdc++-33 compat-libstdc++-33*.i686 elfutils-libelf-devel gcc gcc-c++ glibc*.i686 glibc glibc-devel glibc-devel*.i686 ksh libgcc*.i686 libgcc libstdc++ libstdc++*.i686 libstdc++-devel libstdc++-devel*.i686 libaio libaio*.i686 libaio-devel libaio-devel*.i686 make sysstat unixODBC unixODBC*.i686 unixODBC-devel unixODBC-devel*.i686 libXp
```

## 3. 建立用户和组
```
[root@oracle12c ~]$ groupadd oinstall
[root@oracle12c ~]$ groupadd dba
[root@oracle12c ~]$ groupadd oper
[root@oracle12c ~]$ useradd -g oinstall -G dba,oper oracle
[root@oracle12c ~]$ echo oracle | passwd --stdin oracle
```

## 4. 创建安装、备份目录
```
[root@oracle12c ~]$ mkdir -p /u01/app/oracle/product/12.2.0
[root@oracle12c ~]$ chown -R oracle:oinstall /u01
```

## 5. 更改共享内存
```
[root@oracle12c ~]$ echo "shmfs /dev/shm tmpfs size=12g 0" >> /etc/fstab
```

## 6. 修改内核参数

```
[root@oracle12c ~]$ vim /etc/sysctl.conf
```

```
fs.file-max = 6815744
kernel.sem = 250 32000 100 128
kernel.shmmni = 4096
kernel.shmall = 4294967296
kernel.shmmax = 4398046511104
kernel.panic_on_oops = 1
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
net.ipv4.conf.all.rp_filter = 2
net.ipv4.conf.default.rp_filter = 2
fs.aio-max-nr = 1048576
net.ipv4.ip_local_port_range = 9000 65500
```

```
[root@oracle12c ~]$ sysctl -p
```

## 7. 改文件限制
```
[root@oracle12c ~]$ vim /etc/security/limits.conf
```

```
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536
oracle soft stack 10240
```

## 8. 修改登陆配置文件
```
[root@oracle12c ~]$ vim /etc/pam.d/login
```
```
session required pam_limits.so
```

## 9. 修改 ulimit
```
[root@oracle12c ~]$ vim /etc/profile
```
```
if [ $USER = "oracle" ]; then
    if [ $SHELL = "/bin/ksh" ]; then
        ulimit -p 16384
        ulimit -n 65536a
    else
        ulimit -u 16384 -n 65536
    fi
fi
```

## 10. 修改环境变量
```
[root@oracle12c ~]$ vim ~/.bash_profile
```
```
PATH=$PATH:$HOME/.local/bin:$HOME/bin:$ORACLE_HOME/bin
ORACLE_BASE=/u01/app/oracle
ORACLE_HOME=$ORACLE_BASE/product/12.2.0
ORACLE_SID=orcl
export ORACLE_BASE ORACLE_HOME ORACLE_SID
export NLS_LANG="AMERICAN_AMERICA.UTF8"
export PATH
```

## 11. 修改 hosts 文件
```
[root@oracle12c ~]$ vim /etc/hosts
```
```
[ipaddr] oracle12c
```

## 12. 上传安装包 linuxx64_12201_database.zip
```
[oracle@oracle12c ~]$ rz
```

## 13. 静默安装数据库软件
```
[oracle@oracle12c ~]$ cd database
[oracle@oracle12c database]$ ./runInstaller  -silent -ignoreSysPrereqs -ignorePrereq -responseFile /home/oracle/database/response/db_install.rsp
```

/home/oracle/database/response/db_install.rsp

```
oracle.install.responseFileVersion=/oracle/install/rspfmt_dbinstall_response_schema_v12.2.0
oracle.install.option=INSTALL_DB_SWONLY
UNIX_GROUP_NAME=dba
INVENTORY_LOCATION=/u01/app/oracle/oraInventory
ORACLE_HOME=/u01/app/oracle/product/12.2.0
ORACLE_BASE=/u01/app/oracle
oracle.install.db.InstallEdition=EE
oracle.install.db.OSDBA_GROUP=dba
oracle.install.db.OSOPER_GROUP=dba
oracle.install.db.OSBACKUPDBA_GROUP=dba
oracle.install.db.OSDGDBA_GROUP=dba
oracle.install.db.OSKMDBA_GROUP=dba
oracle.install.db.OSRACDBA_GROUP=dba
oracle.install.db.CLUSTER_NODES=
oracle.install.db.isRACOneInstall=false
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false
DECLINE_SECURITY_UPDATES=true
```

## 14. 建立数据库
```
[oracle@oracle12c ~]$ $ORACLE_HOME/bin/dbca -silent -createDatabase -templateName General_Purpose.dbc -gdbname orcl -sid orcl -responseFile NO_VALUE -characterSet AL32UTF8 -memoryPercentage 50 -emConfiguration LOCAL
```

root 用户下执行以下命令

```
[root@oracle12c ~]$ /oracle/oraInventory/orainstRoot.sh
[root@oracle12c ~]$ /oracle/product/12.2.0/root.sh
```

## 15. 修改参数
```
[oracle@oracle12c ~]$ sqlplus / as sysdba
```
```sql
SQL> alter system set sga_target=0 scope=spfile;
SQL> alter system set pga_aggregate_target=0 scope=spfile;
SQL> alter system set db_files=2000 scope=spfile;
SQL> alter system set memory_max_target=12g scope=spfile;
SQL> alter system set memory_target=8g scope=spfile;
SQL> alter system set LOG_ARCHIVE_DEST_1='LOCATION=/backup/arch' scope=spfile;
SQL> alter SYSTEM set db_recovery_file_dest_size=4096M scope=spfile;
SQL> alter system set undo_retention=3600 scope=spfile;
SQL> alter system set processes=6000 scope=spfile;
SQL> alter system set sessions=8000 scope=spfile;
SQL> alter system set open_cursors=2000 scope=both;
SQL> create pfile='/tmp/pfile01.ora' from spfile;
SQL> alter system set filesystemio_options='asynch' scope=spfile;
SQL> alter system set disk_asynch_io=true SCOPE=SPFILE;
SQL> create directory expdp_dir as '/backup/expdp';
SQL> grant write,read on directory expdp_dir to public;
SQL> alter database add logfile group 4 '/u01/app/oracle/oradata/orcl/redo04.log' size 1G;
SQL> alter database add logfile group 5 '/u01/app/oracle/oradata/orcl/redo05.log' size 1G;
SQL> alter database add logfile group 6 '/u01/app/oracle/oradata/orcl/redo06.log' size 1G;
SQL> alter system switch logfile;
SQL> alter system switch logfile;
SQL> alter system switch logfile;
SQL> alter system checkpoint;
SQL> alter database drop logfile group 1;
SQL> alter database drop logfile group 2;
SQL> alter database drop logfile group 3;
SQL> host rm -rf /u01/app/oracle/oradata/orcl/redo0[1-3].log
SQL> alter database add logfile group 1 '/u01/app/oracle/oradata/orcl/redo01.log' size  1G;
SQL> alter database add logfile group 2 '/u01/app/oracle/oradata/orcl/redo02.log' size  1G;
SQL> alter database add logfile group 3 '/u01/app/oracle/oradata/orcl/redo03.log' size  1G; 
SQL> alter system switch logfile;
SQL> alter system switch logfile;
SQL> alter system switch logfile;
SQL> alter system checkpoint;
SQL> shutdown immediate
SQL> startup mount
SQL> alter database archivelog;
SQL> alter database open;
SQL> alter profile default limit FAILED_LOGIN_ATTEMPTS unlimited;
SQL> create pfile='/tmp/pfile02.ora' from spfile;
```

## 16. 修改 Oracle 服务启动配置
```
[root@oracle12c ~]$ vim /etc/oratab
```
```
orcl:/u01/app/oracle/product/12.2.0:Y
```
