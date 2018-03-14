---
categories: k8s

tags: 
  - k8s
  - kubernetes
  - kops

title: kops HTTP正向代理支持

date: 2018-03-14

---
# HTTP Forward Proxy Support

可以从http转发代理（“corporate proxy”）后面启动Kubernetes群集。 为此，需要为群集配置egressProxy。

假定代理已经存在。如果您想在AWS上使用私有拓扑网络，例如使用代理而不是NAT实例，则需要自行创建代理。 请参阅[在共享VPC中运行](https://github.com/kubernetes/kops/blob/master/docs/run_in_existing_vpc.md)。

此配置仅管理Kops和Kubernetes群集的代理配置。我们无法处理应用程序容器和pods的代理配置。

## Configuration

添加spec.egressProxy 端口和url

```yaml
spec:
  egressProxy:
    httpProxy:
      host: proxy.corp.local
      port: 3128
```

目前我们对http和https采用相同的配置。

## Proxy Excludes

除非另行配置，否则大多数客户端会盲目地尝试使用代理进行所有呼叫，甚至是本地主机和本地子网。 在初始群集创建时为您添加了成功启动和操作所需的一些基本排除项。 如果您希望添加额外的排除项，请使用逗号分隔的主机名列表的egressProxy.excludes上添加或编辑。基于后缀的匹配规则，即，corp.local将匹配images.corp.local，并且 .corp.local将匹配corp.local和images.corp.local，遵循典型的no_proxy环境变量约定。

```yaml
spec:
  egressProxy:
    httpProxy:
      host: proxy.corp.local
      port: 3128
    excludes: corp.local,internal.corp.com
```

## AWS VPC Endpoints and S3 access

如果您在AWS上托管已为S3或其他服务配置VPC“端点”，则可能需要将这些添加到spec.egressProxy.excludes。 请记住，S3存储桶必须与VPC位于相同的区域，才能通过端点访问。