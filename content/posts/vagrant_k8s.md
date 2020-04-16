---
title: 使用 vagrant 搭建 kubernetes 基础环境
date: 2019-09-15 01:31:26
categories:
  - linux
tags:
  - vagrant
  - virtualbox
  - kubernetes
---

使用 vagrant 搭建 kubernetes 基础环境
<!--more-->


```
# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV["LC_ALL"] = "en_US.UTF-8"
Vagrant.configure("2") do |config|
  
  config.vm.define "kubemaster" do |kubemaster|
    kubemaster.vm.box = "centos/7"
    kubemaster.vm.hostname = "kubemaster"
    kubemaster.vm.network "private_network", ip: "192.168.33.100"
    kubemaster.vm.provider "virtualbox" do |vb|
      vb.gui = false
      vb.cpus = 2
      vb.memory = "1024"
    end
  end

  config.vm.define "kubenode01" do |kubenode01|
    kubenode01.vm.box = "centos/7"
    kubenode01.vm.hostname = "kubenode01"
    kubenode01.vm.network "private_network", ip: "192.168.33.101"
    kubenode01.vm.provider "virtualbox" do |vb|
      vb.gui = false
      vb.cpus = 2
      vb.memory = "1024"
    end
  end

  config.vm.define "kubenode02" do |kubenode02|
    kubenode02.vm.box = "centos/7"
    kubenode02.vm.hostname = "kubenode02"
    kubenode02.vm.network "private_network", ip: "192.168.33.102"
    kubenode02.vm.provider "virtualbox" do |vb|
      vb.gui = false
      vb.cpus = 2
      vb.memory = "1024"
    end
  end
            
  config.vm.provision "shell", inline: <<-SHELL
    sudo -i
    echo "设置SSH"
    sed -i 's/^#PermitRootLogin yes/PermitRootLogin yes/g' /etc/ssh/sshd_config
    sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
    systemctl restart sshd
    echo "manager" | passwd --stdin root
    echo "manager" | passwd --stdin vagrant

    echo "# 关闭firewalld、selinux"
    setenforce 0
    sed -i 's/^SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
    systemctl disable firewalld && systemctl stop firewalld

    echo "# 设置/etc/hosts"
    cat << EOF >> /etc/hosts
192.168.33.100 kubemaster
192.168.33.101 kubenode01
192.168.33.102 kubenode02
EOF

    echo "# 安装依赖"
    yum makecache
    yum -y install vim wget curl sysstat bind-utils zlib-devel openssl-devel conntrack ipvsadm sysstat wget git iptables-services yum-utils device-mapper-persistent-data lvm2 net-tools

    echo "# 安装docker"
    yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    yum install -y docker-ce
    systemctl enable docker && systemctl start docker
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
    systemctl daemon-reload && systemctl restart docker
    
    echo "# 启用iptables"
    systemctl enable iptables && systemctl start iptables
    iptables -F
    service iptables save

    echo "# 关闭系统不需要的服务"
    systemctl disable postfix && systemctl stop postfix

    echo "# kube-proxy开启ipvs"
    modprobe br_netfilter
    cat << EOF > /etc/sysconfig/modules/ipvs.modules
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
    chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules
    lsmod | grep -e ip_vs -e nf_conntrack_ipv4

    echo "# 调整内核参数"
    cat << EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv4.ip_forward = 1
EOF
    sysctl -p /etc/sysctl.d/k8s.conf

    echo "# 添加kubernetes源"
    cat << EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
  SHELL
end
```
