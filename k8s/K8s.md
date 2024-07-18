# K8s

## 一、基本概念-K8s简介

### 1. 应用部署方式

![image-20240616145251500](./assets/image-20240616145251500.png)

+ 传统部署方式：所有应用都部署在一台服务器上
+ 虚拟机部署方式：在一套硬件设备上模拟多个虚拟机，分别部署不同的应用服务
+ 容器化部署方式：将应用打成一个个容器镜像(docker)，以容器的形式部署

当部署的服务很大，比如由几百个容器组成，这就需要有一个对大量容器的统一的管理系统，即容器编排系统，k8s即是这样的作用

### 2. k8s的特性

k8s具有以下特性：

+ **服务发现和负载均衡**：k8s可以使用DNS名称或自己的IP地址公开容器，如果进入容器的流量很大，k8s可以负载均衡并分配网络流量，从而使部署稳定
+ **存储编排**：k8s允许自动挂载选择的存储系统，如本地存储，公有云提供商等
+ **自动部署和回滚**：可以使用k8s描述已部署容器的所需状态，它可以以受控的速率将实际状态更新为期望状态。比如容器上部署的服务为A版本，现在即将部署B版本，但是B版本本身有bug，现网不可用，k8s支持服务的回滚，将服务重置为A版本
+ **自动完成装箱计算**：k8s允许部署者指定每个容器所需的CPU和内存，当容器指定了资源请求时，k8s可以做出更好的决策来管理容器的资源
+ **自我修复**：k8s可以重新启动失败的容器，替换容器，杀死不响应用户定义的运行状况检查的容器
+ **弹性伸缩**：可以根据需要，自动对集群中正在运行的容器数量进行调整

k8s提供了一个可弹性运行分布式系统的框架，满足用户的扩展需求、故障转移、部署模式等

### 3. k8s的组件架构

一个k8s集群主要是由控制节点(master) 和工作节点(node)构成，每个几点上都会安装不同的组件

控制节点负责集群的决策

工作节点负责为容器提供运行环境

<img src="./assets/image-20240629125945996.png" alt="image-20240629125945996" style="zoom:80%;" />

结构介绍：

+ **API Server: **资源操作的唯一入口，接受用户输入的命令，提供认证，授权，API注册和发现等机制
+ **Controller Manager: **集群的决策者，负责维护集群的状态，比如程序部署安排，故障检测，自动扩展，滚动更新等
+ **ETCD: **k-v数据存储结构，存储整个集群的工作数据，负责存储集群中各种资源对象的信息
+ **Kubelet: **负责维护容器的生命周期，即通过控制容器化组件，来创建，更新，销毁容器
+ **Kube-proxy: **所有工作节点的访问入口，负责提供集群内部的服务发现和负载均衡
+ **Scheduler: **调度者，将服务分配给具体的工作节点，负责集群资源调度，按照预定的调度策略将pod调度到相应的node节点上

### 4. 部署k8s集群

> 每个机器的集群需要有docker的环境

+ 每个节点机器需要2G或者更多的内存
+ 每台机器需要至少2核CPU
+ 集群中的所有机器的网络彼此均能相互连通

前置操作

```powershell
#各个机器设置自己的主机名称
hostnamectl set-hostname master
#重启生效
reboot
#每台机器设置公网ip对应的主机名称
cat >> /etc/hosts << EOF
公网ip hostname
EOF
#重启网络
systemctl restart NetworkManager.service

# 开启ip转发
echo "1" >> /proc/sys/net/ipv4/ip_forward 
# 将 SELinux 设置为 permissive 模式（相当于将其禁用）
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

#关闭swap
swapoff -a  
sed -ri 's/.*swap.*/#&/' /etc/fstab

#允许 iptables 检查桥接流量
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
	
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
	
# 开启内核支持
cat >> /etc/sysctl.conf << EOF
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
	
sysctl -p
	
# 开启ipvs支持
yum -y install ipvsadm  ipset

#永久生效
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
```



在每台机器上配置k8s的yum源地址 安装 kubelet、kebeadm、kubectl

```powershell
# 配置 k8s 的 yum 源地址
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
   http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

在每个节点的机器上执行yum install下载kubelet

```powershell
# 安装 kubelet、kubeadm、kubectl
yum install -y kubelet-1.20.9 kubeadm-1.20.9 kubectl-1.20.9 --disableexcludes=kubernetes

# 启动kubelet
systemctl enable --now kubelet

# 查看 kubelet 状态：一会停止 一会运行。 这个状态是对的，kubelet 等待 kubeadm 发号指令。
systemctl status kubelet
```

配置镜像

```powershell
# 配置镜像，生成 images.sh
sudo tee ./images.sh <<-'EOF'
#!/bin/bash
images=(
kube-apiserver:v1.20.9
kube-proxy:v1.20.9
kube-controller-manager:v1.20.9
kube-scheduler:v1.20.9
coredns:1.7.0
etcd:3.4.13-0
pause:3.2
)
for imageName in ${images[@]} ; do
docker pull registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/$imageName
done
EOF

# 拉取镜像
chmod +x ./images.sh && ./images.sh
```



云服务器搭建集群的重点：设置虚拟网卡

```powershell
cat > /etc/sysconfig/network-scripts/ifcfg-eth0:1 <<EOF
BOOTPROTO=static
DEVICE=eth0:1
IPADDR=各个节点的公网ip
PREFIX=32
TYPE=Ethernet
USERCTL=no
ONBOOT=yes
EOF
```

修改`kubelet`启动参数(每个节点执行)

```
# 此文件安装kubeadm后就存在了
vim /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf

# 注意，这步很重要，如果不做，节点仍然会使用内网IP注册进集群
# 在末尾添加参数 --node-ip=公网IP

# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/sysconfig/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS --node-ip=各个节点的公网ip
```

主节点初始化

```powershell
kubeadm init \
--apiserver-advertise-address=主节点公网ip \
--control-plane-endpoint=cluster-endpoint \
--image-repository registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images \
--kubernetes-version v1.20.9 \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=192.168.0.0/16
```

```
##主节点初始化成功
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

  kubeadm join k8s-master:6443 --token elkum0.ghbv2b4i2vmuzjl0 \
    --discovery-token-ca-cert-hash sha256:b17cf51f51bc0a7fd4e9442618487201816d5edf79d3272d1526744a02ebe29b \
    --control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join k8s-master:6443 --token elkum0.ghbv2b4i2vmuzjl0 \
    --discovery-token-ca-cert-hash sha256:b17cf51f51bc0a7fd4e9442618487201816d5edf79d3272d1526744a02ebe29b 
```



## 二、资源管理

### 2.1 基本介绍

在k8s中，所有内容都抽象都称为资源，用户需要通过操作资源来管理k8s

> k8s本质上就是一个集群系统，用户可以在集群中部署各种服务，所谓的部署服务，气质就是在k8s集群中运行一个个容器，并将指定的程序跑在容器中
>
> k8s的**最小管理单元是pod**而不是容器，所以只能将容器放在pod中，而k8s一般也不会直接管理pod，而是通过pod控制器来管理pod
>
> pod可以提供服务之后，就要考虑如何访问pod中的服务，k8s提供了service资源实现这个功能
>
> 当然，如果pod中程序的数据需要持久化，k8s还提供了各种存储系统

### 2.2 资源管理方式

**命令式对象管理：**直接使用命令取操作k8s的资源

```
#启动一个pod 用于运行nginx镜像
kubectl run nginx-pod --image=nginx:1.17.1 --port 
```

**命令式对象配置：**通过命令配置和配置文件去操作k8s的资源

```
#pod的配置编写在nginx-pod.yml中
kubectl create/patch -f nginx-pod.yml
```

**声明式对象配置：**通过apply命令和配置文件去操作k8s资源

```
kubectl apply -f nginx-pod.yml
```

### 2.3 命令式对象管理

kubectl是k8s集群的命令行工具，通过它能够对集群本身进行管理，并能够在集群上进行容器化应用的安装部署

```
kubectl [command] [resource-type] [resource-name] [flags]
```

+ **command：**指定要对资源执行的操作，例如：create，get，delete
+ **type：**指定资源类型，比如deployment，pod，service
+ **name：**指定资源的名称，名称大小写敏感
+ **flags：**指定额外的可选参数

例如：

```
#查看集群中所有的pod信息
kubectl get pod

#查看指定pod的信息
kubectl get pod [pod name]

#将信息以json或者yaml的格式展示
kubectl get pod [pod name] -o [yaml/json]
```

#### 2.3.1 command

k8s允许对资源进行多种操作，可以通过`kubectl --help`查看所有的命令操作

常用命令如下：

**基本命令**

| 命令    | 翻译 | 命令作用     |
| ------- | ---- | ------------ |
| create  | 创建 | 创建一个资源 |
| edit    | 编辑 | 编辑一个资源 |
| get     | 获取 | 获取一个资源 |
| patch   | 更新 | 更新一个资源 |
| delete  | 删除 | 删除一个资源 |
| explain | 解释 | 展示资源文档 |

**运行和调试命令**
| 命令      | 翻译     | 命令作用                   |
| --------- | -------- | -------------------------- |
| run       | 运行     | 在集群中运行一个指定的镜像 |
| expose    | 暴露     | 暴露资源为Service          |
| describe  | 描述     | 显示资源内部信息           |
| logs      | 日志     | 输出容器在Pod中的日志      |
| attach    | 缠绕     | 进入运行中的容器           |
| exec      | 执行     | 执行容器中的一个命令       |
| cp        | 复制     | 在Pod内外复制文件          |
| rollout   | 首次展示 | 管理资源的发布             |
| scale     | 规模     | 扩（缩）容Pod的数量        |
| autoscale | 自动调整 | 自动调整Pod的数量          |

**高级命令**
| 命令  | 翻译 | 命令作用               |
| ----- | ---- | ---------------------- |
| apply | 应用 | 通过文件对资源进行配置 |
| label | 标签 | 更新资源上的标签       |

**其他命令**
| 命令         | 翻译     | 命令作用                     |
| :----------- | -------- | ---------------------------- |
| cluster-info | 集群信息 | 显示集群信息                 |
| version      | 版本     | 显示当前Client和Server的版本 |

#### 2.3.2 type

kubernetes中所有的内容都抽象为资源，可以通过`kubectl api-resources`进行查看：

常用资源如下：

**集群级别的资源**
| 资源名称   | 缩写 | 资源作用     |
| ---------- | ---- | ------------ |
| nodes      | no   | 集群组成部分 |
| namespaces | ns   | 隔离Pod      |

**pod资源**
| 资源名称 | 缩写 | 资源作用 |
| -------- | ---- | -------- |
| Pods     | po   | 装载容器 |

**pod控制器资源**
| 资源名称                 | 缩写   | 资源作用    |
| ------------------------ | ------ | ----------- |
| replicationcontrollers   | rc     | 控制Pod资源 |
| replicasets              | rs     | 控制Pod资源 |
| deployments              | deploy | 控制Pod资源 |
| daemonsets               | ds     | 控制Pod资源 |
| jobs                     |        | 控制Pod资源 |
| cronjobs                 | cj     | 控制Pod资源 |
| horizontalpodautoscalers | hpa    | 控制Pod资源 |
| statefulsets             | sts    | 控制Pod资源 |

**服务发现资源**
| 资源名称 | 缩写 | 资源作用        |
| -------- | ---- | --------------- |
| services | svc  | 统一Pod对外接口 |
| ingress  | ing  | 统一Pod对外接口 |

**存储资源**
| 资源名称               | 缩写 | 资源作用 |
| ---------------------- | ---- | -------- |
| volumeattachments      |      | 存储     |
| persistentvolumes      | pv   | 存储     |
| persistentvolumeclaims | pvc  | 存储     |

**配置资源**
| 资源名称   | 缩写 | 资源作用 |
| ---------- | ---- | -------- |
| configmaps | cm   | 配置     |
| secrets    |      | 配置     |

### 2.4 命令式对象配置

命令式对象配置就是使用命令配合配置文件一起来操作k8s

(1). 创建一个yaml文件，以启动nginx为例

```yaml
apiVersion: v1
kind: namespace
metadata: 
  name: dev
  
---

apiVersion: v1
kind: pod
metadata: 
  name: nginxPod
  namespace: dev
spec:
  containers:
  - name: nginx-containers
  image: nginx:1.17.1
```

(2). 执行create命令，创建资源：

```powershell
[root@master ~]# kubectl create -f ngincpod.yaml
namespace/dev created
pod/nginxpod created
```

此命令创建了两个资源对象，分别是namespace和pod

(3). 执行get命令，查看资源

```powershell
[root@master ~]# kubectl get -f ngincpod.yaml
NAME            STATUS     AGE
namespace/dev   Active     18s

NAME            READY      STATUS        RESTARTS     AGE
pod/nginxPod    1/1        Running        0            17s      
```

> 命令式对象配置的方式操作资源，可以简单的认为：命令 + yaml配置文件  
>
> yaml文件中是命令需要的各种参数



### 2.5 声明式对象配置

声明式对象配置跟命令式对象配置很相似，但是它只有一个命令apply

```powershell
#首先执行一次kubectl apply -f yaml文件，创建资源
[root@master ~]# kubectl apply -f ngincpod.yaml
namespace/dev created
pod/nginxpod created

#再执行一次kubectl apply -f yaml文件，发现资源没有变动
[root@master ~]# kubectl apply -f ngincpod.yaml
namespace/dev unchanged
pod/nginxpod unchangesd
```

> 声明式对象配置就是使用apply描述一个资源最终的状态，使用apply操作资源：
>
> + 如果资源不存在，就创建，执行`kubectl create`
> + 如果资源已存在，则更新，执行`kubectl patch`



## 三、k8s资源

### 3.1 Namespace

namespace是k8s系统中一种非常重要的资源，主要作用是用来实现多套环境的资源隔离，或者多租户的资源隔离

在默认情况下，k8s集群中的所有pod都是可以相互访问的，但在实际中，可能不想让两个pod之间进行相互访问，那此时就可以将两个pod划分到不同的namespace下。k8s通过将集群内部的资源分配到不同的namespace中，可以形成逻辑上的组，以方便不同的组的资源进行隔离使用和管理

可以通过k8s的授权机制，将不同的namespace交给不同的租户进行管理，这样就实现了多租户的资源隔离，此时还能结合k8s的资源配额机制，限定不同租户能占用的资源，例如cpu使用量，内存使用量等

<img src="./assets/image-20240707100220667.png" alt="image-20240707100220667" style="zoom:80%;" />



k8s集群启动之后会默认创建几个namespace，可以通过`kubectl get ns`命令查看

+ `default`: 所有未指定Namespace的对象都会被分配到default的命名空间
+ `kube-node-lease`: 集群节点之间的心跳维护
+ `kube-public`: 此命名空间下的资源可以被所有人访问
+ `kube-system`: 所有由k8s系统创建的资源都处于这个命名空间 (集群组件pod都在这个namespace下，包括kube-apiservice，kube-controller等)

**namespace的具体操作**





### 3.2 pod

Pod是k8s集群进行管理的最小单元，程序要运行必须部署在容器中，而容器必须存在于Pod中

Pod可以认为是容器的封装，一个Pod中可以存在一个或者多个容器

<img src="./assets/image-20240712095553197.png" alt="image-20240712095553197" style="zoom:80%;" />

k8s集群在启动之后，集群中的各个组件也都是以pod方式运行的，可以通过命令`kubectl get pod -n kube-system`查看

```powershell
[root@k8s-master ~]# kubectl get pod -n kube-system
NAME                                       READY   STATUS                  RESTARTS   AGE
calico-kube-controllers-577f77cb5c-86mz2   0/1     Pending                 0          61m
calico-node-2kw8k                          0/1     Init:ImagePullBackOff   0          20m
coredns-5897cd56c4-4dm2b                   0/1     Pending                 0          18h
coredns-5897cd56c4-55ggx                   0/1     Pending                 0          18h
etcd-k8s-master                            1/1     Running                 1          18h
kube-apiserver-k8s-master                  1/1     Running                 1          18h
kube-controller-manager-k8s-master         1/1     Running                 2          18h
kube-proxy-nq84s                           1/1     Running                 1          18h
kube-scheduler-k8s-master                  1/1     Running                 2          18h
```

k8s没有提供单独运行Pod的命令，都是通过Pod控制器来实现的

```powershell
#命令格式：kubectl run (pod控制器名称) [参数]
# --image  指定pod的镜像
# --port   指定端口
# --namespace  指定namespace
[root@k8s-master ~]# kubectl run nginx --image=nginx:1.17.1 --port=80 --namespace dev
deployment.apps/nginx created
```

查看pod的信息

```powershell
#kubectl get pods -n [namespace] -o wide
[root@k8s-master ~]# kubectl get pod -n default
NAME    READY   STATUS    RESTARTS   AGE
nginx   0/1     Pending   0          2m31s

#查看pod的详细信息
[root@k8s-master ~]# kubectl describe pod/nginx -n default
```

访问Pod

```powershell
#获取pod ip
[root@k8s-master ~]# kubectl get pods -n default -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP       NODE     NOMINATED NODE   READINESS GATES
nginx   0/1     Pending   0          8m15s   <none>   <none>   <none>           <none>
#访问pod
[root@k8s-master ~]# curl http://ip:port
```

删除pod

```powershell
#删除pod
[root@k8s-master ~]# kubectl get pod -n default
No resources found in default namespace.
```

### 3.3 Deployment

在k8s中，pod是最小的控制单元，但是k8s很少直接控制pod，一般都是通过pod控制器来完成的，pod控制器用于pod的管理，确保pod资源符合预期的状态，当pod的资源出现故障时，会尝试进行重启或者重建pod

在k8s中，pod控制器的种类有很多，deployment就是其中的一种

<img src="./assets/image-20240713093521610.png" alt="image-20240713093521610" style="zoom:80%;" />

deployment的基本命令

```powershell
#kubectl run deployment名称 [参数]   新版本k8s可以使用kubectl create deployment创建
# --image   指定pod镜像
# --port    指定端口
# --replicas指定创建pod的数量
# --namespace指定命名空间
[root@k8s-master ~]# kubectl run nginx --image=nginx:1.17.1 --port=80 --replicas=3 -n dev
deployment.apps/nginx created

#查看deployment信息
[root@k8s-master ~]# kubectl get deploy -n namspace
```



### 3.4 Label

Label是k8s系统中的一个重要概念，它的作用就是在资源上添加标识，用来对它们进行区分和选择

Label的特点: 

+ 一个Label会以key/value键值对的形式附加到各种对象上，如Node，Pod，Service等
+ 一个资源对象可以定义任意数量的Label，同一个Label也可以被添加到任意数量的资源对象上去
+ Label通常在资源对象定义时确定，当然也可以在对象创建后动态添加或者删除

可以通过Label实现资源的多维度分组，以便灵活，方便地进行资源分配，调度，配置，部署等管理工作

> 一些常用的Label：
>
> + 版本标签："version": "release"，"version": "stable"
> + 环境标签: "enviroment": "dev"，"env": "test"

标签定义完毕之后，还要考虑到标签的选择，这就要使用到Label Selector，Label Selector用于查询和筛选拥有某些标签的资源对象

当前有两种Label Selector：

+ 基于等式的Label Selector
  + name=slave：选择所有包含Label中key="name"且value="slave"的对象
  + env!=prod：选择所有包括Label中的key="env"且value!="prod"的对象
+ 基于集合的Label Selector
  + name in (master, slave)：选择所有包含Label中的key="name"且value="master"或者"slave"的对象
  + name not in(front)：选择所有包含Label中的key="name"且value!="front"的对象

标签基本命令

```powershell
#为pod资源打标签
[root@k8s-master ~]# kubectl label pod nginx-pod version=1.0 -n default
pod/nginx-pod labeled

#为pod资源更新标签
[root@k8s-master ~]# kubectl label pod nginx-pod verison=2.0 -n defalut --overwrite
pod/nginx-pod labeled

#查看标签
[root@k8s-master ~]# kubectl get pod -n default [-l version=2.0] --show-labels

#删除标签
[root@k8s-master ~]# kubectl label pod nginx-pod version- -n default
```

### 3.5 Service

k8s会为每个pod都分配一个单独的pod ip

+ pod ip会随着pod的重建产生变化
+ pod ip 仅仅是集群内可见的虚拟ip，外部无法访问

这样对于访问这个服务带来了难度，因此，k8s设计了service来解决这个问题

service可以看作是一组同类pod对外的访问接口，借助service，应用可以方便地实现服务发现和负载均衡

<img src="./assets/image-20240712104128381.png" alt="image-20240712104128381" style="zoom:80%;" />

创建集群内部可访问的service

```powershell
#暴露service
[root@k8s-master ~]# kubectl expose pod podName --name=service-name --type=ClusterIP --port=80 --target-port=80 -n namespace
service/service-name exposed

#查看service
[root@k8s-master ~]# kubectl get service -n namespace -o wide
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   19h   <none>


#在service的详细信息中有一个CLUSTER-IP，这就是service的ip，在service的生命周期中，这个地址不会改变
#可以通过访问这个ip访问当前service对应的pod
```

创建集群外部也可访问的service

```powershell
#上面创建的service的type类型为ClusterIP，这个ip地址只能在k8s集群内部访问
#如果需要创建外部也可以访问的Service，需要修改type为NodePort
[root@k8s-master ~]# kubectl expose pod podName --name=service-name --type=NodePort --port=80 --target-port=80 -n namespace
service/service-name exposed

#此时查看，会发现出现了NodePort类型的service，而且有一对Port clusterPort:publicPort
[root@k8s-master ~]# kubectl get service service-name -n namespace -o wide

##接下来就可以通过集群外的主机访问节点ip:publicPort访问服务了
```

## 四、Pod详解

### 4.1 Pod结构

每个Pod中都可以包含一个或者多个容器

<img src="./assets/image-20240713102520286.png" alt="image-20240713102520286" style="zoom:80%;" />

这些容器可以分为两类：

+ 用户程序所在的容器，数量可多可少
+ Pause容器，这是每个Pod都会有的一个根容器，作用有两个：
  + 可以以此容器为依据，评估整个pod的健康状态
  + 可以在根容器上设置ip地址，其他容器都使用这个ip，以实现pod内部的网络通信

### 4.2 Pod定义

一个相对完整的pod配置如下：

```yaml
apiVersion: v1     #必选，版本号，例如v1
kind: Pod       　 #必选，资源类型，例如 Pod
metadata:       　 #必选，元数据
  name: string     #必选，Pod名称
  namespace: string  #Pod所属的命名空间,默认为"default"
  labels:       　　  #自定义标签列表
    - name: string      　          
spec:  #必选，Pod中容器的详细定义
  containers:  #必选，Pod中容器列表
  - name: string   #必选，容器名称
    image: string  #必选，容器的镜像名称
    imagePullPolicy: [ Always|Never|IfNotPresent ]  #获取镜像的策略 
    command: [string]   #容器的启动命令列表，如不指定，使用打包时使用的启动命令
    args: [string]      #容器的启动命令参数列表
    workingDir: string  #容器的工作目录
    volumeMounts:       #挂载到容器内部的存储卷配置
    - name: string      #引用pod定义的共享存储卷的名称，需用volumes[]部分定义的的卷名
      mountPath: string #存储卷在容器内mount的绝对路径，应少于512字符
      readOnly: boolean #是否为只读模式
    ports: #需要暴露的端口库号列表
    - name: string        #端口的名称
      containerPort: int  #容器需要监听的端口号
      hostPort: int       #容器所在主机需要监听的端口号，默认与Container相同
      protocol: string    #端口协议，支持TCP和UDP，默认TCP
    env:   #容器运行前需设置的环境变量列表
    - name: string  #环境变量名称
      value: string #环境变量的值
    resources: #资源限制和请求的设置
      limits:  #资源限制的设置
        cpu: string     #Cpu的限制，单位为core数，将用于docker run --cpu-shares参数
        memory: string  #内存限制，单位可以为Mib/Gib，将用于docker run --memory参数
      requests: #资源请求的设置
        cpu: string    #Cpu请求，容器启动的初始可用数量
        memory: string #内存请求,容器启动的初始可用数量
    lifecycle: #生命周期钩子
		postStart: #容器启动后立即执行此钩子,如果执行失败,会根据重启策略进行重启
		preStop: #容器终止前执行此钩子,无论结果如何,容器都会终止
    livenessProbe:  #对Pod内各容器健康检查的设置，当探测无响应几次后将自动重启该容器
      exec:       　 #对Pod容器内检查方式设置为exec方式
        command: [string]  #exec方式需要制定的命令或脚本
      httpGet:       #对Pod内个容器健康检查方法设置为HttpGet，需要制定Path、port
        path: string
        port: number
        host: string
        scheme: string
        HttpHeaders:
        - name: string
          value: string
      tcpSocket:     #对Pod内个容器健康检查方式设置为tcpSocket方式
         port: number
       initialDelaySeconds: 0       #容器启动完成后首次探测的时间，单位为秒
       timeoutSeconds: 0    　　    #对容器健康检查探测等待响应的超时时间，单位秒，默认1秒
       periodSeconds: 0     　　    #对容器监控检查的定期探测时间设置，单位秒，默认10秒一次
       successThreshold: 0
       failureThreshold: 0
       securityContext:
         privileged: false
  restartPolicy: [Always | Never | OnFailure]  #Pod的重启策略
  nodeName: <string> #设置NodeName表示将该Pod调度到指定到名称的node节点上
  nodeSelector: obeject #设置NodeSelector表示将该Pod调度到包含这个label的node上
  imagePullSecrets: #Pull镜像时使用的secret名称，以key：secretkey格式指定
  - name: string
  hostNetwork: false   #是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
  volumes:   #在该pod上定义共享存储卷列表
  - name: string    #共享存储卷名称 （volumes类型有很多种）
    emptyDir: {}       #类型为emtyDir的存储卷，与Pod同生命周期的一个临时目录。为空值
    hostPath: string   #类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录
      path: string      　　        #Pod所在宿主机的目录，将被用于同期中mount的目录
    secret:       　　　#类型为secret的存储卷，挂载集群与定义的secret对象到容器内部
      scretname: string  
      items:     
      - key: string
        path: string
    configMap:         #类型为configMap的存储卷，挂载预定义的configMap对象到容器内部
      name: string
      items:
      - key: string
        path: string
```

可以通过如下命令查看每种资源的可配置项 

```powershell
# kubectl explain 资源类型   查看资源可配置的一级属性
# kubectl explain 资源类型.属性  查看属性的子属性
[root@k8s-master ~]# kubectl explain pod
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.

FIELDS:
   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind	<string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata	<Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec	<Object>
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

   status	<Object>
     Most recently observed status of the pod. This data may not be up to date.
     Populated by the system. Read-only. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

```

在k8s中基本所有资源的一级属性都是一样的，主要包含5部分：

+ apiVersion  <string>  版本，由k8s内部定义，版本号的取值范围可以通过`kubectl api-versions`查询
+ kind  <string>  类型，由k8s内部定义，类型的取值范围可以用`kubectl api-resources`查询
+ metadata  <Object>  元数据，主要是资源标识和说明，常用的有: name(资源名称)，namespace(资源所属的命名空间)，labels(资源的标签)等
+ spec  <Object>  资源描述，这是配置中最重要的一部分，里面是对各种资源配置的详细描述
+ status  <Object>  状态信息，里面的内容不需要定义，由k8s自动生成

其中spec属性是配置的重点，会有以下常见的子属性：

+ containers  <[]Object> 容器列表，用于定义容器的详细信息
+ nodeName  <string> 根据nodeName的值将pod调度到指定的Node节点上
+ nodeSelector  <map> 根据nodeSelector中定义的信息选择将该pod调度到包含这些label的Node上
+ hostNetWork <bool> 是否使用宿主机的网络模式，默认为false
+ volumes  <[]Object>  存储卷，用于定义Pod上面挂载的存储信息
+ restartPolicy  <string> 重启策略，表示在pod遇到故障时的处理策略  

### 4.3 Pod配置

查看`spec.containers`中的相关配置

```powershell
[root@k8s-master ~]# kubectl explain pod.spec.containers
KIND:     Pod
VERSION:  v1
RESOURCE: containers <[]Object>   # 数组，代表可以有多个容器FIELDS:
  name  <string>     # 容器名称
  image <string>     # 容器需要的镜像地址
  imagePullPolicy  <string> # 镜像拉取策略 
  command  <[]string> # 容器的启动命令列表，如不指定，使用打包时使用的启动命令
  args   <[]string> # 容器的启动命令需要的参数列表 
  env    <[]Object> # 容器环境变量的配置
  ports  <[]Object>  # 容器需要暴露的端口号列表
  resources <Object> # 资源限制和资源请求的设置
```

#### 4.3.1 基本配置

创建一个`pod-base.yml`文件，内容如下：

```yaml
apiVersion: v1
kind: pod
metadata:
  name: pod-base
  namespace: dev
  labels:
    user: progZhou
spec:
  containers:
    - name: nginx
      image: nginx:1.17.1
    - name: busybox
      image: busybox:1.30
```

上面定义了一个比较简单的pod配置，里面有两个容器：

+ nginx：用1.17.1版本的nginx镜像创建
+ busybox：用1.30版本的busybox镜像创建 (busybox是一个小巧的linux命令集合)

```powershell
#创建pod
[root@k8s-master ~]# kubectl apply -f pod-base.yml
pod/pod-base created

#查看pod状况
# READY m/n  表示当前pod中有n个容器，已经就绪了m个
# RESTARTS   重启次数
[root@k8s-master ~]# kubectl get pod -n dev
NAME                                 READY   STATUS                  RESTARTS   AGE
pod-base                             1/2     Running                 4          84m

#通过kubectl describe查看pod内部的详情
[root@k8s-master ~]# kubectl describe pod pod-base -n dev
```

#### 4.3.2 镜像拉取策略

imagePullPolicy 用于设置镜像拉取策略，k8s支持配置三种拉取策略：

+ `Always`：总是从远程仓库拉取镜像
+ `IfNotPresent`：本地有则使用本地镜像，本地没有则从远程仓库拉取镜像
+ `Never`：只使用本地镜像，从不去远程仓库拉取，本地没有就报错

> 默认值：
>
> + 如果镜像tag为具体的版本号，默认策略是IfNotPresent
> + 如果镜像tag为latest，默认策略是always

```powershell
[root@k8s-master ~]# kubectl explain pod.spec.containers
...
imagePullPolicy	<string>
     Image pull policy. One of Always, Never, IfNotPresent. Defaults to Always
     if :latest tag is specified, or IfNotPresent otherwise. Cannot be updated.
     More info:
     https://kubernetes.io/docs/concepts/containers/images#updating-images
...
```

#### 4.3.3 启动命令

busybox并不是一个程序，而是类似与一个工具类的集合，k8s集群启动管理后，它会自动关闭，所以会导致pod启动不成功

这就需要指定busybox的启动方式，解决方法就是让其一直运行，这就需要用到containers的command配置

```powershell
[root@k8s-master ~]# kubectl explain pod.spec.containers.command
KIND:     Pod
VERSION:  v1

FIELD:    command <[]string>

DESCRIPTION:
     Entrypoint array. Not executed within a shell. The docker image's
     ENTRYPOINT is used if this is not provided. Variable references $(VAR_NAME)
     are expanded using the container's environment. If a variable cannot be
     resolved, the reference in the input string will be unchanged. The
     $(VAR_NAME) syntax can be escaped with a double $$, ie: $$(VAR_NAME).
     Escaped references will never be expanded, regardless of whether the
     variable exists or not. Cannot be updated. More info:
     https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/#running-a-command-in-a-shell
```

创建一个pod-command.yml文件

```yaml
apiVersion: v1
kind: pod
metadata:
  name: pod-command
  namespace: dev
  labels:
    user: progZhou
spec:
  containers:
    - name: nginx
      image: nginx:1.17.1
    - name: busybox
      image: busybox:1.30
      command: ["/bin/sh", "-c", "touch /tmp/hello.txt;while true; do /bin/echo $(date +%T) >> /tmp/hello.txt; sleep 3; done;"]
```

command用于在pod中的容器初始化完毕之后运行一个命令

```powershell
#创建pod
[root@k8s-master ~]# kubectl create -f pod-command.yml
pod/pod-command created

#进入pod容器  
#命令格式：kubectl exec pod名称 -n 命名空间 -it -c 容器名称 /bin/sh   高版本k8s kubectl exec pod名称 --COMMAND参数
[root@k8s-master ~]# kubectl exec pod-command -n dev -it -c busybox /bin/sh
```

#### 4.3.4 环境变量

```powershell
[root@k8s-master manifests]# kubectl explain pod.spec.containers.env
KIND:     Pod
VERSION:  v1

RESOURCE: env <[]Object>

DESCRIPTION:
     List of environment variables to set in the container. Cannot be updated.

     EnvVar represents an environment variable present in a Container.

FIELDS:
   name	<string> -required-
     Name of the environment variable. Must be a C_IDENTIFIER.

   value	<string>
     Variable references $(VAR_NAME) are expanded using the previous defined
     environment variables in the container and any service environment
     variables. If a variable cannot be resolved, the reference in the input
     string will be unchanged. The $(VAR_NAME) syntax can be escaped with a
     double $$, ie: $$(VAR_NAME). Escaped references will never be expanded,
     regardless of whether the variable exists or not. Defaults to "".

   valueFrom	<Object>
     Source for the environment variable's value. Cannot be used if value is not
     empty.
```

创建pod-env.yaml

```yaml
apiVersion: v1
kind: pod
metadata:
  name: pod-command
  namespace: dev
  labels:
    user: progZhou
spec:
  containers:
    - name: nginx
      image: nginx:1.17.1
    - name: busybox
      image: busybox:1.30
      command: ["/bin/sh", "-c", "touch /tmp/hello.txt;while true; do /bin/echo $(date +%T) >> /tmp/hello.txt; sleep 3; done;"]
      env:  #环境变量实际就是一组键值对
      - name: "username"
        value: "prog"
      - name: "password"
        value: "123456"
```

env用于在pod中配置环境变量



#### 4.3.5 端口设置

```powershell
[root@k8s-master manifests]# kubectl explain pod.spec.containers.ports
KIND:     Pod
VERSION:  v1

RESOURCE: ports <[]Object>

DESCRIPTION:
     List of ports to expose from the container. Exposing a port here gives the
     system additional information about the network connections a container
     uses, but is primarily informational. Not specifying a port here DOES NOT
     prevent that port from being exposed. Any port which is listening on the
     default "0.0.0.0" address inside a container will be accessible from the
     network. Cannot be updated.

     ContainerPort represents a network port in a single container.

FIELDS:
   containerPort	<integer> -required-  #容器要监听的端口
     Number of port to expose on the pod's IP address. This must be a valid port
     number, 0 < x < 65536.

   hostIP	<string>  #要将外部端口绑定到主机ip 一般不做配置
     What host IP to bind the external port to.

   hostPort	<integer> #容器要在主机上公开的端口，为防止端口冲突，如果设置了此项，则主机上只能运行一个副本
     Number of port to expose on the host. If specified, this must be a valid
     port number, 0 < x < 65536. If HostNetwork is specified, this must match
     ContainerPort. Most containers do not need this.

   name	<string>   #端口号的名称，如果要指定名称，需要保证这个名称在pod中是唯一的
     If specified, this must be an IANA_SVC_NAME and unique within the pod. Each
     named port in a pod must have a unique name. Name for the port that can be
     referred to by services.

   protocol	<string>  #端口协议，必须是UDP TCP或者STCP 默认为TCP
     Protocol for port. Must be UDP, TCP, or SCTP. Defaults to "TCP".

```

创建pod-ports.yaml

```yaml
apiVersion: v1
kind: pod
metadata:
  name: pod-base
  namespace: dev
  labels:
    user: progZhou
spec:
  containers:
    - name: nginx
      image: nginx:1.17.1
      ports:
      - containerPort: 80
        name: nginx-port
        protocol: TCP
```

如果要访问容器中的程序，需要通过`podId:containerPort`进行访问



#### 4.3.6 资源配额

容器中的程序要运行，需要占用一定的资源，比如CPU和内存等，如果不对某个容器的资源做限制，那么在程序运行异常或者需要大量资源程序的运行过程中会吃掉大量的机器资源，导致其他容器无法运行，针对这种情况，k8s提供了对内存和cpu资源进行配额的机制，这种机制主要通过resources配置项实现

+ `limits`：用于限制运行时容器的最大占用资源，当容器占用资源超过limits设置的值时会被终止，并进行重启
+ `requests`：用于设置容器需要的最小资源，如果环境资源不够，容器将无法启动

```powershell
[root@k8s-master manifests]# kubectl explain pod.spec.containers.resources
KIND:     Pod
VERSION:  v1

RESOURCE: resources <Object>

DESCRIPTION:
     Compute Resources required by this container. Cannot be updated. More info:
     https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/

     ResourceRequirements describes the compute resource requirements.

FIELDS:
   limits	<map[string]string>
     Limits describes the maximum amount of compute resources allowed. More
     info:
     https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/

   requests	<map[string]string>
     Requests describes the minimum amount of compute resources required. If
     Requests is omitted for a container, it defaults to Limits if that is
     explicitly specified, otherwise to an implementation-defined value. More
     info:
     https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/
```

创建pod-resources.yaml

```yaml
apiVersion: v1
kind: pod
metadata:
  name: pod-base
  namespace: dev
  labels:
    user: progZhou
spec:
  containers:
    - name: nginx
      image: nginx:1.17.1
      ports:
      - containerPort: 80
        name: nginx-port
        protocol: TCP
      resoueces:
        limits:  #需要资源的上限，不能超过这个值
          cpu: "2"
          memory: "10Gi"
        requests: #需要资源的下限，不能小于这个值
          cpu: "1"
          memory: "10Mi"
```

> 当前k8s只支持对cpu核数和内存大小进行限制

### 4.4 Pod生命周期

pod的生命周期指的是pod对象从创建到终止的这段时间范围，主要包含几个过程：

+ pod创建过程
+ 运行【初始化容器】的过程
+ 运行【主容器】的过程
  + 容器启动后钩子(post start)、容器终止前钩子(pre stop)
  + 容器存活性探测(liveness probe)、就绪性探测(readiness probe)
+ pod终止过程

<img src="./assets/image-20240718192624468.png" alt="image-20240718192624468" style="zoom:80%;" />

在pod的生命周期中，会出现5种状态：

+ 挂起(Pending)：apiserver已经创建了pod资源对象，但它尚未被调度完成或者仍处于下载镜像的过程中
+ 运行中(Running)：pod已经被调度至某节点，并且所有容器都已经被kubelet创建完成
+ 成功(Succeeded)：pod中的所有容器都已经成功终止并且不会被重启
+ 失败(Failed)：所有容器都已经终止，但至少有一个容器终止失败，即容器返回了非0值的退出状态
+ 未知(Unkonwn)：apiserver无法正常获取到pod对象的状态信息，通常由网络通信失败所致

#### 4.4.1 Pod创建和终止

**Pod创建过程**

<img src="./assets/image-20240718193321931.png" alt="image-20240718193321931" style="zoom:80%;" />

+ (1). 用户通过kubectl或其他api客户端提交需要创建的pod信息给apiserver (`kubectl apply -f / kubectl run xxx`)
+ (2). apiserver开始生成pod对象的信息，并将信息存入etcd，然后返回确认信息至客户端
+ (3). apiserver开始反映etcd中的pod对象的变化，其他组件使用watch机制来跟踪检查apiserver上的变动
+ (4). scheduler发现有新的pod对象要创建，开始为pod分配主机并将结果信息更新至apiserver
+ (5). node节点上的kubelet发现有pod调度过来，尝试调用docker启动容器，并将结果返回给apiserver
+ (6). apiserver将接收到的pod状态信息存入etcd中

**Pod终止过程**

+ (1). 用户向apiserver发送删除pod的指令 (`kubectl delete pod xxx`)
+ (2). apiserver中的pod对象信息会随着时间的推移而更新，在宽限期内(一般为30s)，pod被视为dead状态
+ (3). 将pod标记为terminating状态(正在删除)
+ (4). kubelet在监控到pod对象转为terminating状态的同时启动pod关闭过程
+ (5). 端点控制器监控到pod对象的关闭行为时将其从所有匹配到此端点的service资源的端点列表中移除
+ (6). 如果当前pod对象定义了preStop钩子处理器，则在其标记为terminating后即会以同步的方式启动执行
+ (7). pod对象中的容器进程收到停止信号
+ (8). 宽限期结束后，若pod中还存在仍在运行的进程，那么pod对象会收到立即终止的信号
+ (9). kubelet请求apiserver将此pod资源的宽限期设置为0从而完成删除操作
