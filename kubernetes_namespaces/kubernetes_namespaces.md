- [名字空间](#名字空间)
  - [名字空间概念](#名字空间概念)
  - [名字空间用途](#名字空间用途)
  - [列出集群中现存的名字空间](#列出集群中现存的名字空间)
  - [`Kubernetes`创建的四个初始名字空间](#kubernetes创建的四个初始名字空间)
  - [为请求设置名字空间](#为请求设置名字空间)
  - [设置名字空间偏好](#设置名字空间偏好)
  - [创建名字空间](#创建名字空间)
  - [删除名字空间](#删除名字空间)
  - [查看哪些`Kubernetes`资源在名字空间中](#查看哪些kubernetes资源在名字空间中)
  - [查看哪些`Kubernetes`资源不在名字空间中](#查看哪些kubernetes资源不在名字空间中)
  - [列出集群中现有的名字空间](#列出集群中现有的名字空间)
  - [获取特定名字空间的摘要](#获取特定名字空间的摘要)
  - [列出集群中现有的名字空间的详细信息](#列出集群中现有的名字空间的详细信息)
  - [获取特定名字空间的详细信息](#获取特定名字空间的详细信息)
  - [通过`yaml`文件创建名字空间](#通过yaml文件创建名字空间)
    - [创建`yaml`文件](#创建yaml文件)
    - [应用`yaml`文件创建名字空间](#应用yaml文件创建名字空间)
  - [通过`yaml`文件删除名字空间](#通过yaml文件删除名字空间)
    - [创建`yaml`文件](#创建yaml文件-1)
    - [通过`yaml`文件删除名字空间](#通过yaml文件删除名字空间-1)
  - [查看指定名字空间的`pod`](#查看指定名字空间的pod)

# 名字空间

## 名字空间概念

```
Kubernetes支持多个虚拟集群，它们底层依赖于同一个物理集群。 这些虚拟集群被称为名字空间。 在一些文档里名字空间也称为命名空间。
```

## 名字空间用途

```
在多个用户之间划分集群资源的一种方法（通过资源配额）
```

## 列出集群中现存的名字空间

```shell
[root@k8s-master ~]# kubectl get namespace
```

## `Kubernetes`创建的四个初始名字空间

- `default` 

```
没有指明使用其它名字空间的对象所使用的默认名字空间
```

- `kube-system`

```
Kubernetes系统创建对象所使用的名字空间
```

- `kube-public` 

```shell
这个名字空间是自动创建的，所有用户（包括未经过身份验证的用户）都可以读取它。 这个名字空间主要用于集群使用，以防某些资源在整个集群中应该是可见和可读的。 这个名字空间的公共方面只是一种约定，而不是要求。
```

- `kube-node-lease` 

```
此名字空间用于与各个节点相关的租期（Lease）对象； 此对象的设计使得集群规模很大时节点心跳检测性能得到提升。
```

## 为请求设置名字空间

```shell
kubectl run nginx --image=nginx --namespace=<名字空间名称>
kubectl get pods --namespace=<名字空间名称>
```

## 设置名字空间偏好

* 你可以永久保存名字空间，以用于对应上下文中所有后续`kubectl`命令。

```shell
[root@k8s-master ~]# kubectl config set-context --current --namespace=<名字空间名称>
[root@k8s-master ~]# kubectl config view | grep namespace:
```

## 创建名字空间

```shell
[root@k8s-master ~]# kubectl create namespace qiqi
[root@k8s-master ~]# kubectl create ns xiaobudianer
```

## 删除名字空间

```shell
[root@k8s-master ~]# kubectl delete namespace qiqi
[root@k8s-master ~]# kubectl delete ns xiaobudianer
```

## 查看哪些`Kubernetes`资源在名字空间中

```shell
[root@k8s-master ~]# kubectl api-resources --namespaced=true
```

## 查看哪些`Kubernetes`资源不在名字空间中

```shell
[root@k8s-master ~]# kubectl api-resources --namespaced=false
```

## 列出集群中现有的名字空间

```shell
[root@k8s-master ~]# kubectl get namespace
[root@k8s-master ~]# kubectl get namespaces
```

## 获取特定名字空间的摘要

```shell
[root@k8s-master ~]# kubectl get namespace <name>
[root@k8s-master ~]# kubectl get namespaces <name>
```

## 列出集群中现有的名字空间的详细信息

```shell
[root@k8s-master ~]# kubectl describe namespace
[root@k8s-master ~]# kubectl describe namespaces
```

## 获取特定名字空间的详细信息

```shell
[root@k8s-master ~]# kubectl describe namespace default
[root@k8s-master ~]# kubectl describe namespaces default
```

## 通过`yaml`文件创建名字空间

### 创建`yaml`文件

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: qiqi
```

### 应用`yaml`文件创建名字空间

```shell
[root@k8s-master ~]# kubectl apply -f qiqi.yaml
[root@k8s-master ~]# kubectl create -f qiqi.yaml
```

## 通过`yaml`文件删除名字空间

### 创建`yaml`文件

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: qiqi
```

### 通过`yaml`文件删除名字空间

```shell
[root@k8s-master ~]# kubectl delete -f qiqi.yaml
```

## 查看指定名字空间的`pod`

```shell
[root@k8s-master ~]# kubectl get pod -n default
[root@k8s-master ~]# kubectl get pods -n default
```



