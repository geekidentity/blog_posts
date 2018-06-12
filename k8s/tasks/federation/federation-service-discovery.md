# 使用Federated Services 进行跨集群服务发现

本指南介绍了如何使用Kubernetes Federated Services跨多个Kubernetes集群部署服务。这让你可以轻松地为你的Kubernetes应用程序实现跨集群服务发现和可用区(AZ)容错。

## 准备

本指南假定您有一个正在运行的Kubernetes Cluster Federation。如果没有，请转到[federation 管理指南](https://kubernetes.io/docs/admin/federation/)以了解如何启动集群federation 。 其他教程，例如Kelsey Hightower的这个[教程](https://github.com/kelseyhightower/kubernetes-cluster-federation)也可以帮助你。

你也需要知道[Kubernetes的一些基本工作原理](https://kubernetes.io/docs/setup/)，尤其是[Service](https://kubernetes.io/docs/concepts/services-networking/service/)。

## 概览

Federated Services创建方式与传统[Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/) 非常相似，都是指定服务的所需属性，使用API调用。 对于Federated Services，此API调用将定向到Federation API端点，而不是Kubernetes群集API端点。 联邦服务的API与传统Kubernetes服务的API 100％兼容。

一旦创建， Federated Service将自动执行下面操作：

1. 在群集Federation中的每个集群中创建对应的Kubernetes服务，
2. 监控那些服务“shards”（以及它们所在的集群）的健康状况，以及
3. 在公共DNS提供商（例如Google Cloud DNS或AWS Route 53）中管理一组DNS记录，从而确保你的 federated service的客户端在发生群集、可用性区域或区域性中断时，也可以随时无缝定位正确的健康服务端点。

Federated Kubernetes群集中的客户端（即Pods）会自动在所在集群中找到Federated Service的本地shard （如果它存在并且健康），或者在不同集群中最近的健康shard （如果其所在集群中不存在健康的 shard）。

## 混合云功能

Kubernetes集群Federations 可以包括运行在不同云提供商（例如Google Cloud，AWS）和本地（例如OpenStack）中的集群。只需在相应的云提供商或本地创建所需的集群，并在Federation API服务器上注册每个群集的API端点和凭证（有关详细信息，请参阅[federation管理指南](https://kubernetes.io/docs/admin/federation/)）。

## 创建 federated service

创建命令如下：

```bash
kubectl --context=federation-cluster create -f services/nginx.yaml
```

–context=federation-cluster 标志告诉kubectl使用正确的证书将请求提交给Federation API端点。 如果您尚未配置此类上下文，请访问 federation管理指南或其中一个[管理教程](https://github.com/kelseyhightower/kubernetes-cluster-federation)，以了解如何执行此操作。

如上所述， Federated Service将自动在federation底层的所有集群中创建和维护一致的Kubernetes服务。

你可以通过进入每个集群来验证这一点，例如：

```shell
kubectl --context=gce-asia-east1a get services nginx
NAME      CLUSTER-IP     EXTERNAL-IP      PORT(S)   AGE
nginx     10.63.250.98   104.199.136.89   80/TCP    9m
```

上面假定您的客户端中为你的群集在该区域(zone)中配置了名为“gce-asia-east1a”的上下文。 底层服务的名称和名称空间将自动匹配您在上面创建的 Federated Service的名称和名称空间（如果您碰巧有任何这些群集中已存在相同名称和名称空间的服务，它们将自动使用Federation所采用的配置并进行更新，以符合您的 Federated Service 配置 - 无论哪种方式，最终结果将是相同的）。

 Federated Service的状态将自动反映基础Kubernetes服务的实时状态，例如：

```shell
$ kubectl --context=federation-cluster describe services nginx

Name:                   nginx
Namespace:              default
Labels:                 run=nginx
Annotations:            <none>
Selector:               run=nginx
Type:                   LoadBalancer
IP:                     10.63.250.98
LoadBalancer Ingress:   104.197.246.190, 130.211.57.243, 104.196.14.231, 104.199.136.89, ...
Port:                   http    80/TCP
Endpoints:              <none>
Session Affinity:       None
Events:                 <none>
```

