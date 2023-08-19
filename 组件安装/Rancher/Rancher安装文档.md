# Rancher安装文档

[TOC]

## 一 概述

​	`Rancher`是一个为使用容器的公司打造的容器管理平台。`Rancher`使得开发者可以随处运行`kubernetes`，满足IT需求规范，赋能devops团队。

## 二 离线Helm CLI安装

​	本文介绍如何使用helm cli在离线环境中安装`rancher server`。rancher安装在rek kubernetes集群、k3s集群、或者耽搁docker容器上对应的安装步骤也会有所不同，本文着重介绍k3s集群搭建rancher server的过程以及需要注意的事项。

### 安装摘要

离线安装主要分为以下四个步骤：

+ [设置基础设施和私有就像仓库](#1-设置基础设施和私有镜像仓库)
+ [收集镜像到私有仓库](#2-收集镜像到私有仓库)
+ [安装Kubernetes集群（k3s集群）](#3-安装Kubernetes集群（k3s集群）)
+ [安装rancher](#4-安装rancher)

### 1 设置基础设施和私有镜像仓库

为了实现高可用安装，建议设置以下基础设施：

+ **2个Linux节点** ：可以是云服务器或者是私有虚拟机。
+ **1个外部数据库** ：用于存储集群数据。支持postgresql、mysql、etcd。
+ **一个负载均衡器** ： 用于将流量转发到两个节点中。这里一般使用nginx。
+ **1个DNS** ：用于将url映射到负载均衡器。此dns记录将成为rancher server的URL，下游集群需要可以访问到这个地址。
+ **私有镜像仓库** ：用于将容器镜像分别发送到你的主机。

#### 额外配置设置

> a.配置Linux节点
>
> ​	在安裝之前还需要一些准备工作，比如设置linux系统参数、关闭防火墙、设置服务器时间同步等。具体参照[Linux常规配置]()。
>
> **b.配置外部数据库**
>
> ​	K3s 与其他 Kubernetes 发行版不同，在于其支持使用 etcd 以外的数据库来运行 Kubernetes。该功能让 Kubernetes 运维更加灵活。你可以根据实际情况选择合适的数据库。这里我们选择mysql作为外部数据存储。
>
> 再安装kubernetes时，需要传入k3s连接数据库的详细信息。
>
> ###### 设置外部数据库
>
> > --datastore-endpoint="mysql://root:zzy@123@tcp(192.168.201.75:33306)/k3s"

#### 配置外部数据库

> 

在每个节点上启动ks3之前，将tar文件放到`images`目录中。

~~~sh
sudo mkdir -p /var/lib/rancher/k3s/agent/images/
sudo cp ./k3s-airgap-images-amd64.tar /var/lib/rancher/k3s/agent/images/
~~~

创建镜像仓库配置

把 `registries.yaml` 文件创建到 `/etc/rancher/k3s/registries.yaml` 中。此文件为 K3s 提供连接到你的私有镜像仓库的详细信息。详情见附件。

安装k3s：

~~~sh
INSTALL_K3S_SKIP_DOWNLOAD=true ./install.sh --datastore-endpoint="mysql://root:zzy@123@tcp(192.168.201.75:33306)/k3s"
~~~

保存并使用kubeconfig文件

在每个 Rancher Server 节点安装 K3s 时，会在每个节点的 `/etc/rancher/k3s/k3s.yaml` 中生成一个 `kubeconfig` 文件。该文件包含访问集群的凭证。请将该文件保存在安全的位置。

如要使用该 `kubeconfig` 文件：

+ 安装 Kubernetes 命令行工具 [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl)。
+ 复制 `/etc/rancher/k3s/k3s.yaml` 文件并保存到本地主机的 `~/.kube/config` 目录上。
+ 在 kubeconfig 文件中，`server` 的参数为 localhost。你需要将 `server` 配置为负载均衡器的 DNS，并指定端口 6443（通过端口 6443 访问 Kubernetes API Server，通过端口 80 和 443 访问 Rancher Server）。以下是一个 `k3s.yaml` 示例：

~~~yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: [CERTIFICATE-DATA]
    server: [LOAD-BALANCER-DNS]:6443 # 编辑此行
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    password: [PASSWORD]
    username: admin
~~~

### 2 安装Rancher

#### 2.1 安装helm chart仓库

[安装helm](../Helm安装.md)

#### 2.2 添加TLS密文

我们使用证书和密钥将 `cattle-system` 命名空间中的 `tls-rancher-ingress` 密文配置好后，Kubernetes 会为 Rancher 创建对象和服务。

将服务器证书和所需的所有中间证书合并到名为 `tls.crt`的文件中。将证书密钥复制到名为 `tls.key` 的文件中。[证书安装](../../服务器相关/openssl生成证书.md)

使用 `kubectl` 创建 `tls` 类型的密文。

~~~sh
kubectl -n cattle-system create secret tls tls-rancher-ingress \
  --cert=tls.crt \
  --key=tls.key
~~~

> 备注
>
> 如需替换证书，你可以运行 `kubectl -n cattle-system delete secret tls-rancher-ingress` 来删除 `tls-rancher-ingress` 密文，然后运行上方命令来添加新的密文。如果你使用的是私有 CA 签名证书，仅当新证书与当前证书是由同一个 CA 签发的，才可以替换。

使用私有ca签名证书

如果你使用的是私有 CA，Rancher 需要私有 CA 的根证书或证书链的副本，Rancher Agent 使用它来校验与 Server 的连接。

创建一个名为 `cacerts.pem` 的文件，该文件仅包含私有 CA 的根 CA 证书或证书链，并使用 `kubectl` 在 `cattle-system` 命名空间中创建 `tls-ca` Secret。

~~~sh
kubectl -n cattle-system create secret generic tls-ca \
  --from-file=cacerts.pem=./cacerts.pem
~~~

安装rancher

~~~sh
helm install rancher ./rancher-2.6.10.tgz \
    --namespace cattle-system \
	--set rancherImageTag=v2.6.10 \
    --set hostname=rancher.eason.com \
    --set rancherImage=registry.eason.com:8082/rancher/rancher \
    --set ingress.tls.source=secret \
    --set privateCA=true \
    --set systemDefaultRegistry=registry.eason.com:8082 \
    --set useBundledSystemChart=true
~~~



## 附件

`registries.yaml`

~~~yaml
mirrors:
  "*":
    endpoint:
      - "http://registry.eason.com:8082"
  docker.io:
    endpoint:
      - "http://registry.eason.com:8082"
~~~



