[TOC]

# 使用`kubeadm`部署`kubernetes`

## 准备工作

```
安装好的3台CentOS 7.9机器
每台机器最少2GB内存
每台机器最少2核CPU
集群中的所有机器的网络彼此均能相互连接(公网和内网都可以)
节点之中不可以有重复的主机名、MAC地址或product_uuid
开启机器上的某些端口
为了保证kubelet正常工作，必须禁用交换分区
```

## 为3台机器设置主机名

```shell
[root@node01 ~]# hostnamectl set-hostname k8s-master
```

```shell
[root@node02 ~]# hostnamectl set-hostname k8s-node1
```

```shell
[root@node03 ~]# hostnamectl set-hostname k8s-node2
```

## 查看3台机器的`MAC`地址

```shell
[root@k8s-master ~]# ip link
```

```shell
[root@k8s-node1 ~]# ip link
```

```shell
[root@k8s-node2 ~]# ip link
```

## 查看3台机器的`product_uuid`

```shell
[root@k8s-master ~]# cat /sys/class/dmi/id/product_uuid
```

```shell
[root@k8s-node1 ~]# cat /sys/class/dmi/id/product_uuid
```

```shell
[root@k8s-node2 ~]# cat /sys/class/dmi/id/product_uuid
```

## 关闭防火墙并禁用开机自启动

```shell
[root@k8s-master ~]# systemctl stop firewalld.service
[root@k8s-master ~]# systemctl status firewalld.service
[root@k8s-master ~]# systemctl disable firewalld.service
[root@k8s-master ~]# systemctl list-unit-files | grep firewalld.service
```

```shell
[root@k8s-node1 ~]# systemctl stop firewalld.service
[root@k8s-node1 ~]# systemctl status firewalld.service
[root@k8s-node1 ~]# systemctl disable firewalld.service
[root@k8s-node1 ~]# systemctl list-unit-files | grep firewalld.service
```

```shell
[root@k8s-node2 ~]# systemctl stop firewalld.service
[root@k8s-node2 ~]# systemctl status firewalld.service
[root@k8s-node2 ~]# systemctl disable firewalld.service
[root@k8s-node2 ~]# systemctl list-unit-files | grep firewalld.service
```

## 永久关闭3台机器的`SELINUX`

```shell
[root@k8s-master ~]# sed -i "s/SELINUX=enforcing/SELINUX=disabled/" /etc/selinux/config
```

```shell
[root@k8s-node1 ~]# sed -i "s/SELINUX=enforcing/SELINUX=disabled/" /etc/selinux/config
```

```shell
[root@k8s-node2 ~]# sed -i "s/SELINUX=enforcing/SELINUX=disabled/" /etc/selinux/config
```

## 永久关闭`swap`分区

```shell
[root@k8s-master ~]# sed -ri "s/.*swap.*/# &/" /etc/fstab
```

```shell
[root@k8s-node1 ~]# sed -ri "s/.*swap.*/# &/" /etc/fstab
```

```shell
[root@k8s-node2 ~]# sed -ri "s/.*swap.*/# &/" /etc/fstab
```

## 为3台机器添加`hosts`

```shell
[root@k8s-master ~]# cat >> /etc/hosts << EOF
192.168.86.101 k8s-master
192.168.86.102 k8s-node1
192.168.86.103 k8s-node2
EOF
```

```shell
[root@k8s-node1 ~]# cat >> /etc/hosts << EOF
192.168.86.101 k8s-master
192.168.86.102 k8s-node1
192.168.86.103 k8s-node2
EOF
```

```shell
[root@k8s-node2 ~]# cat >> /etc/hosts << EOF
192.168.86.101 k8s-master
192.168.86.102 k8s-node1
192.168.86.103 k8s-node2
EOF
```

## 配置时间同步

```shell
[root@k8s-master ~]# yum -y install ntp
[root@k8s-master ~]# sed -i "s/server 0.centos.pool.ntp.org iburst/server ntp.aliyun.com iburst/" /etc/ntp.conf
[root@k8s-master ~]# sed -i "s/server 1.centos.pool.ntp.org iburst/server ntp1.aliyun.com iburst/" /etc/ntp.conf
[root@k8s-master ~]# sed -i "s/server 2.centos.pool.ntp.org iburst/server ntp2.aliyun.com iburst/" /etc/ntp.conf
[root@k8s-master ~]# sed -i "s/server 3.centos.pool.ntp.org iburst/server ntp3.aliyun.com iburst/" /etc/ntp.conf
[root@k8s-master ~]# systemctl enable ntpd.service
[root@k8s-master ~]# systemctl start ntpd.service
[root@k8s-master ~]# systemctl status ntpd.service
[root@k8s-master ~]# ntpq -p
```

```shell
[root@k8s-node1 ~]# yum -y install ntp
[root@k8s-node1 ~]# sed -i "s/server 0.centos.pool.ntp.org iburst/server ntp.aliyun.com iburst/" /etc/ntp.conf
[root@k8s-node1 ~]# sed -i "s/server 1.centos.pool.ntp.org iburst/server ntp1.aliyun.com iburst/" /etc/ntp.conf
[root@k8s-node1 ~]# sed -i "s/server 2.centos.pool.ntp.org iburst/server ntp2.aliyun.com iburst/" /etc/ntp.conf
[root@k8s-node1 ~]# sed -i "s/server 3.centos.pool.ntp.org iburst/server ntp3.aliyun.com iburst/" /etc/ntp.conf
[root@k8s-node1 ~]# systemctl enable ntpd.service
[root@k8s-node1 ~]# systemctl start ntpd.service
[root@k8s-node1 ~]# systemctl status ntpd.service
[root@k8s-node1 ~]# ntpq -p
```

```shell
[root@k8s-node2 ~]# yum -y install ntp
[root@k8s-node2 ~]# sed -i "s/server 0.centos.pool.ntp.org iburst/server ntp.aliyun.com iburst/" /etc/ntp.conf
[root@k8s-node2 ~]# sed -i "s/server 1.centos.pool.ntp.org iburst/server ntp1.aliyun.com iburst/" /etc/ntp.conf
[root@k8s-node2 ~]# sed -i "s/server 2.centos.pool.ntp.org iburst/server ntp2.aliyun.com iburst/" /etc/ntp.conf
[root@k8s-node2 ~]# sed -i "s/server 3.centos.pool.ntp.org iburst/server ntp3.aliyun.com iburst/" /etc/ntp.conf
[root@k8s-node2 ~]# systemctl enable ntpd.service
[root@k8s-node2 ~]# systemctl start ntpd.service
[root@k8s-node2 ~]# systemctl status ntpd.service
[root@k8s-node2 ~]# ntpq -p
```

## 确保加载`br_netfilter`模块

```shell
[root@k8s-master ~]# lsmod | grep br_netfilter
[root@k8s-master ~]# modprobe br_netfilter
```

```shell
[root@k8s-node1 ~]# lsmod | grep br_netfilter
[root@k8s-node1 ~]# modprobe br_netfilter
```

```shell
[root@k8s-node2 ~]# lsmod | grep br_netfilter
[root@k8s-node2 ~]# modprobe br_netfilter
```

## 让`Linux`节点上的`iptables`能够正确地查看桥接流量

```shell
[root@k8s-master ~]# cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
[root@k8s-master ~]# cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
[root@k8s-master ~]# sudo sysctl --system
```

```shell
[root@k8s-node1 ~]# cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
[root@k8s-node1 ~]# cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
[root@k8s-node1 ~]# sudo sysctl --system
```

```shell
[root@k8s-node2 ~]# cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
[root@k8s-node2 ~]# cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
[root@k8s-node2 ~]# sudo sysctl --system
```

## 检查所需端口

### 控制平面节点

| 协议 | 方向 | 端口范围  | 作用                      | 使用者                        |
| ---- | ---- | --------- | ------------------------- | ----------------------------- |
| TCP  | 入站 | 6443      | `Kubernetes API` 服务器   | 所有组件                      |
| TCP  | 入站 | 2379-2380 | `etcd`服务器客户端`API`   | `kube-apiserver`, `etcd`      |
| TCP  | 入站 | 10250     | `Kubelet API`             | `kubelet`自身、控制平面组件   |
| TCP  | 入站 | 10251     | `kube-scheduler`          | `kube-scheduler`自身          |
| TCP  | 入站 | 10252     | `kube-controller-manager` | `kube-controller-manager`自身 |

### 工作节点

| 协议 | 方向 | 端口范围    | 作用           | 使用者                      |
| ---- | ---- | ----------- | -------------- | --------------------------- |
| TCP  | 入站 | 10250       | `Kubelet API`  | `Kubelet`自身、控制平面组件 |
| TCP  | 入站 | 30000-32767 | `NodePort`服务 | 所有组件                    |

## 在3个机器上安装`docker`引擎

### 卸载旧版本

```shell
[root@k8s-master ~]# yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine podman runc
```

```shell
[root@k8s-node1 ~]# yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine podman runc
```

```shell
[root@k8s-node2 ~]# yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine podman runc
```

### 安装必要的一些系统工具

```shell
[root@k8s-master ~]# yum install -y yum-utils device-mapper-persistent-data lvm2
```

```shell
[root@k8s-node1 ~]# yum install -y yum-utils device-mapper-persistent-data lvm2
```

```shell
[root@k8s-node2 ~]# yum install -y yum-utils device-mapper-persistent-data lvm2
```

### 添加软件源信息

```shell
[root@k8s-master ~]# yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
[root@k8s-master ~]# sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
```

```shell
[root@k8s-node1 ~]# yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
[root@k8s-node1 ~]# sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
```

```shell
[root@k8s-node2 ~]# yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
[root@k8s-node2 ~]# sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
```

### 更新并安装`docker`

```shell
[root@k8s-master ~]# yum makecache fast
[root@k8s-master ~]# yum -y install docker-ce docker-ce-cli containerd.io
```

```shell
[root@k8s-node1 ~]# yum makecache fast
[root@k8s-node1 ~]# yum -y install docker-ce docker-ce-cli containerd.io
```

```shell
[root@k8s-node2 ~]# yum makecache fast
[root@k8s-node2 ~]# yum -y install docker-ce docker-ce-cli containerd.io
```

### 查看`docker`版本

```shell
[root@k8s-master ~]# docker version
[root@k8s-master ~]# docker -v
[root@k8s-master ~]# docker --version
```

```shell
[root@k8s-node1 ~]# docker version
[root@k8s-node1 ~]# docker -v
[root@k8s-node1 ~]# docker --version
```

```shell
[root@k8s-node2 ~]# docker version
[root@k8s-node2 ~]# docker -v
[root@k8s-node2 ~]# docker --version
```

### 启动`docker`

```shell
[root@k8s-master ~]# systemctl start docker
```

```shell
[root@k8s-node1 ~]# systemctl start docker
```

```shell
[root@k8s-node2 ~]# systemctl start docker
```

### 验证`docker`是否正确安装

```shell
[root@k8s-master ~]# docker run hello-world
```

```shell
[root@k8s-node1 ~]# docker run hello-world
```

```shell
[root@k8s-node2 ~]# docker run hello-world
```

### 为`docker`设置开机自启动

```shell
[root@k8s-master ~]# systemctl enable docker.service
[root@k8s-master ~]# systemctl list-unit-files | grep docker.service
```

```shell
[root@k8s-node1 ~]# systemctl enable docker.service
[root@k8s-node1 ~]# systemctl list-unit-files | grep docker.service
```

```shell
[root@k8s-node2 ~]# systemctl enable docker.service
[root@k8s-node2 ~]# systemctl list-unit-files | grep docker.service
```

## 配置镜像加速器

### 配置阿里云镜像加速器

```shell
[root@k8s-master ~]# mkdir -p /etc/docker
[root@k8s-master ~]# tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://2r3tctoz.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
```

```shell
[root@k8s-node1 ~]# mkdir -p /etc/docker
[root@k8s-node1 ~]# tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://2r3tctoz.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
```

```shell
[root@k8s-node2 ~]# mkdir -p /etc/docker
[root@k8s-node2 ~]# tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://2r3tctoz.mirror.aliyuncs.com"]
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
```

### 守护进程重新加载

```shell
[root@k8s-master ~]# systemctl daemon-reload
```

```shell
[root@k8s-node1 ~]# systemctl daemon-reload
```

```shell
[root@k8s-node2 ~]# systemctl daemon-reload
```

### 重启`docker`

```shell
[root@k8s-master ~]# systemctl restart docker
```

```shell
[root@k8s-node1 ~]# systemctl restart docker
```

```shell
[root@k8s-node2 ~]# systemctl restart docker
```

### 查看`docker`服务信息

```shell
[root@k8s-master ~]# docker info
```

```shell
[root@k8s-node1 ~]# docker info
```

```shell
[root@k8s-node2 ~]# docker info
```

## 配置`kubernetes`阿里云`yum`源

```shell
[root@k8s-master ~]# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

```shell
[root@k8s-node1 ~]# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

```shell
[root@k8s-node2 ~]# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

## 安装` kubelet`、`kubeadm`、`kubectl`，并设置开机启动

```shell
[root@k8s-master ~]# yum -y install kubelet kubeadm kubectl --disableexcludes=kubernetes
[root@k8s-master ~]# systemctl enable --now kubelet
[root@k8s-master ~]# systemctl start kubelet
[root@k8s-master ~]# systemctl status kubelet
```

```shell
[root@k8s-node1 ~]# yum -y install kubelet kubeadm kubectl --disableexcludes=kubernetes
[root@k8s-node1 ~]# sudo systemctl enable --now kubelet
[root@k8s-node1 ~]# systemctl start kubelet
[root@k8s-node1 ~]# systemctl status kubelet
```

```shell
[root@k8s-node2 ~]# yum -y install kubelet kubeadm kubectl --disableexcludes=kubernetes
[root@k8s-node2 ~]# sudo systemctl enable --now kubelet
[root@k8s-node2 ~]# sudo systemctl start kubelet
[root@k8s-node2 ~]# systemctl status kubelet
```

```shell
[root@k8s-master ~]# kubeadm init --apiserver-advertise-address 192.168.86.101 --control-plane-endpoint k8s-master --kubernetes-version v1.22.2 --image-repository registry.aliyuncs.com/google_containers --service-cidr 10.96.0.0/16 --pod-network-cidr 172.31.0.0/16 --ignore-preflight-errors all
```

## 出现以下信息，说明`Kubernetes`控制平面已初始化成功!

```
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

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join k8s-master:6443 --token yfspmy.wrg2w4mrnz6p8r66 \
	--discovery-token-ca-cert-hash sha256:00e4eb4c3681a81c27968ff837d2de59832feed6eb399ad25320a515d6180e0b \
	--control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join k8s-master:6443 --token yfspmy.wrg2w4mrnz6p8r66 \
	--discovery-token-ca-cert-hash sha256:00e4eb4c3681a81c27968ff837d2de59832feed6eb399ad25320a515d6180e0b
```

## 拷贝配置文件，开始使用集群

```shell
[root@k8s-master ~]# mkdir -p $HOME/.kube
[root@k8s-master ~]# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@k8s-master ~]# chown $(id -u):$(id -g) $HOME/.kube/config
```

## 将`pod`网络部署到集群中

```shell
[root@k8s-master ~]# curl https://docs.projectcalico.org/manifests/calico.yaml -O
[root@k8s-master ~]# kubectl apply -f calico.yaml
```

## 查看节点信息

```shell
[root@k8s-master ~]# kubectl get nodes
```

## 查看`pods`

```shell
[root@k8s-master ~]# kubectl get pods -A
```

## 将工作节点加入到集群

```shell
[root@k8s-node1 ~]# kubeadm join k8s-master:6443 --token yfspmy.wrg2w4mrnz6p8r66 --discovery-token-ca-cert-hash sha256:00e4eb4c3681a81c27968ff837d2de59832feed6eb399ad25320a515d6180e0b
```

```shell
[root@k8s-node2 ~]# kubeadm join k8s-master:6443 --token yfspmy.wrg2w4mrnz6p8r66 --discovery-token-ca-cert-hash sha256:00e4eb4c3681a81c27968ff837d2de59832feed6eb399ad25320a515d6180e0b
```

## 出现以下信息，说明该工作节点已加入到集群

```
This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

## 在`master`节点查看工作节点是否已经加入到集群中

```shell
[root@k8s-master ~]# kubectl get nodes
```

## 在`master`节点查看`pods`状态，观察变化

```shell
[root@k8s-master ~]# kubectl get pods -A -w
```

## 重启以检验`kubernetes`自我修复能力

```shell
[root@k8s-master ~]# reboot
```

```shell
[root@k8s-node1 ~]# reboot
```

```shell
[root@k8s-node1 ~]# reboot
```

## 重启后查看集群节点是否正常，`pods`是否正常

```shell
[root@k8s-master ~]# kubectl get nodes
```

```shell
[root@k8s-master ~]# watch -n 1 kubectl get pods -A
```

## 下载`kubernetes-dashboard`

```shell
[root@k8s-master ~]# wget -O kubernetes-dashboard.yaml https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
```

## 修改配置文件，增加`type: NodePort`和`nodePort: 32520`两行

```yaml
---

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 32520
  type: NodePort
  selector:
    k8s-app: kubernetes-dashboard

---
```

## 修改后的完整`kubernetes-dashboard yaml`文件

```yaml
# Copyright 2017 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

apiVersion: v1
kind: Namespace
metadata:
  name: kubernetes-dashboard

---

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard

---

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 32520
  type: NodePort
  selector:
    k8s-app: kubernetes-dashboard

---

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-certs
  namespace: kubernetes-dashboard
type: Opaque

---

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-csrf
  namespace: kubernetes-dashboard
type: Opaque
data:
  csrf: ""

---

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-key-holder
  namespace: kubernetes-dashboard
type: Opaque

---

kind: ConfigMap
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-settings
  namespace: kubernetes-dashboard

---

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
rules:
  # Allow Dashboard to get, update and delete Dashboard exclusive secrets.
  - apiGroups: [""]
    resources: ["secrets"]
    resourceNames: ["kubernetes-dashboard-key-holder", "kubernetes-dashboard-certs", "kubernetes-dashboard-csrf"]
    verbs: ["get", "update", "delete"]
    # Allow Dashboard to get and update 'kubernetes-dashboard-settings' config map.
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["kubernetes-dashboard-settings"]
    verbs: ["get", "update"]
    # Allow Dashboard to get metrics.
  - apiGroups: [""]
    resources: ["services"]
    resourceNames: ["heapster", "dashboard-metrics-scraper"]
    verbs: ["proxy"]
  - apiGroups: [""]
    resources: ["services/proxy"]
    resourceNames: ["heapster", "http:heapster:", "https:heapster:", "dashboard-metrics-scraper", "http:dashboard-metrics-scraper"]
    verbs: ["get"]

---

kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
rules:
  # Allow Metrics Scraper to get metrics from the Metrics server
  - apiGroups: ["metrics.k8s.io"]
    resources: ["pods", "nodes"]
    verbs: ["get", "list", "watch"]

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kubernetes-dashboard
subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kubernetes-dashboard
subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard

---

kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      containers:
        - name: kubernetes-dashboard
          image: kubernetesui/dashboard:v2.2.0
          imagePullPolicy: Always
          ports:
            - containerPort: 8443
              protocol: TCP
          args:
            - --auto-generate-certificates
            - --namespace=kubernetes-dashboard
            # Uncomment the following line to manually specify Kubernetes API server Host
            # If not specified, Dashboard will attempt to auto discover the API server and connect
            # to it. Uncomment only if the default does not work.
            # - --apiserver-host=http://my-address:port
          volumeMounts:
            - name: kubernetes-dashboard-certs
              mountPath: /certs
              # Create on-disk volume to store exec logs
            - mountPath: /tmp
              name: tmp-volume
          livenessProbe:
            httpGet:
              scheme: HTTPS
              path: /
              port: 8443
            initialDelaySeconds: 30
            timeoutSeconds: 30
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            runAsUser: 1001
            runAsGroup: 2001
      volumes:
        - name: kubernetes-dashboard-certs
          secret:
            secretName: kubernetes-dashboard-certs
        - name: tmp-volume
          emptyDir: {}
      serviceAccountName: kubernetes-dashboard
      nodeSelector:
        "kubernetes.io/os": linux
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule

---

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: dashboard-metrics-scraper
  name: dashboard-metrics-scraper
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 8000
      targetPort: 8000
  selector:
    k8s-app: dashboard-metrics-scraper

---

kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: dashboard-metrics-scraper
  name: dashboard-metrics-scraper
  namespace: kubernetes-dashboard
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: dashboard-metrics-scraper
  template:
    metadata:
      labels:
        k8s-app: dashboard-metrics-scraper
      annotations:
        seccomp.security.alpha.kubernetes.io/pod: 'runtime/default'
    spec:
      containers:
        - name: dashboard-metrics-scraper
          image: kubernetesui/metrics-scraper:v1.0.6
          ports:
            - containerPort: 8000
              protocol: TCP
          livenessProbe:
            httpGet:
              scheme: HTTP
              path: /
              port: 8000
            initialDelaySeconds: 30
            timeoutSeconds: 30
          volumeMounts:
          - mountPath: /tmp
            name: tmp-volume
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            runAsUser: 1001
            runAsGroup: 2001
      serviceAccountName: kubernetes-dashboard
      nodeSelector:
        "kubernetes.io/os": linux
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule
      volumes:
        - name: tmp-volume
          emptyDir: {}
```

## 部署`dashboard`

```shell
[root@k8s-master ~]# kubectl apply -f kubernetes-dashboard.yaml
```

## 创建`admin-user.yaml`文件

```shell
[root@k8s-master ~]# cat >> admin-user.yaml << EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

## 创建服务账户

```shell
[root@k8s-master ~]# kubectl apply -f admin-user.yaml
```

## 创建`admin-user-role.yaml`文件

```shell
[root@k8s-master ~]# cat >> admin-user-cluster-role-binding.yaml << EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

## 创建集群角色绑定

```shell
[root@k8s-master ~]# kubectl apply -f admin-user-cluster-role-binding.yaml
```

## 获取`token`

```shell
[root@k8s-master ~]# kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```

## 使用生成的`token`通过浏览器访问`kubernetes-dashboard`

```
https://IP:32520
```

## 完结撒花

## 其他相关命令

### 如果令牌过期，使用以下命令重新生成`token`

```shell
[root@k8s-master ~]# kubeadm token create --print-join-command
```

### 修改`kubernetes-dashboard`访问端口

#### 将`type: ClusterIP`修改为`NodePort`

```shell
[root@k8s-master ~]# kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard
```

#### 查看`kubernetes-dashboard`访问端口

```shell
[root@k8s-master ~]# kubectl get svc -A | grep kubernetes-dashboard
```