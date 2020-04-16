---
title: 搭建基于 HDFS 碎片文件存储服务
date: 2018-12-10
categories:
  - linux
tags:
  - hadoop
---

搭建基于 HDFS 碎片文件存储服务
<!--more-->

## 准备 Java 环境
```bash
yum -y install java-1.8.0-openjdk*
java -version
vim /etc/profile
```

```profile
export JAVA_HOME=/usr/lib/jvm/jre-1.8.0-openjdk
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

```bash
source /etc/profile
```

## 准备 HDFS 环境
### 配置 SSH
先后执行如下两行命令，配置 SSH 以无密码模式登陆：
```bash
ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
```
接着可以验证下，在不输入密码的情况下，应该能使用 ssh 命令成功连接本机：

### 安装 Hadoop

```bash
# 创建 /data/hadoop 目录，然后进入该目录：
mkdir -p /data/hadoop && cd $_
# 下载并解压 Hadoop：
wget http://archive.apache.org/dist/hadoop/core/hadoop-2.7.1/hadoop-2.7.1.tar.gz
tar -zxvf hadoop-2.7.1.tar.gz
mv hadoop-2.7.1 hadoop && mv $_ /usr/local/
```

修改 Hadoop 环境配置文件
```bash
vim /usr/local/hadoop/etc/hadoop/hadoop-env.sh
```
```conf
export JAVA_HOME=/usr/lib/jvm/jre-1.8.0-openjdk
```
查看下 Hadoop 的版本：
```bash
/usr/local/hadoop/bin/hadoop version
```

## 修改 Hadoop 配置
由于我们的实践环境是在单机下进行的，所以此处把 Hadoop 配置为伪分布式模式。  
新建若干临时文件夹，我们在后续的配置中以及 HDFS 和 Hadoop 的启动过程中会使用到这些文件夹：

```bash
cd /usr/local/hadoop && mkdir -p tmp dfs/name dfs/data
```

修改 HDFS 配置文件 core-site.xml

```bash
vim /usr/local/hadoop/etc/hadoop/core-site.xml
```

```xml
<configuration>  
    <property>  
        <name>hadoop.tmp.dir</name>  
        <value>/usr/local/hadoop/tmp</value>
    </property>  
    <property>  
        <name>fs.defaultFS</name>  
        <value>hdfs://localhost:9000</value>
    </property>  
</configuration>
```
修改 HDFS 配置文件 hdfs-site.xml

```
vim /usr/local/hadoop/etc/hadoop/hdfs-site.xml
```

```xml
<configuration>    
    <property>    
        <name>dfs.replication</name>    
        <value>1</value>    
    </property>    
    <property>    
        <name>dfs.namenode.name.dir</name>    
        <value>file:/usr/local/hadoop/dfs/name</value>    
    </property>    
    <property>    
        <name>dfs.datanode.data.dir</name>    
        <value>file:/usr/local/hadoop/dfs/data</value>    
    </property>    
    <property>
        <name>dfs.permissions</name>    
        <value>false</value>    
    </property>    
</configuration>
```

修改 HDFS 配置文件 yarn-site.xml

```bash
vim /usr/local/hadoop/etc/hadoop/yarn-site.xml
```

```xml
<configuration>  
    <property>  
        <name>mapreduce.framework.name</name>  
        <value>yarn</value>  
    </property>  

    <property>  
        <name>yarn.nodemanager.aux-services</name>  
        <value>mapreduce_shuffle</value>  
    </property>  
</configuration>
```

启动 Hadoop
```bash
cd /usr/local/hadoop/bin/
# 对 HDFS 文件系统进行格式化：
./hdfs namenode -format
# 接着进入如下目录
cd /usr/local/hadoop/sbin/
# 先后执行如下两个脚本启动 Hadoop
./start-dfs.sh
./start-yarn.sh
# 验证 Hadoop 是否启动成功：
jps
```

在浏览器中访问如下链接，应该能正常访问：

http://<您的 IP 地址>:50070/explorer.html#/
接下来，我们实践下如何将碎片文件存储到 HDFS 中。


## 存储碎片文件

### 准备碎片文件
```bash
# 创建目录用于存放碎片文件，进入该目录：

mkdir -p /data/file && cd /data/file
# 新建一批碎片文件到该目录下
i=1; while [ $i -le 99 ]; do name=`printf "test%02d.txt"  $i`; touch "$name"; i=$(($i+1)); done
ls
```

### 将碎片文件存储在 HDFS 中
```bash
# 在 HDFS 上新建目录
/usr/local/hadoop/bin/hadoop fs -mkdir /dest
```
此时，在浏览器是访问如下链接，可以看到 /dest 目录已创建，但是暂时还没有内容：

http://<您的 IP 地址>:50070/explorer.html#/dest  

### 上传碎片文件  

```bash
groupadd supergroup
usermod -a -G supergroup root
# 将之前创建的碎片文件上传到 HDFS 中：
cd /data/file && /usr/local/hadoop/bin/hadoop fs -put *.txt /dest
/usr/local/hadoop/bin/hadoop fs -ls /dest
# 在上传之前，我们需先创建 HDFS 用户组 supergroup，然后将 root 用户添加到该组中，否则会因权限问题页报告异常。
```

### 访问服务
在浏览器是访问如下链接，可以看到 /dest 目录的文件内容：
```
http://<您的 IP 地址>:50070/explorer.html#/dest
```
