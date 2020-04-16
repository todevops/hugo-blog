---
title: Docker /etc/docker/daemon.json 文件的参数
date: 2019-11-20 17:25:46
tags:
---

Docker /etc/docker/daemon.json 文件的参数清单
<!--more-->


```json
{
 "api-cors-header":"", 
 "authorization-plugins":[],
 "bip": "",
 "bridge":"",
 "cgroup-parent":"",
 "cluster-store":"",
 "cluster-store-opts":{},
 "cluster-advertise":"",
 "debug": true,
 "default-gateway":"",
 "default-gateway-v6":"",
 "default-runtime":"runc",
 "default-ulimits":{},
 "disable-legacy-registry":false,
 "dns": ["192.168.1.1"],        # 设定容器DNS的地址，在容器的 /etc/resolv.conf文件中可查看。
 "dns-opts": [],                # 容器 /etc/resolv.conf 文件，其他设置
 "dns-search": [],              # 设定容器的搜索域，当设定搜索域为 .example.com 时，在搜索一个名为 host 的 主机时，DNS不仅搜索host，还会搜索host.example.com 。 注意：如果不设置， Docker 会默认用主机上的 /etc/resolv.conf 来配置容器。
 
"exec-opts": [],
 "exec-root":"",
 "fixed-cidr":"",
 "fixed-cidr-v6":"",
 "data-root":"/var/lib/docker",     ＃ Docker运行时使用的根路径,根路径下的内容稍后介绍，默认/var/lib/docker
 "group": "",                       # Unix套接字的属组,仅指/var/run/docker.sock
 "hosts": [],                       # 设置容器hosts
 "icc": false,
 "insecure-registries": [],         # 配置docker的私库地址
 "ip":"0.0.0.0",
 "iptables": false,
 "ipv6": false,
 "ip-forward": false,               # 默认true, 启用 net.ipv4.ip_forward ,进入容器后使用 sysctl -a | grepnet.ipv4.ip_forward 查看
 
  "ip-masq":false,
 "labels":["nodeName=node-121"],   # docker主机的标签，很实用的功能,例如定义：–label nodeName=host-121
 
 "live-restore": true,
 "log-driver":"",
 "log-level":"",
 "log-opts": {},
 "max-concurrent-downloads":3,
 "max-concurrent-uploads":5,
 "mtu": 0,
 "oom-score-adjust":-500,
 "pidfile": "",                    # Docker守护进程的PID文件
 "raw-logs": false,
 "registry-mirrors":["xxxx"],      # 镜像加速的地址，增加后在 docker info 中可查看。
 "runtimes": {
 "runc": {
 "path": "runc"
 },
 "custom": {
 "path":"/usr/local/bin/my-runc-replacement",
 "runtimeArgs": [
 "--debug"
 ]
 }
 },
 "selinux-enabled": false,          # 默认 false，启用 selinux 支持
 
 "storage-driver":"",
 "storage-opts": [],
 "swarm-default-advertise-addr":"",
 "tls": true,                       # 默认 false, 启动TLS认证开关
 "tlscacert": "",                   # 默认 ~/.docker/ca.pem，通过CA认证过的的certificate文件路径
 "tlscert": "",                     # 默认 ~/.docker/cert.pem，TLS的certificate文件路径
 "tlskey": "",                      # 默认 ~/.docker/key.pem，TLS的key文件路径
 "tlsverify": true,                 # 默认 false，使用TLS并做后台进程与客户端通讯的验证
 "userland-proxy":false,
 "userns-remap":""
 }
```