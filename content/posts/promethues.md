---
title: Prometheus + Grafana
date: 2019-02-27 19:59:14
categories:
  - monitor
tags:
  - linux
  - prometheus
  - grafana
---

开源监控平台 Prometheus
<!--more-->

{% asset_img prometheus.jpg prometheus %}

## 安装 GO
```shell
[root@master ~]# yum -y install go
[root@mysqlnode05 go]# go version
```

## 安装 prometheus
```shell
[root@master ~]# tar -zxvf prometheus-2.7.1.linux-amd64.tar.gz
[root@master ~]# mv prometheus-2.7.1.linux-amd64 /appdata/prometheus
[root@master ~]# cd /appdata/prometheus/
# 修改prometheus.yml文件，配置targets其中的IP和端口则是对应的exporter的监听端口
[root@master ~]# vim prometheus.yml
```
```yml
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    static_configs:
    - targets: ['192.168.41.131:9090']
  - job_name: 'master'
    static_configs:
      - targets: ['192.168.41.131:9100']
        labels:
          instance: master
  - job_name: 'slave'
    static_configs:
      - targets: ['192.168.41.132:9100']
        labels:
          instance: slave
  - job_name: 'mysql-master'
    static_configs:
      - targets: ['192.168.41.131:9104']
        labels:
          instance: master
  - job_name: 'mysql-slave'
    static_configs:
      - targets: ['192.168.41.132:9104']
        labels:
          instance: slave
```

```shell
# 启动并通过web访问
[root@master ~]# /appdata/prometheus/prometheus --config.file /appdata/prometheus/prometheus.yml >> /appdata/prometheus/prometheus.log 2>&1 &
```

## 安装 mysqld_exporter
```shell
[root@master ~]# tar -zxvf mysqld_exporter-0.11.0.linux-amd64.tar.gz
[root@master ~]# cp mysqld_exporter-0.11.0.linux-amd64/mysqld_exporter /appdata/prometheus/
[root@master ~]# mysql -uroot -p
# 创建专用用户
mysql> create user 'prometheus'@'%' identified by 'xxxxxx';
mysql> grant REPLICATION CLIENT,PROCESS,SELECT ON *.* TO 'prometheus'@'%';
mysql> flush privileges;
# 创建exporter配置文件
[root@master ~]# vim /appdata/prometheus/.my.cnf
```

```conf
[client]
user=mysql_prome
password=Aa123456789
```

```shell
# 启动mysqld_exporter
[root@master ~]# /appdata/prometheus/mysqld_exporter --config.my-cnf /appdata/prometheus/.my.cnf >> /appdata/prometheus/mysqld_exporter.log 2>&1 &
```

## 安装 node_exporter
```shell
[root@master ~]# tar -zxvf node_exporter-0.17.0.linux-amd64.tar.gz
[root@master ~]# cp node_exporter-0.17.0.linux-amd64/node_exporter /appdata/prometheus/
# 启动node_exporter
[root@master ~]# /appdata/prometheus/node_exporter >> /appdata/prometheus/node_exporter.log 2>&1 &
```

## 查看 target status
浏览器访问 http://192.168.31.131:9090

{% asset_img prometheus02.png prometheus02 %}

## 安装 grafana
```shell
[root@master ~]# wget https://dl.grafana.com/oss/release/grafana-6.0.0-1.x86_64.rpm 
[root@master ~]# yum -y install grafana-6.0.0-1.x86_64.rpm
# 启动grafana
[root@master ~]# systemctl enable grafana-server
[root@master ~]# systemctl start grafana-server
```
## 添加数据源
浏览器访问 http://192.168.31.131:3000

{% asset_img prometheus03.png prometheus03 %}