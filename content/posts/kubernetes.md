---
title: Kubernetes 离线安装
date: 2019-10-15 01:31:26
categories:
  - linux
tags:
  - Kubernetes
  - docker
---

Kubernetes 离线安装
<!--more-->

IPADDRESS | HOSTNAME | ENVIROMENT | KERNEL
---|---|---|---
192.168.33.50 | k8smaster | CentOS 7 | 5.3.7-1.el7.elrepo.x86_64
192.168.33.51 | k8snode01 | CentOS 7 | 5.3.7-1.el7.elrepo.x86_64
192.168.33.52 | k8snode01 | CentOS 7 | 5.3.7-1.el7.elrepo.x86_64


### 1. 设置域名解析
```bash
cat << EOF >> /etc/hosts
192.168.33.50 k8smaster
192.168.33.51 k8snode01
192.168.33.52 k8snode01
EOF
```

### 2. 安装相关依赖包

```
yum -y install conntrack ipvsadm sysstat wget git
```

### 3. 设置防火墙、SELINUX

```
systemctl disable firewalld && systemctl stop firewalld
yum -y install iptables-services
systemctl enable iptables && systemctl start iptables
iptables -F
service iptables save

setenforce 0
vim /etc/selinux/config

SELINUX=disabled
```

### 4. 设置时区调整系统时间

```
timedatectl set-timezone Asia/Shanghai
# 将当前时间写入硬件时钟
timedatectl set-local-rtc 0
# 重启依赖于系统时间的服务
systemctl restart rsyslog
systemctl restart crond
```

### 5. 关闭系统不需要的服务

```
systemctl disable postfix && systemctl stop postfix
```

### 6. 设置rsyslogd和systemd journald

```
# 创建持久化保存日志的目录
mkdir /var/log/journal
mkdir /etc/systemd/journald.conf.d
cat << EOF > /etc/systemd/journald.conf.d/99-prophet.conf
[Journal]

# 持久化保存到磁盘
Storage=persistent

# 压缩历史日志
Compress=yes
SyncIntervalSec=5m
RateLimitInterval=30s
RateLimitBurst=1000

# 最大占用空间
SystemMaxUse=5G

# 单个日志文件最大100M
SystemMaxFileSize=100M

# 日志保存时间2周
MaxRetentionSec=2week

# 不将日志转发到syslog
ForwardToSyslog=no
EOF

systemctl restart systemd-journald
```

### 7. 升级系统内核

```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
yum install https://www.elrepo.org/elrepo-release-7.0-4.el7.elrepo.noarch.rpm
yum --enablerepo=elrepo-kernel install kernel-ml -y
grub2-mkconfig -o /boot/grub2/grub.cfg

# 设置默认启动为新内核
sed -i 's/GRUB_DEFAULT=saved/GRUB_DEFAULT=0/g' /etc/default/grub

# 重启
reboot
```

### 8. 关闭SWAP

```
swapoff -a
sed -i 's|^/swapfile|#/swapfile|g' /etc/fstab
```

### 9. 调整内核参数
```
cat << EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv4.ip_forward = 1
EOF

sysctl -p /etc/sysctl.d/k8s.conf
```

## 10. kube-proxy开启ipvs的前置条件

```bash
modprobe br_netfilter

cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF

chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules

# 查看是否已经正确加载所需的内核模块
lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

## 11. 安装Docker
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装 Docker-CE
sudo yum makecache fast

yum install -y docker-ce

systemctl enable docker && systemctl start docker

# 配置daemon
cat << EOF > /etc/docker/daemon.json
{
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "100m"
    }
}
EOF

mkdir -p /etc/systemd/system/docker.service.d

# 重启docker服务
systemctl daemon-reload && systemctl restart docker
```

### 12. 安装kubeadm和kubelet（1.4.1-0）

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# yum remove -y kubelet kubeadm kubectl
yum -y install kubelet-1.14.1-0.x86_64 kubeadm-1.14.1-0.x86_64 kubectl-1.14.1-0.x86_64
systemctl enable kubelet && systemctl start kubelet
```

## 13. 使用kubeadm init初始化集群

+ 下载离线安装所需的docker image（需要科学上网）
```bash
kubeadm config images list
```

+ 所需镜像清单

```
k8s.gcr.io#coredns#1.3.1.tar
k8s.gcr.io#etcd#3.3.10.tar
k8s.gcr.io#kube-apiserver#v1.14.1.tar
k8s.gcr.io#kube-controller-manager#v1.14.1.tar
k8s.gcr.io#kube-proxy#v1.14.1.tar
k8s.gcr.io#kube-scheduler#v1.14.1.tar
k8s.gcr.io#pause#3.1.tar
quay.io#coreos#flannel#v0.11.0-amd64.tar
```

+ 导入docker image

```
for i in `ls -1`;
do
docker load < $i
done
```

------

### 14. k8smaster初始化k8s集群

```
kubeadm config print init-defaults > kubeadm-config.yaml
```

```
apiVersion: kubeadm.k8s.io/v1beta1
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.33.50
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8smaster
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta1
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: ""
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: v1.14.1
networking:
  dnsDomain: cluster.local
  podSubnet: "10.244.0.0/16"
  serviceSubnet: 10.96.0.0/12
scheduler: {}
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
featureGates:
  SupportIPVSProxyMode: true
mode: ipvs
```

```
kubeadm init --config=kubeadm-config.yaml | tee kubeadm-init.log

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

+ 查看节点状态

```
kubectl get node

NAME        STATUS     ROLES    AGE    VERSION
k8smaster   NotReady   master   2m2s   v1.14.1
```


```
kubectl get cs

NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                  
controller-manager   Healthy   ok                  
etcd-0               Healthy   {"health":"true"}
```

```
kubectl get pod -n kube-system -o wide

NAME                                READY   STATUS    RESTARTS   AGE     IP              NODE        NOMINATED NODE   READINESS GATES
coredns-fb8b8dccf-4sr7j             0/1     Pending   0          10m     <none>          <none>      <none>           <none>
coredns-fb8b8dccf-9lxf8             0/1     Pending   0          10m     <none>          <none>      <none>           <none>
etcd-k8smaster                      1/1     Running   0          10m     192.168.33.50   k8smaster   <none>           <none>
kube-apiserver-k8smaster            1/1     Running   0          9m58s   192.168.33.50   k8smaster   <none>           <none>
kube-controller-manager-k8smaster   1/1     Running   0          9m56s   192.168.33.50   k8smaster   <none>           <none>
kube-flannel-ds-amd64-r68r5         1/1     Running   0          14s     192.168.33.50   k8smaster   <none>           <none>
kube-proxy-gc9ms                    1/1     Running   0          10m     192.168.33.50   k8smaster   <none>           <none>
kube-scheduler-k8smaster            1/1     Running   0          10m     192.168.33.50   k8smaster   <none>           <none>
```

### 15. k8smaster安装flannel

```bash
mkdir -p ~/k8s/
cd ~/k8s
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f  kube-flannel.yml
```


### 16. k8snode01、k8snode02加入集群
```
kubeadm join 192.168.33.50:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:91dd4c335d92e948075b8980f78beb112ebf7a23fffe156b64f90097b88d1612
# 查看加入语句
kubeadm token create --print-join-command
```

```
kubectl get pod -n kube-system -o wide
NAME                                READY   STATUS    RESTARTS   AGE     IP              NODE        NOMINATED NODE   READINESS GATES
coredns-fb8b8dccf-4sr7j             1/1     Running   0          14m     10.244.0.2      k8smaster   <none>           <none>
coredns-fb8b8dccf-9lxf8             1/1     Running   0          14m     10.244.0.3      k8smaster   <none>           <none>
etcd-k8smaster                      1/1     Running   0          13m     192.168.33.50   k8smaster   <none>           <none>
kube-apiserver-k8smaster            1/1     Running   0          13m     192.168.33.50   k8smaster   <none>           <none>
kube-controller-manager-k8smaster   1/1     Running   0          13m     192.168.33.50   k8smaster   <none>           <none>
kube-flannel-ds-amd64-m6mtp         1/1     Running   0          2m11s   192.168.33.51   k8snode01   <none>           <none>
kube-flannel-ds-amd64-qgmwp         1/1     Running   0          2m5s    192.168.33.52   k8snode02   <none>           <none>
kube-flannel-ds-amd64-r68r5         1/1     Running   0          3m47s   192.168.33.50   k8smaster   <none>           <none>
kube-proxy-9tpgd                    1/1     Running   0          2m5s    192.168.33.52   k8snode02   <none>           <none>
kube-proxy-gc9ms                    1/1     Running   0          14m     192.168.33.50   k8smaster   <none>           <none>
kube-proxy-tdz9k                    1/1     Running   0          2m11s   192.168.33.51   k8snode01   <none>           <none>
kube-scheduler-k8smaster            1/1     Running   0          13m     192.168.33.50   k8smaster   <none>           <none>
```

```
kubectl get node

NAME        STATUS   ROLES    AGE     VERSION
k8smaster   Ready    master   16m     v1.14.1
k8snode01   Ready    <none>   3m54s   v1.14.1
k8snode02   Ready    <none>   3m48s   v1.14.1
```

### 17. 安装kubernetes-dashboard




