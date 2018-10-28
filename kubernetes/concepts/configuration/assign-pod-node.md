---
categories: Kubernetes

tags: 
  - k8s
  - Kubernetes


title: 亲和性和反亲和性

date: 2018-10-28

---

亲和性和反亲和性

nodeSelector 提供了一种非常简单的方法，将 pod 分配到有指定 label 的 node 结点上。亲和性/反亲和性可以极大地扩展调度策略。
关键改进主要有：

1. 更丰富的语言（不在是精确匹配）。
2. 你可以指明规则是 soft/preference，而不是必须满足规则，这样如果调度规则无法满足，pod 依然可以正常分配到 node 上。
3. ​

亲和性策略有两种类型的亲和性：node 亲和、pod之间的亲和与反亲和。node 亲和有点像 nodeSelector （当然有上面列举的前两个改进）。pod 之间的亲和与反亲和性将会限制 pod 的调度。

nodeSelector 现在可以正常使用，但最终会被弃用，因为 node 亲和性可以代替 nodeSelector 所有功能。

## Node 亲和性

node 亲和性在 Kubernetes 1.2 中作为 alpha 引入。node 亲和性在概念上类似于 nodeSelector，可以根据标签约束 pod 调度到有指定标签的 node 上。

目前有两种类型的 node 亲和：requiredDuringSchedulingIgnoredDuringExecution 和 preferredDuringSchedulingIgnoredDuringExecution ，前者是将 pod 调度 node 上必须满足的规则，后者指调度程序将尝试执行但不一定满足规则。IgnoredDuringExecution 与 nodeSelector 工作方式类似，如果 node 上的标签在运行时发生了变化，此时即使标签不再满足 pod 上的亲和性规则，pod 仍将会在节点上继续运行。未来计划提供 requiredDuringSchedulingRequiredDuringExecution，这将像 requiredDuringSchedulingIgnoredDuringExecution 一样，除了它将从不再满足pod的node 亲和性要求的 node 中驱逐 pod。

因此下面 requiredDuringSchedulingIgnoredDuringExecution 的示例将“仅在具有Intel CPU的节点上运行pod”，并且示例 preferredDuringSchedulingIgnoredDuringExecution 将“尝试在可用区XYZ中运行这组 pod，但如果不可能，则允许一些在其他地方运行”。

node 亲和性在PodSpec中指定为 affinity.nodeAffinity。

以下是使用 node 亲和性的 pod 的示例：

```yaml
pods/pod-with-node-affinity.yaml  
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name
            operator: In
            values:
            - e2e-az1
            - e2e-az2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: k8s.gcr.io/pause:2.0

```

此 node 亲和性规则表示，该 pod 只能放置在标签的键为 kubernetes.io/e2e-az-name 其值为 e2e-az1 或 e2e-az2 的 node 上。 此外，在满足该条件的 node 中，应优先选择具有其键为 another-node-label-key 且其值为 another-node-label-value 的标签的 node。

在示例中可以看到运算符 In。 新的 node 亲和性语法支持以下运算符：In，NotIn，Exists，DoesNotExist，Gt，Lt。您可以使用 NotIn 和 DoesNotExist 来实现 node 反亲和性，或使用 node taints 来排除特定 node。

如果同时指定 nodeSelector 和 nodeAffinity，则必须满足两者才能将 pod 调度到候选节点上。

如果指定 nodeAffinity 的多个 nodeSelectorTerms，则可以只满足其中一个 nodeSelectorTerms 的情况下将pod调度到节点上。

如果指定 nodeSelectorTerms 的多个 matchExpressions，则必须在满足所有matchExpressions的情况下才能将pod调度到节点上。

如果删除或更改已调度容器的 node 的标签，则不会删除该容器。 换句话说，亲和性选择仅在调度 pod 时起作用。

preferredDuringSchedulingIgnoredDuringExecution 中的 weight 字段取值在 1-100 内。 对于满足所有调度要求的每个 node（资源请求，RequiredDuringScheduling 亲和性表达式等），调度程序将通过迭代此字段的元素计算总和，并在节点与对应的节点匹配时将“weight”添加到总和MatchExpressions。 然后将该分数与节点的其他优先级函数的分数组合。 总得分最高的节点是最优选的。

## Pod 间的亲和性和反亲和性

在 Kubernetes 1.4 中引入了 pod 间亲和性和反亲和性。通过 Pod 间亲和性和反亲和性，可以根据已在 node 上运行的 pod 的标签而不是基于 node 上的标签来约束 pod 可以调度到哪些节点。如果 X 已经在运行一个或多个符合规则 Y 的 pod，则规则的形式为“此 pod 应该（或者，如果是反亲和性，则不应该）在X中运行”。 Y表示为 LabelSelector ，带有关联的命名空间列表；与 node 不同，因为 pod 是在一个 namespace 中（因此pod上的标签是隐式包含命名空间），pod标签上的标签选择器必须指定选择器应该应用于哪些命名空间。从概念上讲，X是一个拓扑域，如节点，机架，云提供商区域，云提供商区域等。您使用 topologyKey 表达它，拓扑键是系统用于表示此类拓扑域的节点标签的关键。

**注意**：pod 间的亲和性和反亲和性需要大量的处理，这会显着减慢大型集群中的调度。 我们不建议在大于几百个节点的群集中使用它们。

**注意**：Pod 反亲和性要求 node 一致地标记，即集群中的每个节点必须具有匹配的匹配topologyKey的标签。 如果某些或所有节点缺少指定的topologyKey标签，则可能导致意外行为。

与节点关联性一样，目前有两种类型的 pod 亲和性和反亲和性，称为 requiredDuringSchedulingIgnoredDuringExecution 和 preferredDuringSchedulingIgnoredDuringExecution，表示“硬”与“软”要求。 请参阅前面 node 亲和性部分中的说明。 requiredDuringSchedulingIgnoredDuringExecution亲和力的一个例子是“将服务A和服务B的pod放在同一区域中，因为它们彼此之间进行了很多通信”，并且示例preferredDuringSchedulingIgnoredDuringExecution反关联性将“从此服务传播pod” 区域“（一个硬性要求没有意义，因为你可能有比区域更多的 pod）。

在PodSpec中，将pod间亲和性关系指定为字段亲和性关系的字段podAffinity。 并且pod-pod反亲和性被指定为PodSpec中字段反亲和力的字段podAntiAffinity。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: failure-domain.beta.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: security
              operator: In
              values:
              - S2
          topologyKey: kubernetes.io/hostname
  containers:
  - name: with-pod-affinity
    image: k8s.gcr.io/pause:2.0
```

上面的 pod 亲和性关系定义了一个 pod 亲和性规则和一个 pod 反亲和性规则。在此示例中，podAffinity是requiredDchedSchedulingIgnoredDuringExecution，而podAntiAffinity是preferredDuringSchedulingIgnoredDuringExecution。 pod 亲和性规则表示只有当该节点与至少一个已经运行的pod具有相同 zone 时，才能将该pod安排到节点上，该pod具有带有“security”和“S1”键的标签。 （更确切地说，如果节点N具有密钥failure-domain.beta.kubernetes.io/zone的标签，并且某些值V使得集群中至少有一个节点出现密钥故障，则该pod有资格在节点N上运行-domain.beta.kubernetes.io/zone和运行具有密钥“security”和值“S1”的标签的pod的值V.）pod反亲和性规则表明pod不安排到该 node，如果该节点已在运行具有键“security”和值“S2”的标签的pod。 （如果topologyKey是failure-domain.beta.kubernetes.io/zone那么这意味着如果该节点与具有密钥“security”和值的标签的pod位于同一区域中，则无法将该pod安排到节点上S2“。）有关pod亲和力和反亲和力的更多示例，请参阅设计文档，其中包括requiredDuringSchedulingIgnoredDuringExecution flavor和preferredDuringSchedulingIgnoredDuringExecution flavor。

pod亲和性和反亲和性的合法操作是In，NotIn，Exists，DoesNotExist。

原则上，topologyKey 可以是任何合法的标签密钥。 但是，出于性能和安全性原因，topologyKey 存在一些限制：

1. 对于亲和性和requiredDuringSchedulingIgnoredDuringExecution pod反亲和性，不允许使用的topologyKey。
2. 对于requiredDuringSchedulingIgnoredDuringExecution pod反亲和性，引入了许可控制器LimitPodHardAntiAffinityTopology以将topologyKey限制为kubernetes.io/hostname。 如果要使其可用于自定义拓扑，则可以修改许可控制器，或者只是禁用它。
3. 对于preferredDuringSchedulingIgnoredDuringExecution pod反关联，空topologyKey被解释为“所有拓扑”（此处“所有拓扑”现在仅限于kubernetes.io/hostname，failure-domain.beta.kubernetes.io/zone和failure-domain的组合.beta.kubernetes.io/区域）。
4. 除上述情况外，topologyKey 可以是任何合法的标签密钥。