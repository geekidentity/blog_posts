---
categories: k8s

tags: 
  - k8s
  - kubernetes
  - kops
  - federation

title: Kubernetes 联邦机制介绍

date: 2018-03-17

---

# Federation（联邦）

此页面解释了为什么以及如何使用联邦来管理多个Kubernetes集群。

- Why federation
  - [Caveats](https://kubernetes.io/docs/concepts/cluster-administration/federation/#caveats)
  - [Hybrid cloud capabilities](https://kubernetes.io/docs/concepts/cluster-administration/federation/#hybrid-cloud-capabilities)
- [Setting up federation](https://kubernetes.io/docs/concepts/cluster-administration/federation/#setting-up-federation)
- [API resources](https://kubernetes.io/docs/concepts/cluster-administration/federation/#api-resources)
- [Cascading deletion](https://kubernetes.io/docs/concepts/cluster-administration/federation/#cascading-deletion)
- [Scope of a single cluster](https://kubernetes.io/docs/concepts/cluster-administration/federation/#scope-of-a-single-cluster)
- [Selecting the right number of clusters](https://kubernetes.io/docs/concepts/cluster-administration/federation/#selecting-the-right-number-of-clusters)
- [What’s next](https://kubernetes.io/docs/concepts/cluster-administration/federation/#whats-next)

## 为什么联邦

联合可以轻松管理多个群集。 它通过提供2个主要构件来实现：

- 跨群集同步资源：联邦可以使多个群集中的资源保持同步。 例如，可以确保多个群集中部署相同的程序。


- 跨群集发现：联邦提供了自动配置DNS服务器和负载均衡器与所有群集后端的功能。例如，您可以确保可以使用全局VIP或DNS记录来访问多个群集的后端。

联邦的一些其它用处如下：

- 高可用：通过在群集之间传播负载并自动配置DNS服务器和负载平衡器，联邦会将群集故障的影响降至最低。
- 避免提供者锁定(lock-in)：通过更轻松地跨群集迁移应用程序，联邦会阻止群集提供者锁定(lock-in)。

除非有多个集群，否则联邦并没有任何用。你可能需要多个集群的一些原因有：

- 低延迟：让多个区域中的集群通过向距离它们最近的集群提供服务来最大限度地减少延迟。
- 故障隔离：最好有多个小型集群而不是一个单独人大型集群来进行故障隔离（例如：云提供商的不同可用区域中有多个集群）。
- 可扩展性：单个kubernetes集群具有可扩展性限制（大多数用户不应该这样做，更多详情请参阅[Kubernetes Scaling和Performance Goals](https://git.k8s.io/community/sig-scalability/goals.md)）。
- [混合云](https://kubernetes.io/docs/concepts/cluster-administration/federation/#hybrid-cloud-capabilities)：可以在不同的云提供商或本地数据中心上拥有多个群集。

### 注意事项

虽然联邦有很多有吸引力的用处，但也有一些注意事项：

- 增加网络带宽和成本：联邦控制台监视所有群集以确保当前状态符合预期。如果集群在云提供商或不同云提供商的不同区域(regions)运行，这可能会导致显着的网络成本。
- 减少跨群集隔离：联邦控制台中的错误可能影响所有群集。通过将联邦控制台中的逻辑保持最简，可以缓解这一问题。 只要可能，它大部分都会委托给控制台的kubernetes集群中。 设计和实施也在安全方面做了很多考虑，并避免发生错误时多集群停机。
- 成熟度：联邦项目相对较新，不太成熟。 并非所有资源都可用，许多资源仍然是alpha状态。  [Issue 88](https://github.com/kubernetes/federation/issues/88)列举了团队忙于解决的系统已知问题。

### 混合云功能

Kubernetes集群联邦可以运行在不同云提供商（例如Google Cloud，AWS）和本地（例如OpenStack）中的集群。 Kubefed是部署联邦集群的推荐方式。

此后，您的API资源可以跨越不同的集群和云提供商。

## 设置联邦

为了能够联合多个集群，首先需要设置联邦控制台。按照[设置指南](https://kubernetes.io/docs/tutorials/federation/set-up-cluster-federation-kubefed/)进行设置。

## API 资源

一旦设置了控制台，就可以开始创建联邦API资源。 以下指南详细解释了一些资源：

- [Cluster](https://kubernetes.io/docs/tasks/administer-federation/cluster/)
- [ConfigMap](https://kubernetes.io/docs/tasks/administer-federation/configmap/)
- [DaemonSets](https://kubernetes.io/docs/tasks/administer-federation/daemonset/)
- [Deployment](https://kubernetes.io/docs/tasks/administer-federation/deployment/)
- [Events](https://kubernetes.io/docs/tasks/administer-federation/events/)
- [Hpa](https://kubernetes.io/docs/tasks/administer-federation/hpa/)
- [Ingress](https://kubernetes.io/docs/tasks/administer-federation/ingress/)
- [Jobs](https://kubernetes.io/docs/tasks/administer-federation/job/)
- [Namespaces](https://kubernetes.io/docs/tasks/administer-federation/namespaces/)
- [ReplicaSets](https://kubernetes.io/docs/tasks/administer-federation/replicaset/)
- [Secrets](https://kubernetes.io/docs/tasks/administer-federation/secret/)
- [Services](https://kubernetes.io/docs/concepts/cluster-administration/federation-service-discovery/)

[API参考文档](https://kubernetes.io/docs/reference/generated/federation/)列出了联邦apiserver支持的所有资源。

## 级联删除

Kubernetes 1.6版支持级联删除联邦资源。当从联邦控制台中删除资源时，还会删除所有基础集群中的相应资源。

在使用REST API时，级联删除在默认情况下不会启用。要启用它，请在使用REST API从联邦控制台中删除资源时设置DeleteOptions.orphanDependents=false选项。 使用kubectl delete可以在默认情况下启用级联删除。还可以通过运行kubectl delete --cascade=false来禁用它

注意：Kubernetes版本1.5包括对联邦资源子集的级联删除支持。

## 单个群集的范围

在诸如Google Compute Engine或Amazon Web Services之类的IaaS供应商中，虚拟机存在于zone 或AZ中。 我们建议Kubernetes集群中的所有虚拟机应位于相同的可用区域中，因为：

- 与具有单个全局Kubernetes集群相比，单点故障的数量更少。
- 与跨越可用区域的集群相比，更容易推断单区域群集的可用性属性。
- 当Kubernetes开发人员正在设计系统时（例如对延迟，带宽或相关故障进行假设），他们假设所有机器都位于单个数据中心或连接非常近。

建议在每个可用区域运行更少的虚拟机群集; 但可以在每个可用区域运行多个群集。

选择每个可用区域较少群集的理由是：

- 在某些情况下，在一个群集中有更多节点（更少的资源），可以改进Pod的装箱包装。
- 降低了运维开销（尽管随着操作工具和流程的成熟，优势没那么明显了）。
- 降低每个集群固定资源花费的成本，例如， apiserver虚拟机（但对于大中型集群整体集群成本的比例很小）。

有多个集群的原因包括：

- 严格的安全策略要求将一类工作与另一类工作隔离（但请参阅下面的分区集群）
- 测试群集canary 到新Kubernetes版本或其他群集软件。

## 选择正确数量的集群

选择Kubernetes集群的数量一般是不会变的，只是偶尔会重新审视(revisited occasionally)。 相比之下，集群中的节点数量和服务中的pod数量可能会随着负载和业务增长频繁变化。

要选择集群数量，首先需要确定您需要在哪些区域(region)进行部署，以便在Kubernetes上运行的服务为所有最终用户提供最低的延迟，（如果使用内容分发网络，则CDN- 托管内容不需要考虑）。 法律问题也可能会对此产生影响。 例如，一家拥有全球客户群的公司可能会决定在美国，欧盟，美联社和南非地区拥有集群。需要使用的区域的数量为R。

其次，确定在不影响整体业务的情况下，最多可以容忍有多少个群集同时不可用。将不最多不可用集群数量设为U.如果您不确定，那么U=1是一个不错的选择。

如果允许负载平衡在发生集群故障时将流量引导至任何区域，则至少需要较大的R或U + 1集群。 如果不是（例如，如果要确保发生群集故障时所有用户的延迟较低），则需要具有R *（U + 1）群集（每个R区域中的U + 1）。 无论如何，尝试将每个集群放入不同的区域。

最后，如果需要构建一个比Kubernetes的最大建议节点数量多的集群，则可能需要多个群集。 Kubernetes v1.3支持最多1000个节点的群集。 Kubernetes v1.8支持多达5000个节点的集群。 有关更多指导，请参阅[构建大型集群](https://kubernetes.io/docs/admin/cluster-large/)。

## What’s next

- 详细了解[联邦方案](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/multicluster/federation.md)。
- 请参阅本[设置指南](https://kubernetes.io/docs/tutorials/federation/set-up-cluster-federation-kubefed/)了解群集联邦。
- See this [Kubecon2016 talk on federation](https://www.youtube.com/watch?v=pq9lbkmxpS8)
- See this [Kubecon2017 Europe update on federation](https://www.youtube.com/watch?v=kwOvOLnFYck)