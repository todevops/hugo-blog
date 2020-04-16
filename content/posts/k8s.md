---
title: Kubernetes 离线安装
date: 2019-02-07 01:31:26
categories:
  - linux
tags:
  - Kubernetes
  - docker
---

Kubernetes 离线安装
<!--more-->

```bash
cat /etc/hosts
192.168.61.11 node1
192.168.61.12 node2


systemctl stop firewalld
systemctl disable firewalld

setenforce 0
vim /etc/selinux/config

SELINUX=disabled

vim /etc/sysctl.d/k8s.conf

net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1

modprobe br_netfilter
sysctl -p /etc/sysctl.d/k8s.conf
```

## kube-proxy开启ipvs的前置条件

```bash
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4

# 查看是否已经正确加载所需的内核模块
lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

## 安装Docker
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装 Docker-CE
sudo yum makecache fast

yum list docker-ce.x86_64  --showduplicates |sort -r

yum makecache fast

yum install -y --setopt=obsoletes=0 \
  docker-ce-18.06.1.ce-3.el7

systemctl start docker
systemctl enable docker
```

```
iptables -nvL
```




## 安装kubeadm和kubelet
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

yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
```

```
kubelet –help

cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

vim /etc/sysconfig/kubelet

KUBELET_EXTRA_ARGS=--fail-swap-on=false
```

## 使用kubeadm init初始化集群

```bash
kubeadm config images list


#!/bin/bash
images=(kube-apiserver:v1.13.3 kube-controller-manager:v1.13.3 kube-scheduler:v1.13.3 kube-proxy:v1.13.3 pause:3.1 etcd:3.2.24)

for imageName in ${images[@]} ; do

  docker pull mirrorgooglecontainers/$imageName
  docker tag mirrorgooglecontainers/$imageName k8s.gcr.io/$imageName
  docker rmi mirrorgooglecontainers/$imageName
done


 docker pull coredns/coredns:1.2.6
 docker tag coredns/coredns:1.2.6  k8s.gcr.io/coredns:1.2.6
 docker rmi coredns/coredns:1.2.6

kubeadm init \
  --kubernetes-version=v1.13.0 \
  --pod-network-cidr=10.244.0.0/16 \
  --apiserver-advertise-address=192.168.41.132 \
  --ignore-preflight-errors=Swap
# running with swap on is not supported. Please disable swap
# 添加--ignore-preflight-errors=Swap参数忽略这个错误


mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubeadm join 192.168.41.132:6443 --token e4xr2p.xvfc3dr6a4dvz8se --discovery-token-ca-cert-hash sha256:7f99258581c9118551ec500b61bf3d732ed6e31799e5074fc5f75b784633410a

kubectl get cs

# 集群初始化如果遇到问题，可以使用下面的命令进行清理
kubeadm reset


```

## 安装Pod Network

```bash
mkdir -p ~/k8s/
cd ~/k8s
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f  kube-flannel.yml
```




