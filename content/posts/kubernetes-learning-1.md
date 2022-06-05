---
title: Kubernetes 历史/集群架构
date: '2022-06-05'
tags: ['kubernetes']
category: etc
---

## 部署技术演进历史

![](https://d33wubrfki0l68.cloudfront.net/26a177ede4d7b032362289c6fccd448fc4a91174/eb693/images/docs/container_evolution.svg)

- 传统部署：在一个物理机上直接部署多个app。资源抢占问题和依赖冲突问题严重。
- 虚拟化部署：在一个物理机上运行多个 VM。解决了资源抢占问题和依赖冲突问题，但是带来了性能问题。
- 容器部署：在一个物理机上部署多个容器，容器之间共享 OS，相比 VM 性能开销更低，性能更好。

相比于直接在物理机上部署 Docker ，运维人员需要手动处理负载均衡、存储编排、部署和回滚、故障重启等问题。以上问题都可以交给 Kubernetes 解决。

> Kubernetes/k8s 是一个开源容器编排平台，可以自动完成在部署、管理和扩展容器化应用过程中涉及的许多手动操作。

## Kubernetes 集群架构

![](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)

- 控制面组件
    - kube-apiserver: 公开了 Kubernetes API，是 Kubernetes 控制面的前端
    - etcd: 分布式KV数据库，保存 Kubernetes 所有集群数据的后台数据库
    - kube-scheduler: 监视新创建的、未指定运行 node 的 Pods，选择节点让 Pod 在上面运行。
    - kube-controller-manager: 包括 Node Controller、Job controller、Endpoints Controller、Service Account & Token Controllers
    - cloud-controller-manager: 特定于云平台的控制回路
- node 组件 ( 也叫 worker node )
    - kubelet: node 上运行的代理，接收一组通过各类机制提供给它的 PodSpecs，确保这些 PodSpecs 中描述的容器处于运行状态且健康
    - kube-proxy: node 上运行的网络代理，维护网络规则，允许从集群内部或外部的网络会话与 Pod 进行网络通信
    - Container Runtime: 负责运行容器，事实标准是 Docker
- Addons: 使用 Kubernetes 资源实现集群功能，Addons 中命名空间域的资源属于 kube-system 命名空间。
    - DNS: DNS 服务器
    - Dashboard: Web 的用户界面
    - 容器资源监控: 将关于容器的一些常见的时间序列度量值保存到一个集中的数据库中，并提供用于浏览这些数据的界面
    - 集群层面日志: 负责将容器的日志数据 保存到一个集中的日志存储中，该存储能够提供搜索和浏览接口


## 通过cli访问集群

在得到一个可用的 k8s 集群后，可以使用如下命令进行验证：

```bash
# save connection configuration yaml in ~/.kube/config/xxx.yaml
$ kubectl --kubeconfig ~/.kube/config/xxx.yaml get node
NAME            STATUS   ROLES    AGE   VERSION
172.24.66.222   Ready    <none>   17d   v1.20.15
```

如果可以看到所示节点信息和版本，即可说明 k8s 集群搭建完成

## 参考资料 

- https://kubernetes.io