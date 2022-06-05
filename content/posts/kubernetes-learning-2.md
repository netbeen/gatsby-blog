---
title: Kubernetes Pod/Deployment/Service/Ingress
date: '2022-06-05'
tags: ['kubernetes']
category: etc
---

## Pod

Pod 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。

创建 Pod 示例：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

```bash
$ kubectl apply -f https://k8s.io/examples/pods/simple-pod.yaml
```
上述命令创建了一个 Pod ，其中包含了一个运行 nginx 容器

> 一个 Pod 里可以包含多个业务容器，但是通常情况下只包含一个业务容器。除了业务容器， Pod 里还有一个名为 pause 的系统容器。

## Deployment

通过直接创建 Pod 的方式管理应用，无法使用 k8s 的自愈能力，即当某个 Pod 失效后，在合适的 node 上再次创建一个 Pod ,以保持 Pod 数量满足期望。

一般使用工作负载在管理 Pod，最常见的一个工作负载是 Deployment。

示例：该 Deployment 创建了一个 ReplicaSet，负责启动三个 nginx Pods，并且维持的 Pod 数量满足期望。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

```bash
$ kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
```

## Service

通过 Deployment 解决了 Pod 的数量保持后，我们需要面对如何将服务暴露给其他 Pod 使用的问题。

Kubernetes Service 定义了这样一种抽象：逻辑上的一组 Pod，一种可以访问它们的策略 —— 通常称为微服务。

示例：创建一个名称为 "my-service" 的 Service 对象，它会将请求代理到使用 TCP 端口 9376，并且具有标签 "app=MyApp" 的 Pod 上，同时完成负载均衡。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

### DNS

Kubernetes 为 Service 和 Pods 创建 DNS 记录。当对象为 Service 时，主机名为 service-name （ 同 namespace 时 ），或者 service-name.namespace（ 不同 namespace 时 ）

## Ingress

DNS 解决了 Service 间互相调用的问题，Ingress 则公开了从集群外部到集群内 Service 的 HTTP 和 HTTPS 路由。

Ingress 可为 Service 提供外部可访问的 URL、负载均衡流量、终止 SSL/TLS，以及基于名称的虚拟托管。 Ingress Controller 通常负责通过负载均衡器来实现 Ingress，尽管它也可以配置边缘路由器或其他前端来帮助处理流量。

你必须拥有一个 Ingress Controller 才能满足 Ingress 的要求。 仅创建 Ingress 资源本身没有任何效果。

示例：
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80
```

### Ingress Controller

为了让 Ingress 资源工作，集群必须有一个正在运行的 Ingress 控制器。

## 参考资料 

- https://kubernetes.io