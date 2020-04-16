---
title: Filebeat 指南
date: 2019-11-19 00:56:12
tags:
---

使用 filebeat 进行日志抓取
<!--more-->


## 配置 Filebeat
### 日志输入
使用 log 输入从日志文件中读取行。

```yaml
filebeat.inputs:
- type: log
  paths:
    - /var/log/messages
    - /var/log/*.log
```

您可以配置额外的设置（例如 fields，include_lines，exclude_lines，multiline 等），从这些文件中获取行。您指定的选项将应用于此输入收集的所有文件。  
要将不同的配置设置应用于不同的文件，您需要定义多个输入节：

```yaml
filebeat.inputs:
- type: log 
  paths:
    - /var/log/system.log
    - /var/log/wifi.log
- type: log 
  paths:
    - "/var/log/apache2/*"
  fields:
    apache: true
  fields_under_root: true
```