---
title: Tomcat 配置与优化
date: 2018-12-01
categories:
  - linux
tags:
  - linux
  - tomcat
---

Tomcat 配置与优化
<!--more-->

## 1. JVM 配置
### 添加 tomcat 管理员

```
[root@server ~]# vim /tomcat/conf/tomcat-users.xml
```
```xml
<role rolename="manager-gui"/>
<role rolename="admin-gui"/>
<user username="manager" password="manager" roles="admin-gui,manager-gui"/>
```

### 修改 JVM 虚拟内存

![](images/jvm.png)

```sh
JAVA_OPTS="-server -Xms1024m -Xmx1024m -Xmn256m -XX:PermSize=512m -XX:MaxPermSize=1024m -XX:ReservedCodeCacheSize=256M -Djava.awt.headless=true -Dfile.encoding=utf-8"
```
+ 堆设置：  
-Xms=n   初始堆大小  
-Xmx=n   最大堆大小  
-XX:NewSize=n   设置年轻代大小  
-XX:NewRatio=n  设置年轻代和年老代的比值.如:为3,表示年轻代与年老代比值为1:3,年轻代占整个年轻代年老代和的1/4  
-XX:SurvivorRatio=n 年轻代中Eden区与两个Survivor区的比值.注意Survivor区有两个.如:3,表示Eden:Survivor=3:2,一个Survivor区占整个年轻代的1/5  
-XX:MaxPermSize=n   设置持久代大小  
+ 收集器设置：  
-XX:+UseSerialGC    设置串行收集器 
-XX:+UseParallelGC  设置并行收集器 
-XX:+UseParalledlOldGC  设置并行年老代收集器 
-XX:+UseConcMarkSweepGC 设置并发收集器
+ 垃圾回收统计信息：  
-XX:+PrintGC  
-XX:+PrintGCDetails  
-XX:+PrintGCTimeStamps  
-Xloggc:filename  
+ 并行收集器设置：  
-XX:ParallelGCThreads=n 设置并行收集器收集时使用的CPU数.并行收集线程数.  
-XX:MaxGCPauseMillis=n  设置并行收集最大暂停时间  
-XX:GCTimeRatio=n   设置垃圾回收时间占程序运行时间的百分比.公式为1/(1+n)并发收集器设置   
-XX:+CMSIncrementalMode 设置为增量模式.适用于单CPU情况.  
-XX:ParallelGCThreads=n 设置并发收集器年轻代收集方式为并行收集时,使用的CPU数.并行收集线程数

## 2. apr 安装
### 安装依赖

```
[root@server ~]# yum -y install gcc expat-devel
```
### 安装 [apr]()

```
[root@server ~]# mkdir /usr/local/apr
[root@server apr-1.6.3]# ./configure --prefix=/usr/local/apr
[root@server apr-1.6.3]# make && make install
```

### 安装 [apr-util]()

```
[root@server apr-util-1.6.1]# ./configure --prefix=/usr/local/apr --with-apr=/usr/local/apr
[root@server apr-util-1.6.1]# make && make install
```

### 安装 [tomcat-native]()

```
[root@server ~]# cd tomcat-native-1.2.17-src/native/
[root@server native]# ./configure --prefix=/usr/local/apr --with-apr=/usr/local/apr --with-java-home=/usr/java/jdk1.7.0_80/
[root@server native]# make && make install
```

### 修改 tomcat 配置

```
[root@server ~]# vim /tomcat/bin/catalina.sh
```
```sh
# 添加 CATALINA_OPTS
CATALINA_OPTS="-Djava.library.path=/usr/local/apr/lib"
```

```
[root@server ~]# vim /tomcat/conf/server.xml
```
```xml
<Connector port="8001"
    protocol="org.apache.coyote.http11.Http11AprProtocol"
    executor="tomcatThreadPool"
    maxThreads="1000"
    enableLookups="false"
    acceptCount="1000"
    connectionTimeout="30000"
    redirectPort="9100"
    maxPostSize="8388608"
    maxParameterCount="40000"
    disableUploadTimeout="true"
    URIEncoding="UTF-8"/>
```

## 3. SSLEngine Error
### 查看 tomcat 日志，出现 SSLEngine Error

org.apache.catalina.core.AprLifecycleListener.lifecycleEvent Failed to initialize the SSLEngine.
{:.error}

### 解决办法

```
[root@server ~]# vim /tomcat/conf/server.xml
```
```xml
<Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="off" />
```