# rancher安装文档

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





