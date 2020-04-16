---
title: Jenkins 入门指南
date: 2019-12-20
categories:
  - linux
tags:
  - jenkins
  - CI/CD
---

简单的介绍 Jenkins 的安装与使用
<!--more-->

## Jenkins 安装
### 方法1. 使用 yum 安装（Redhat/CentOS/Fedora）
```bash
[root@jenkins ~]# wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
[root@jenkins ~]# rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

[root@jenkins ~]# yum -y install jenkins

[root@jenkins ~]# systemctl start jenkins
[root@jenkins ~]# systemctl status jenkins
● jenkins.service - LSB: Jenkins Automation Server
   Loaded: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)
   Active: active (running) since 五 2019-12-20 10:37:48 CST; 6s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 8910 ExecStart=/etc/rc.d/init.d/jenkins start (code=exited, status=0/SUCCESS)
    Tasks: 48
   Memory: 442.5M
   CGroup: /system.slice/jenkins.service
           └─8934 /etc/alternatives/java -Dcom.sun.akuma.Daemon=daemonized -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war --logfile=/var/log/jen...

12月 20 10:37:47 jenkins systemd[1]: Starting LSB: Jenkins Automation Server...
12月 20 10:37:47 jenkins runuser[8915]: pam_unix(runuser:session): session opened for user jenkins by (uid=0)
12月 20 10:37:48 jenkins runuser[8915]: pam_unix(runuser:session): session closed for user jenkins
12月 20 10:37:48 jenkins jenkins[8910]: Starting Jenkins [  确定  ]
12月 20 10:37:48 jenkins systemd[1]: Started LSB: Jenkins Automation Server.
```
- 通过 `<IP>:<PORT>` 访问 Jenkins

![](/resources/jenkins/jenkins_01.png)

- 获取 initialAdminPassword

```bash
[root@jenkins ~]# cat /var/lib/jenkins/secrets/initialAdminPassword
```


### 方法2. 使用 Docker 安装

```bash
[root@jenkins ~]# docker pull jenkinsci/blueocean

[root@jenkins ~]# docker run -u root -p 8080:8080 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v "$HOME":/home --name jenkins -d jenkinsci/blueocean
```

- 通过 `<IP>:<PORT>` 访问 Jenkins

- 获取 initialAdminPassword

```bash
# 通过 docker logs 获取
[root@jenkins ~]# docker logs jenkins
...

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

95538a82714644d5a4b5d84c6f998a4f

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

...
```

```bash
# 进入容器内部获取 initialAdminPassword
[root@jenkins ~]# docker exec -it jenkins /bin/bash
bash-4.4# cat /var/jenkins_home/secrets/initialAdminPassword
```

### 安装推荐插件

![](/resources/jenkins/jenkins_02.png)
![](/resources/jenkins/jenkins_03.png)

### 设置用户
![](/resources/jenkins/jenkins_04.png)

### 安装完成

![](/resources/jenkins/jenkins_05.png)
![](/resources/jenkins/jenkins_06.png)

