---
title: "本地部署kubernetes"
date: 2022-11-26T15:47:15+08:00
draft: false
lastmod: 2022-12-18T22:32:59+08:00
tags: ["Kubernetes", "deploy", "vagrant"]
categories: ["code"]
author: "Rouzip"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'
---

> 在本地desktop上安装一下Kubernetes集群

<!--more-->

## Deploy Kubernetes

In order to deploy kubernetes in my desktop, I use Vagrant to get multi nodes.

```vagrantFile
NUM_WORKER_NODES=2
IP_NW="192.168.56."
IP_START=50

Vagrant.configure("2") do |config|
  config.vm.provision "shell", env: {"IP_NW" => IP_NW, "IP_START" => IP_START}, inline: <<-SHELL
      apt-get update -y
      echo "$IP_NW$((IP_START)) master-node" >> /etc/hosts
      echo "$IP_NW$((IP_START+1)) worker-node01" >> /etc/hosts
      echo "$IP_NW$((IP_START+2)) worker-node02" >> /etc/hosts
  SHELL

  config.vm.box = "bento/ubuntu-22.04"
  config.vm.box_check_update = true

  config.vm.define "master" do |master|
    # master.vm.box = "bento/ubuntu-22.04"
    master.vm.hostname = "master-node"
    master.vm.network "private_network", ip: IP_NW + "#{IP_START}"
    master.vm.provider "virtualbox" do |vb|
        vb.memory = 16192
        vb.cpus = 4
    end
  end

  (1..NUM_WORKER_NODES).each do |i|

  config.vm.define "node0#{i}" do |node|
    node.vm.hostname = "worker-node0#{i}"
    node.vm.network "private_network", ip: IP_NW + "#{IP_START + i}"
    node.vm.provider "virtualbox" do |vb|
        vb.memory = 16192
        vb.cpus = 4
    end
  end

  end
end
```

### system setting on all nodes

#### swap off

Ingore safet problem, I excute in root mode.

```bash
swapoff -a
vim /etc/fstab

UUID=634dd626-b98b-4d4c-9f44-4d8007cfec4f /               ext4    errors=remount-ro 0       1
UUID=A52B-5B50  /boot/efi       vfat    umask=0077      0       1
#/swapfile                                 none            swap    sw              0       0
```

#### enable iptables bridged traffic on all nodes

```shell
cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

# sysctl params required by setup, params persist across reboots

cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sysctl --system
```

#### Deploy containerd、runc and cni plugin

get start [link](https://github.com/containerd/containerd/blob/main/docs/getting-started.md) (use option 1)

##### Deploy containerd

```bash
wget https://github.com/containerd/containerd/releases/download/v1.6.9/containerd-1.6.9-linux-amd64.tar.gz
tar Cxzvf /usr/local containerd-1.6.9-linux-amd64.tar.gz
# add containerd unit to systemd
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service > /usr/local/lib/systemd/system/containerd.service
systemctl daemon-reload
systemctl enable --now containerd

mkdir -p /etc/containerd/
containerd config default | tee /etc/containerd/config.toml
```

##### Deploy runc

```bash
wget https://github.com/opencontainers/runc/releases/download/v1.1.4/runc.amd64
install -m 755 runc.amd64 /usr/local/sbin/runc
```

##### Deploy CNI plugins

```bash
wget https://github.com/containernetworking/plugins/releases/download/v1.1.1/cni-plugins-linux-amd64-v1.1.1.tgz
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.tgz
```

##### Deploy nerdctl

```bash
wget https://github.com/containerd/nerdctl/releases/download/v1.0.0/nerdctl-full-1.0.0-linux-amd64.tar.gz
tar Cxzvvf /usr/local nerdctl-full-1.0.0-linux-amd64.tar.gz
systemctl enable --now containerd
```

#### Deploy kubeadm、kubelet、kubectl

```bash
apt-get update
apt-get install -y apt-transport-https ca-certificates curl
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

#### use kubeadm to start a cluster

```bash
# To install kubernetes using flannel, master node initially needs to run:
kubeadm init --pod-network-cidr 10.244.0.0/16
# install cni plugin in master node
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
# use master node as a worker node
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
# it will show kubeadm join command, use it in worker node to join cluster
kubeadm join 10.67.124.180:6443 --token xxxxxxxxxxxxx --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
# (if you forget it, you can run this to get join command again
kubeadm token create --print-join-command
```
