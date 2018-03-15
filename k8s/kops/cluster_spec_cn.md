---
categories: k8s

tags: 
  - k8s
  - kubernetes
  - kops

title: kops中config和cluster.spec中keys 的描述

date: 2018-03-15

---

# kops中config和cluster.spec中keys 的描述

这里的描述并不完整，但旨在记录那些不太容易解释的key。 我们的[godoc](https://godoc.org/k8s.io/kops/pkg/apis/kops)参考提供了更详细的API值列表。 在YAML文件中有描述集群的两个顶级API值：[ClusterSpec](https://godoc.org/k8s.io/kops/pkg/apis/kops#ClusterSpec)定义为kind:YAML，[InstanceGroup](https://godoc.org/k8s.io/kops/pkg/apis/kops#InstanceGroup)定义为kind：YAML。

## spec

### api

该对象配置我们如何公开API：

- `dns` 将允许直接访问master实例，并将DNS配置为直接指向master节点。


- `loadBalancer` 将在主节点前配置负载平衡器（ELB），并将DNS配置为指向ELB。

DNS example:

```yaml
spec:
  api:
    dns: {}
```

当配置LoadBalancer时，可以选择使用公共ELB或内网（仅限VPC）ELB。 `type` 字段应该是`Public` 或 `Internal`。

另外，可以通过设置`additionalSecurityGroups`将预先创建的其他安全组添加到负载均衡器。

```yaml
spec:
  api:
    loadBalancer:
      type: Public
      additionalSecurityGroups:
      - sg-xxxxxxxx
      - sg-xxxxxxxx
```

此外，可以通过设置`idleTimeoutSeconds`来增加负载平衡器的空闲超时(idle timeout)。 默认的空闲超时时间为5分钟，AWS最多允许3600秒（60分钟）。 有关更多信息，请参阅[配置空闲超时](http://docs.aws.amazon.com/elasticloadbalancing/latest/classic/config-idle-timeout.html)。

```yaml
spec:
  api:
    loadBalancer:
      type: Public
      idleTimeoutSeconds: 300
```

### etcdClusters v3 & tls

虽然kops目前默认不是etcd3，但可以打开v3和TLS身份验证以便在群集成员之间进行通信。 这些选项可以通过集群配置文件启用（只在manifests中，没有命令行选项）。 警告：目前没有从v2迁移到v3的升级途径，因此**不要**尝试在v2运行群集上启用此功能，因为它必须在创建群集时完成。 下面的示例代码片段假设一个由三个主服务器组成的HA群集。

```yaml
etcdClusters:
- etcdMembers:
  - instanceGroup: master0-az0
    name: a-1
  - instanceGroup: master1-az0
    name: a-2
  - instanceGroup: master0-az1
    name: b-1
  enableEtcdTLS: true
  name: main
  version: 3.0.17
- etcdMembers:
  - instanceGroup: master0-az0
    name: a-1
  - instanceGroup: master1-az0
    name: a-2
  - instanceGroup: master0-az1
    name: b-1
  enableEtcdTLS: true
  name: events
  version: 3.0.17
```

### kubernetesApiAccess

该数组配置能够访问kubernetes API的CIDR。 在AWS上，这表现为ELB或主安全组上的入站安全组规则。

例如，使用此键可将群集访问限制为办公室IP地址范围。

```yaml
spec:
  kubernetesApiAccess:
    - 12.34.56.78/32
```

### cluster.spec Subnet Keys

#### id

要在现有VPC中共享的子网ID。

#### egress

现有VPC中您想用作处网的“出口”的资源标识符（ID）。

此功能最初设想为允许重新使用NAT网关。用法如下。 虽然NAT网关是面向“公共”资源的，但在集群规范中，必须在专用私有子网部分中指定它们。 考虑这一点的一种方法是你指定“egress”，这是从这个私有子网的默认路由。

```yaml
spec:
  subnets:
  - cidr: 10.20.64.0/21
    name: us-east-1a
    egress: nat-987654321
    type: Private
    zone: us-east-1a
  - cidr: 10.20.32.0/21
    name: utility-us-east-1a
    id: subnet-12345
    type: Utility
    zone: us-east-1a
```

#### publicIP

您想要添加到NAT网关的现有EIP的IP。

```yaml
spec:
  subnets:
  - cidr: 10.20.64.0/21
    name: us-east-1a
    publicIP: 203.93.148.142
    type: Private
    zone: us-east-1a
```

### kubeAPIServer

该部分包含`kube-apiserver`的配置。

#### 用于Open ID Connect令牌的oidc标志

Read more about this here: <https://kubernetes.io/docs/admin/authentication/#openid-connect-tokens>

```yaml
spec:
  kubeAPIServer:
    oidcIssuerURL: https://your-oidc-provider.svc.cluster.local
    oidcClientID: kubernetes
    oidcUsernameClaim: sub
    oidcUsernamePrefix: "oidc:"
    oidcGroupsClaim: user_roles
    oidcGroupsPrefix: "oidc:"
    oidcCAFile: /etc/kubernetes/ssl/kc-ca.pem
```

#### audit logging

Read more about this here: <https://kubernetes.io/docs/admin/audit>

```yaml
spec:
  kubeAPIServer:
    auditLogPath: /var/log/kube-apiserver-audit.log
    auditLogMaxAge: 10
    auditLogMaxBackups: 1
    auditLogMaxSize: 100
    auditPolicyFile: /srv/kubernetes/audit.yaml
```

**注意**：auditPolicyFile是必需的。 如果该标志被省略，则不记录事件。

可以使用[fileAssets](https://github.com/kubernetes/kops/blob/master/docs/cluster_spec.md#fileassets)功能在master节点上推送高级审核策略文件。

示例策略文件可以在[这里](https://raw.githubusercontent.com/kubernetes/website/master/docs/tasks/debug-application-cluster/audit-policy.yaml)找到

#### Max Requests Inflight

The maximum number of non-mutating requests in flight at a given time. 当服务器超过这个时，它拒绝请求。 零为无限制。 （默认400）

```yaml
spec:
  kubeAPIServer:
    maxRequestsInflight: 1000
```

#### runtimeConfig

这里的键和值被翻译成kube-apiserver中`--runtime-config`的值，用逗号分隔。

使用它来启用alpha功能，例如：

```yaml
spec:
  kubeAPIServer:
    runtimeConfig:
      batch/v2alpha1: "true"
      apps/v1alpha1: "true"
```

将产生参数 `--runtime-config=batch/v2alpha1=true,apps/v1alpha1=true` 。 请注意，kube-apiserver接受true作为开关状标志的值。

#### serviceNodePortRange

该值作为kube-apiserver的`--service-node-port-range` 参数。

```yaml
spec:
  kubeAPIServer:
    serviceNodePortRange: 30000-33000
```

### externalDns

这部分配置选项是为外部DNS提供的。当前的外部DNS提供商是kops dns-controller，它可以为Kubernetes资源设置DNS记录。 dns-controller计划逐步淘汰并替换为external-dns。

```yaml
spec:
  externalDns:
    watchIngress: true
```

默认的kops行为是false。watchIngress:true使用默认的dns-controller行为来监视入口控制器的变化。 在某些情况下，设置此选项有中断服务更新的风险。

### kubelet

该部分包含kubelet的配置。 请参阅https://kubernetes.io/docs/admin/kubelet/

注意：如果相应的配置值可以为空，则可以在spec中将字段设置为空，这样可以将空字符串作为配置值传递给kubelet。

```yaml
spec:
  kubelet:
    resolvConf: ""
```

将会建立标志--resolv-conf= 。

#### 启用自定义metrics支持

要按照[自定义metrics doc 我们在kubernetes 中使用自定义指标，我们必须在所有kubelets上将标志--enable-custom-metrics设置为true。 可以在我们的cluster.yml中的kubelet规范中指定它。

```yaml
spec:
  kubelet:
    enableCustomMetrics: true
```

### kubeScheduler

该部分包含kube-scheduler的配置。 请参阅https://kubernetes.io/docs/admin/kube-scheduler/

```yaml
spec:
  kubeScheduler:
    usePolicyConfigMap: true
```

将使kube-scheduler使用命名空间kube-system中configmap的“scheduler-policy”调度程序策略。

请注意，从Kubernetes 1.8.0 kube-scheduler不会自动从configmap重新加载其配置。 需要进入master 实例并手动重启Docker容器。

### kubeControllerManager

该部分包含controller-manager的配置。

```yaml
spec:
  kubeControllerManager:
    horizontalPodAutoscalerSyncPeriod: 15s
    horizontalPodAutoscalerDownscaleDelay: 5m0s
    horizontalPodAutoscalerUpscaleDelay: 3m0s
```

有关horizontalPodAutoscaler标志的更多详细信息，请参阅[官方HPA文档](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)和[Kops指南如何设置它](https://github.com/kubernetes/kops/blob/master/docs/horizontal_pod_autoscaling.md)。

#### Feature Gates

```yaml
spec:
  kubelet:
    featureGates:
      Accelerators: "true"
      AllowExtTrafficLocalEndpoints: "false"
```

将会生成flag --feature-gates=Accelerators=true,AllowExtTrafficLocalEndpoints=false

注：Feature gate `ExperimentalCriticalPodAnnotation`默认启用，因为一些关键组件如`kube-proxy` 依赖它。

#### 计算资源保留

```yaml
spec:
  kubelet:
    kubeReserved:
        cpu: "100m"
        memory: "100Mi"
        storage: "1Gi"
    kubeReservedCgroup: "/kube-reserved"
    systemReserved:
        cpu: "100m"
        memory: "100Mi"
        storage: "1Gi"
    systemReservedCgroup: "/system-reserved"
    enforceNodeAllocatable: "pods,system-reserved,kube-reserved"
```

将会生成标志： `--kube-reserved=cpu=100m,memory=100Mi,storage=1Gi --kube-reserved-cgroup=/kube-reserved --system-reserved=cpu=100mi,memory=100Mi,storage=1Gi --system-reserved-cgroup=/system-reserved --enforce-node-allocatable=pods,system-reserved,kube-reserved`。

了解有关[预留计算资源](https://kubernetes.io/docs/tasks/administer-cluster/reserve-compute-resources/)的更多信息。

### networkID

在AWS上，这是创建群集的VPC的ID。如果需要从头开始创建群集，可以不在创建时指定此字段; kops将为你创建一个VPC。

```yaml
spec:
  networkID: vpc-abcdefg1
```

有关在现有VPC中运行的更多信息，请参阅[此处](https://github.com/kubernetes/kops/blob/master/docs/run_in_existing_vpc.md)。

### hooks

钩子(hooks)允许在群集中的每个节点安装Kubernetes之前执行一些操作。 例如，可以安装Nvidia驱动程序以使用GPU。 这个钩子可以是Docker镜像或manifest（systemd单元）的形式。 hooks可以放在集群spec中，这意味着它们将会全局部署，或者可以将它们放入instanceGroup spec中。 注意：当instanceGroup与集群spec上的服务名相同时，instanceGroup优先并忽略群集spec中的定义；即，如果群集中具有单元文件“myunit.service”，并且instanceGroup中有单元文件“myunit.service”，则应用instanceGroup中的文件。

```yaml
spec:
  # many sections removed
  hooks:
  - before:
    - some_service.service
    requires:
    - docker.service
      execContainer:
      image: kopeio/nvidia-bootstrap:1.6
      # these are added as -e to the docker environment
      environment:
        AWS_REGION: eu-west-1
        SOME_VAR: SOME_VALUE

  # or a raw systemd unit
  hooks:
  - name: iptable-restore.service
    roles:
    - Node
    - Master
    before:
    - kubelet.service
    manifest: |
      [Service]
      EnvironmentFile=/etc/environment
      # do some stuff

  # or disable a systemd unit
  hooks:
  - name: update-engine.service
    disabled: true

  # or you could wrap this into a full unit
  hooks:
  - name: disable-update-engine.service
    before:
    - update-engine.service
    manifest: |
      Type=oneshot
      ExecStart=/usr/bin/systemctl stop update-engine.service
```

安装Ceph

```yaml
spec:
  # many sections removed
  hooks:
  - execContainer:
      command:
      - sh
      - -c
      - chroot /rootfs apt-get update && chroot /rootfs apt-get install -y ceph-common
      image: busybox
```

### fileAssets

FileAssets是一个alpha功能，允许你将内联文件内容放入群集和instanceGroup 配置中。 它被设计为alpha，你可以通过kubernetes daemonsets做替代。

```yaml
spec:
  fileAssets:
  - name: iptable-restore
    # Note if not path is specificied the default path it /srv/kubernetes/assets/<name>
    path: /var/lib/iptables/rules-save
    roles: [Master,Node,Bastion] # a list of roles to apply the asset to, zero defaults to all
    content: |
      some file content
```

### cloudConfig

#### disableSecurityGroupIngress

如果您将aws用作cloudProvider，则可以禁用ELB安全组对Kubernetes Nodes安全组的授权。 换句话说，它不会添加安全组规则。 这可以是有用的，可以避免AWS限制：每个安全组50个规则。

```yaml
spec:
  cloudConfig:
    disableSecurityGroupIngress: true
```

#### elbSecurityGroup

警告：这仅适用于1.7.0以上的Kubernetes版本。

为避免为每个elb创建一个安全组，可以指定安全组id，该id将分配给LoadBalancer。 它必须是安全组ID，而不是名称。 `api.loadBalancer.additionalSecurityGroups` 必须为空，因为Kubernetes将为服务文件中指定的每个端口添加规则。 这可以避免AWS限制：每个区域500个安全组，每个安全组50个规则。

```yaml
spec:
  cloudConfig:
    elbSecurityGroup: sg-123445678
```

### docker

可以覆盖集群中所有masters 和nodes 的Docker守护进程选项。查看[API文档](https://godoc.org/k8s.io/kops/pkg/apis/kops#DockerConfig)以获取完整的选项列表。

#### registryMirrors

如果你有一堆Docker实例（physicsal或vm）正在运行，每当他们中的一个pull主机上不存在的image 时，它将从DockerHub上拉取它。通过缓存这些images，可以将流量保留在本地网络中，避免出口带宽使用。此设置不仅有利于集群配置，而且还有利于images 拉取。

@see [Cache-Mirror Dockerhub For Speed](https://hackernoon.com/mirror-cache-dockerhub-locally-for-speed-f4eebd21a5ca) @see [Configure the Docker daemon](https://docs.docker.com/registry/recipes/mirror/#configure-the-docker-daemon).

```yaml
spec:
  docker:
    registryMirrors:
    - https://registry.example.com
```

#### storage

可以指定Docker[存储驱动程序](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-storage-driver)以覆盖默认值。 确保你选择的驱动程序受操作系统和docker版本的支持。

```yaml
docker:
  storage: devicemapper
  storageOpts:
    - "dm.thinpooldev=/dev/mapper/thin-pool"
    - "dm.use_deferred_deletion=true"
    - "dm.use_deferred_removal=true"
```

### sshKeyName

在某些情况下，可能需要使用现有的AWS SSH密钥，而不是允许kops创建新密钥。 AWS中提供密钥的名称是--ssh-public-key的替代方案。

```yaml
spec:
  sshKeyName: myexistingkey
```

### target

在某些使用情况下，你可能希望使用额外选项来增加目标输出。目标支持最少的选项，你可以做到这一点。 目前只有terraform的目标支持这一点，但如果其他用例出现，kops最终可能会支持更多。

```yaml
spec:
  target:
    terraform:
      providerExtraConfig:
        alias: foo
```