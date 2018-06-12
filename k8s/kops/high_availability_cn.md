# 高可用 (HA)

## 介绍

Kubernetes有两种高可用策略：

- 运行多个独立群集并将它们组合在一个管理平台上：[联邦](https://kubernetes.io/docs/user-guide/federation/)


- 在多个云区的zones中运行单个群集，并使用冗余组件

与使用冗余组件运行相比，kops对集群有更好的支持。 kops能够创建多个kubernetes master结点，因此如果master 实例发生故障，kubernetes API可以继续运行。

但是，当使用单个master节点运行kubernetes时，如果master节点失败，则kubernetes API将不可用，但在（不受影响的）节点上运行的pod和service应继续运行。 在这种情况下，我们将无法做任何涉及API的事情（添加节点，缩放pod，替换已终止的pod），并且kubectl将不起作用。 但是，你的应用程序应该继续运行，并且大多数应用程序可能会忍受一小时或更长时间的API停机。

而且，kops以自动替换模式运行master。 master在auto-scaling组中运行，数据位于EBS卷上。 如果主节点终止，ASG将启动一个新的master实例，并且kops将挂载master卷到替换的master节点上。

简而言之：

- 单个master结点的Kops集群仍然可用; 如果master实例终止，它将被自动替换。但是EBS的使用将我们绑定到单一的AZ，并且在AZ长时间停机的情况下，我们可能会遇到停机时间。


- 多节点kops群集可以容忍单个AZ的中断

## 使用Kops HA

我们可以使用kops创建HA群集，但要注意到从单master集群迁移到多master集群是一个复杂的操作（此处[介绍](https://github.com/kubernetes/kops/blob/master/docs/single-to-multi-master.md)）很重要。所以尽可能在集群创建时就规划好。

当第一次调用kops创建群集时，可以指定--master-zones标志，列出希望master结点运行的区域，例如：

```bash
kops create cluster \
    --node-count 3 \
    --zones us-west-2a,us-west-2b,us-west-2c \
    --master-zones us-west-2a,us-west-2b,us-west-2c \
    --node-size t2.medium \
    --master-size t2.medium \
    --topology private \
    --networking kopeio-vxlan \
    hacluster.example.com
```

Kubernetes依赖于一个名为“etcd”的键值存储，它使用Quorum方法来保持一致性，所以如果有51％的节点可用，它就可用。

因此，在建设HA集群中使用kops时需要考虑一些注意事项：

- 应该创建奇数个master实例。


- Kops具有在同一个AZ中运行多个master的实验性支持，但应谨慎使用。 如果我们在同一个AZ中创建2个（或更多）master实例，那么AZ的故障可能会导致etcd丢失仲裁并停止运行（3个节点）。 因此，运行在同一个AZ中增加了群集中断的风险，尽管它可能是一个有效的场景，特别是与[联邦](https://kubernetes.io/docs/user-guide/federation/)相结合时。

## 高级示例

另一个示例使用[专用网络拓扑](https://github.com/kubernetes/kops/blob/master/docs/topology.md)为HA创建集群调用：

```bash
kops create cluster \
    --node-count 3 \
    --zones us-west-2a,us-west-2b,us-west-2c \
    --master-zones us-west-2a,us-west-2b,us-west-2c \
    --dns-zone example.com \
    --node-size t2.medium \
    --master-size t2.medium \
    --node-security-groups sg-12345678 \
    --master-security-groups sg-12345678,i-abcd1234 \
    --topology private \
    --networking weave \
    --cloud-labels "Team=Dev,Owner=John Doe" \
    --image 293135079892/k8s-1.4-debian-jessie-amd64-hvm-ebs-2016-11-16 \
    ${NAME}
```

## 注意（最佳实践）

在具有2个可用区域的区域中，将3个master部署在一个zone 中，并且nodes可以分布在2个zone之间。 这可以通过指定标志来完成：

```bash
     --master-count=3
     --master-zones=$MASTER_ZONE
     --zones=$NODE_ZONES
```