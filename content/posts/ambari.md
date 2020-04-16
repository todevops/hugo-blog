---
title: 使用 Ambari 部署 Hadoop 集群
date: 2018-12-18
categories:
  - linux
tags:
  - linux
  - ambari
  - hadoop
---

使用 Ambari 部署 Hadoop 集群
<!--more-->

## 配置本地yum源
### 安装httpd
```shell
yum install httpd -y
```
### 在官方下载镜像文件
```shell
wget http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.6.1.0/ambari.repo
wget http://public-repo-1.hortonworks.com/HDP/centos7/2.x/updates/2.6.4.0/hdp.repo
wget http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.6.1.0/ambari-2.6.1.0-centos7.tar.gz
wget http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.22/repos/centos7/HDP-UTILS-1.1.0.22-centos7.tar.gz
wget http://public-repo-1.hortonworks.com/HDP-GPL/centos7/2.x/updates/2.6.4.0/HDP-GPL-2.6.4.0-centos7-rpm.tar.gz
wget http://public-repo-1.hortonworks.com/HDP/centos7/2.x/updates/2.6.4.0/HDP-2.6.4.0-centos7-rpm.tar.gz
```
### 将对应的tar包解压到httpd的文件目录
```shell
cd /var/www/html
tar -zxvf ambari-2.6.1.0-centos7.tar.gz
tar -zxvf HDP-2.6.4.0-centos7-rpm.tar.gz 
tar -zxvf HDP-GPL-2.6.4.0-centos7-rpm.tar.gz 
tar -zxvf HDP-UTILS-1.1.0.22-centos7.tar.gz
```
### 配置基础源，创建hadoop的repo文件

```shell
# ambari 源
vim /etc/yum.repo.d/ambari.repo
```
```conf
[ambari-2.6.1.0]
name=ambari Version - ambari-2.6.1.0
baseurl=http://192.168.10.11/ambari/centos7/2.6.1.0-143
gpgcheck=1
gpgkey=http://192.168.10.11/ambari/centos7/2.6.1.0-143/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1
```

```shell
# hadoop源
vim /etc/yum.repo.d/hdp.repo
```
```conf
#VERSION_NUMBER=2.6.4.0-91
[HDP-2.6.4.0]
name=HDP Version - HDP-2.6.4.0
baseurl=http://192.168.10.11/HDP/centos7/2.6.4.0-91
gpgcheck=1
gpgkey=http://192.168.10.11/HDP/centos7/2.6.4.0-91/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1

[HDP-UTILS-1.1.0.22]
name=HDP-UTILS Version - HDP-UTILS-1.1.0.22
baseurl=http://192.168.10.11/HDP-UTILS/centos7/1.1.0.22/
gpgcheck=1
gpgkey=http://192.168.10.11/HDP-UTILS/centos7/1.1.0.22/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1

[HDP-GPL-2.6.4.0]
name=HDP-GPL Version - HDP-GPL-2.6.4.0
baseurl=http://192.168.10.11/HDP-GPL/centos7/2.6.4.0-91
gpgcheck=1
gpgkey=http://192.168.10.11/HDP-GPL/centos7/2.6.4.0-91/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1
```
### 启动httpd
```shell
systemctl enable httpd
systemctl start httpd
```
### 将本地源的repo配置拷贝到其它节点，并创建缓存
```shell
scp /etc/yum.repos.d/ambari.repo 192.168.135.4:/etc/yum.repos.d/
scp /etc/yum.repos.d/ambari.repo 192.168.135.5:/etc/yum.repos.d/
scp /etc/yum.repos.d/hdp.repo 192.168.135.6:/etc/yum.repos.d/
```
在各个节点创建缓存：

```shell
yum clean all
yum makecache fast
```

## 初始化环境
### 各个节点安装java-1.8.0-openjdk
```shell
 yum install java-1.8.0-openjdk -y
 ```
### 解析主机名
```shell
 echo "192.168.135.4 node2" >> /etc/hosts
 echo "192.168.135.5 node3" >> /etc/hosts
 echo "192.168.135.6 node4" >> /etc/hosts
 ```
### 创建主机信任关系
```shell
ssh-keygen
ssh-copy-id node-1
ssh-copy-id node-2
ssh-copy-id node-3
```

### 安装配置数据库
```shell
yum install mariadb-server -y
systemctl enable mariadb
systemctl start mariadb
```
## 创建数据库
```shell
mysql
```
```sql
set password=password('123456');
grant all on *.* to root@localhost identified by '123456';
grant all on *.* to root@'%' identified by '123456';
create database ambari default character set utf8;
grant all on ambari.* to ambari@localhost identified by 'bigdata';
grant all on ambari.* to ambari@'%' identified by 'bigdata';
create database hive default character set utf8;
grant all on hive.* to hive@localhost identified by 'hive';
grant all on hive.* to hive@'%' identified by 'hive';
```

## 安装Amabri服务
### 在node-1上安装ambari-server,并启动配置向导
```shell
yum install ambari-server -y
ambari-server setup
```


### 按照配置向导信息、配置用户、java_home

```shell
ambari-server setup
Using python  /usr/bin/python
Setup ambari-server
Checking SELinux...
SELinux status is 'disabled'
Customize user account for ambari-server daemon [y/n] (n)? y
Enter user account for ambari-server daemon (root):ambari  
Adjusting ambari-server permissions and ownership...
Checking firewall status...
Checking JDK...
[1] Oracle JDK 1.8 + Java Cryptography Extension (JCE) Policy Files 8
[2] Oracle JDK 1.7 + Java Cryptography Extension (JCE) Policy Files 7
[3] Custom JDK
==============================================================================
Enter choice (1): 3
WARNING: JDK must be installed on all hosts and JAVA_HOME must be valid on all hosts.
WARNING: JCE Policy files are required for configuring Kerberos security. If you plan to use Kerberos,please make sure JCE Unlimited Strength Jurisdiction Policy Files are valid on all hosts.
Path to JAVA_HOME: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64/jre  # 填写java_home
Validating JDK on Ambari Server...done.
Checking GPL software agreement...
GPL License for LZO: https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html
Enable Ambari Server to download and install GPL Licensed LZO packages [y/n] (n)? n
Completing setup...
Configuring database...
Enter advanced database configuration [y/n] (n)? y  
Configuring database...
==============================================================================
Choose one of the following options:
[1] - PostgreSQL (Embedded)
[2] - Oracle
[3] - MySQL / MariaDB
[4] - PostgreSQL
[5] - Microsoft SQL Server (Tech Preview)
[6] - SQL Anywhere
[7] - BDB
==============================================================================
Enter choice (1): 3
Hostname (localhost): 
Port (3306): 
Database name (ambari): 
Username (ambari): 
Enter Database Password (bigdata): 
Configuring ambari database...
WARNING: Before starting Ambari Server, you must copy the MySQL JDBC driver JAR file to /usr/share/java and set property "server.jdbc.driver.path=[path/to/custom_jdbc_driver]" in ambari.properties.
Press <enter> to continue.
```
### 到上面一步时，根据提示上传mysql的jdbc驱动，并修改配置文件，指定jdbc驱动文件位置：
```shell
cd /usr/share/java
ll
tar xf mysql-connector-java-5.1.45.tar.gz 
mv mysql-connector-java-5.1.45/mysql-connector-java-5.1.45-bin.jar ./
# 修改配置文件：
vim /etc/ambari-server/conf/ambari.properties 
```
```conf
server.jdbc.driver.path=/usr/share/java/mysql-connector-java-5.1.45-bin.jar
```
配置完成后继续，会出现如下提示：

Press <enter> to continue.
Configuring remote database connection properties...
WARNING: Before starting Ambari Server, you must run the following DDL against the database to create the schema: /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql
Proceed with configuring remote database connection properties [y/n] (y)? 
### 出现上述提示时，根据信息导入数据库
```
mysql -uroot -p ambari < /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql
```
### 启动服务
```shell
ambari-server start
```
