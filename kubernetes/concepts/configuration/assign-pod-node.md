---
categories: Kubernetes

tags: 
  - k8s
  - Kubernetes


title: 亲和性和反亲和性

date: 2018-10-25

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

