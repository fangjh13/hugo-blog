---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2020-12-23T13:16:29+08:00'
description: 本地k8s集群
lastmod: '2020-12-23T13:18:51+08:00'
showToc: true
tags: [Docker, Kubernetes]
title: 使用kubeadm在虚拟机本地搭建Kubernetes集群
---

# 使用kubeadm在虚拟机本地搭建Kubernetes集群	

本文使用ESXi创建3台ubuntu server 虚拟机搭建一个完整的Kubernetes集群，1台master主节点，2台worker做为工作节点。很多地址都是google的域名，安装下面一些环境可能需要科学上网，创建k8s集群需要如下包

- docker  --  容器运行环境
- kubelet -- Kubernets节点代理  
- kubeadm -- 部署多节点Kubernetes集群的工具
- kubectl -- 用于和Kubernetes交互的命令行工具

会创建如下3台机器

| 主机名     | IP地址     | 角色      |
| ---------- | ---------- | --------- |
| master.k8s | 10.0.0.175 | 主节点    |
| node1.k8s  | 10.0.0.176 | 工作节点1 |
| node2.k8s  | 10.0.0.177 | 工作节点2 |

记一下安装时最新的版本号

- ubuntu server 20.04.1
- kernel version 5.4.0
- kubernetes v1.20.0
- docker-ce  19.03.14

## 创建虚拟机配置网址和安装docker  

 首先在ESXi控制台创建一台ubuntu server虚拟机，配置建议2CPU、2G RAM、20G硬盘，主机名为`master.k8s`开机更新到最新版本后重启安装docker，切换到root用户，以下所有操作都用root用户

 修改主机名

```bash
# set hostname
hostnamectl set-hostname master.k8s
```

然后修改`/etc/hosts`域名解析并添加之后两台的ip地址，如下

```bash
10.0.0.175 master.k8s
10.0.0.176 node1.k8s
10.0.0.177 node2.k8s
```

配置主节点的网络修改`/etc/netplan/00-installer-config.yaml`如下，我这网卡是`ens160`

```bash
network:
  version: 2
  renderer: networkd
  ethernets:
    ens160:                             # change your's
      dhcp4: no
      addresses: [10.0.0.175/24]        # change your's
      gateway4: 10.0.0.1                # change your's
      nameservers:
        addresses: [10.0.0.1]           # change your's
```

保存后运行

```bash
netplan apply
```

可以使用`ip a`查看修改情况，然后取消系统自带的`systemd-resolved.service`这个dns解析服务，是可选的

```bash
systemctl stop systemd-resolved.service
systemctl disable systemd-resolved.service
rm -rf /etc/resolv.conf
echo "nameserver 10.0.0.1" > /etc/resolv.conf      # change your's
```

重启机器，确保网络畅通


然后安装docker

```bash
apt-get update
apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable" 
apt-get update
apt-get install docker-ce docker-ce-cli containerd.io
```

安装好docker后需要配置下`docker`和`containerd`使得和`kubelet`兼容，增加如下配置文件，使`systemd`为cgroup驱动

```bash
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

# Restart Docker
systemctl daemon-reload
systemctl restart docker
# start on boot
systemctl enable docker
```

配置`containerd`

```bash
cat > /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system

# Configure containerd
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
```

修改`/etc/containerd/config.toml`中的`systemd_cgroup = true`使用`systemd`做为cgroup驱动，最后重启

```bash
systemctl restart containerd
```

##　安装 kubeadm, kubelet 和 kubectl

```bash
＃ disable swap
swapoff -a && sed -i '/swap.img/ s/^/# /' /etc/fstab


apt-get update && sudo apt-get install -y apt-transport-https curl

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl

# 以下是可选的是为了 kubectl 命令可以自动补全，ubuntu 20.04 已经安装了 bash-completion 不需要手动安装
# 在~/.bashrc下添加 source /usr/share/bash-completion/bash_completion
# 运行 kubectl completion bash >/etc/bash_completion.d/kubectl
```

现在还没有运行`kubeadm init`所以当你查看`systemctl status kubelet`会显示启动失败，先不用管他，关机进行下一步克隆两台工作机


## 克隆主机

在ESXi web 控制台 Storage > Datastore browser > Create directory 新建`node1.k8s`和`node2.k8s`两个文件夹用来存放两台worker机文件，然后将`master.k8s`文件夹中的`master.k8s.vmdk`和`master.k8s.vmx`复制到新建的两个worker文件夹，最后右击worker文件夹中的 `master.k8s.vmx` > Register VM ，返回主页修改主机名开机后选`I Copied It`，开机进入后和master一样要**修改主机名和IP地址**

## 运行 kubeadm init

做好上面所有准备后把三台机器都开起来，master是10.0.0.175，worker两台分别为10.0.0.176、10.0.0.177，确保可以相互ping通，然后在master上运行`kubeadm init`创建**Kubernetes control-plane**，包括`etcd`数据库、`API Server`(与 kubectl 命令交互)等都是以容器的方式运行，第一次运行会拉取镜像可能需要一些时间取决于你的网络，完成后有类似如下的显示

```bash
...

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.0.0.175:6443 --token hyq0wa.jdu1ihaf0r3pzemo \
    --discovery-token-ca-cert-hash sha256:90ec90f7a2698e19c3f026b7bc31d86a30a801d20fec7aa17dccfe70ec76e419 

```

## 加入master节点

之前在`kubeadm init`提示有加入主节点的命令，分别在两台worker机上运行就行，如果之前没有记可以在master上运行`kubeadm token create --print-join-command`新生成

```bash
kubeadm join 10.0.0.175:6443 --token vdib2x.qq73s0h8nq70x6er     --discovery-token-ca-cert-hash sha256:90ec90f7a2698e19c3f026b7bc31d86a30a801d20fec7aa17dccfe70ec76e419 
```

在两台worker机上运行后 去master节点查看

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
 
kubectl get nodes
NAME         STATUS     ROLES                  AGE   VERSION
master.k8s   NotReady   control-plane,master   12s   v1.20.0
node1.k8s    NotReady   <none>                 12s   v1.20.0
node2.k8s    NotReady   <none>                 12s   v1.20.0
```

有如上输出代表加入master成功，但仔细看会发现他们的状态都是NotReady，那是因为容器网络插件(CNI)还没有安装，接下来安装网络插件我们使用[Weaveworks](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/) ，在master运行

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

之后再查看状态就会变成Ready了，到此为止一个两个node节点一个master的本地集群就搭建完成了

如果需要在另外的机器使用集群 那把`/etc/kubernetes/admin.conf`复制到机器然后设置`KUBECONFIG`环境变量即可

## Reference

[https://kubernetes.io/docs/setup/production-environment/container-runtimes/](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)

[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

[https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-installation](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-installation)
