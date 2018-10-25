---
categories: Kubernetes

tags: 
  - k8s
  - kubernetes

title: 使用 Source IP

date: 2018-06-25

---

# 使用 Source IP

在Kubernetes集群中运行的应用程序通过Service的抽象进行发现以及与外部世界和相互之间进行通信。本文档解释了发送到不同类型Service的数据包的源IP会发生什么事情，以及如何根据你的需求切换这些行为。

- 通过各种类型的服务公开一个简单的应用程序
- 了解每种服务类型如何处理源IP NAT
- 了解保留源IP所涉及的折衷

## 需要准备的东西

您需要有一个Kubernetes集群，并且必须将kubectl可以与您的集群进行通信。如果您还没有群集，可以使用Minikube创建一个群集，也可以使用其中一个Kubernetes playgrounds：

- [Katacoda](https://www.katacoda.com/courses/kubernetes/playground)
- [Play with Kubernetes](http://labs.play-with-k8s.com/)

检查版本：`kubectl version`.

## 配置文件