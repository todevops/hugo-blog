---
title: gitlab-ce 安装指南
date: 2019-01-03
lastmod: 2019-12-20
categories: ["tools"]
tags:
  - linux
  - git
  - gitlab
---

gitlab 安装使用指南
<!--more-->

## 更新软件包
```
yum update -y
```

## 配置防火墙
```bash
[root@gitlab ~]# vim /etc/sysctl.conf
```

```conf
net.ipv4.ip_forward = 1
```
+ 启用并启动防火墙

```bash
[root@gitlab ~]# systemctl enable firewalld
[root@gitlab ~]# systemctl start firewalld
```

+ 放通 HTTP、HTTPS

```bash
[root@gitlab ~]# firewall-cmd --add-service=http --permanent
[root@gitlab ~]# firewall-cmd --add-service=https --permanent
```

+ 重载入防火墙

```bash
[root@gitlab ~]# systemctl reload firewalld
```

## 安装 postfix
GitLab 需要使用 postfix 来发送邮件。当然，也可以使用 SMTP 服务器
```bash
[root@gitlab ~]# vim /etc/postfix/main.cf
```
```conf
# inet_protocols = all
inet_protocols = ipv4
```

+ 启用并启动 postfix

```bash
[root@gitlab ~]# systemctl restart postfix
```
## 配置 swap
由于 GitLab 较为消耗资源，我们需要先创建交换分区，以降低物理内存的压力。
在实际生产环境中，如果服务器配置够高，则不必配置交换分区。

+ 新建 2 GB 大小的交换分区

```bash
[root@gitlab ~]# dd if=/dev/zero of=/root/swapfile bs=1M count=2048
```

+ 格式化为交换分区文件并启用

```bash
[root@gitlab ~]# mkswap /root/swapfile
[root@gitlab ~]# swapon /root/swapfile
```

+ 添加自启用

```bash
[root@gitlab ~]# vim /etc/fstab
```
```
/root/swapfile swap swap defaults 0 0
```
## 安装 GitLab

+ 将软件源修改为国内源

```bash
[root@gitlab ~]# vim /etc/yum.repos.d/gitlab-ce.repo
```
```conf
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1
```

+ 安装 GitLab

```bash
[root@gitlab ~]# yum makecache
[root@gitlab ~]# yum install -y gitlab-ce
```
```
       *.                  *.
      ***                 ***
     *****               *****
    .******             *******
    ********            ********
   ,,,,,,,,,***********,,,,,,,,,
  ,,,,,,,,,,,*********,,,,,,,,,,,
  .,,,,,,,,,,,*******,,,,,,,,,,,,
      ,,,,,,,,,*****,,,,,,,,,.
         ,,,,,,,****,,,,,,
            .,,,***,,,,
                ,*,.



     _______ __  __          __
    / ____(_) /_/ /   ____ _/ /_
   / / __/ / __/ /   / __ `/ __ \
  / /_/ / / /_/ /___/ /_/ / /_/ /
  \____/_/\__/_____/\__,_/_.___/

  Thank you for installing GitLab!
```

## 配置 HTTPS 证书
+ 生成 key 文件

```bash
[root@gitlab ~]# openssl genrsa -out "/etc/gitlab/ssl/gitlab.example.com.key"
```
输出：
```
Generating RSA private key, 2048 bit long modulus
.........................................................................................................+++
..............................................................+++
e is 65537 (0x10001)
```

+ 生成 csr 文件

```bash
[root@gitlab ssl]# openssl req -new -key "/etc/gitlab/ssl/gitlab.example.com.key" -out "/etc/gitlab/ssl/gitlab.example.com.csr"
```
输出：
```
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:cn
State or Province Name (full name) []:sz
Locality Name (eg, city) [Default City]:sz
Organization Name (eg, company) [Default Company Ltd]:example
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:gitlab.example.com
Email Address []:admin@example.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:password
An optional company name []:
```

+ 生成 crt 文件 

```bash
[root@gitlab ~]# openssl x509 -req -days 365 -in "/etc/gitlab/ssl/gitlab.example.com.csr" -signkey "/etc/gitlab/ssl/gitlab.example.com.key" -out "/etc/gitlab/ssl/gitlab.example.com.crt"
```
输出：
```
Signature ok
subject=/C=cn/ST=sz/L=sz/O=example/CN=gitlab.example.com/emailAddress=admin@example.com
Getting Private key
```

+ 生成 dhparams.pem

```bash
[root@gitlab ssl]# openssl dhparam -out /etc/gitlab/ssl/dhparams.pem 2048
```

## 初始化 GitLab

+ 配置 GitLab

```bash
[root@gitlab ~]# vim /etc/gitlab/gitlab.rb
```
```conf
external_url 'http://gitlab.example.com'
nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.example.com.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.example.com.key"
nginx['ssl_dhparam'] = "/etc/gitlab/ssl/dhparams.pem"
```

+ 初始化 GitLab

```bash
[root@gitlab ~]# gitlab-ctl reconfigure
```

+ 修改 nginx 配置

```bash
[root@gitlab ~]# vim /var/opt/gitlab/nginx/conf/gitlab-http.conf
```

```conf
server {
  listen *:80;

  rewrite ^(.*)$ https://$host$1 permanent; # 新增

  server_name gitlab.example.com;
  server_tokens off; ## Don't show the nginx version number, a security best practice


  location / {
    return 301 https://gitlab.example.com:443$request_uri;
  }

  access_log  /var/log/gitlab/nginx/gitlab_access.log gitlab_access;
  error_log   /var/log/gitlab/nginx/gitlab_error.log;
}
```

+ 重启 gitlab

```bash
[root@gitlab ~]# gitlab-ctl restart
```
