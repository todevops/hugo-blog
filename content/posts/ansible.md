---
title: Ansible 自动化运维实战
date: 2018-12-17
categories:
  - linux
tags:
  - ansible
  - python
---

Ansible 自动化运维实战
<!--more-->

## Ad-Hoc命令简介
### 并行和Shell命令
让我们使用Ansible的命令行工具重启atlanta的所有Web服务器，一次10个。首先，让我们设置SSH代理，以便它能记住我们的凭据：
```shell
ssh-agent bash
ssh-add ~/.ssh/id_rsa
```

playbook
```


[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/OS/OSRELEASE/$basearch/
gpgcheck=0
enabled=1
```
