# Kubernetes 引导启动

这是使用kops时，关于Kubernetes集群是如何建立启动的概述。

## 从spec 到完整配置

kops工具本身采用用户指定的集群的（最小）规范，并计算完整配置，设置未指定值的默认值，并导出适当的依赖关系。 “完整”规范包含将传递给所有组件的所有标志集。 所有关于如何安装集群的决定都是在这个阶段做出的，因此如果用户在规范中指定了一个值，理论上可以改变每一个决定。

此完整规范在AutoScaling组（在AWS上）或Managed Instance Group（在GCE上）的启动配置中设置。

在AWS和GCE上，一切（nodes & masters）都在ASG / MIG中运行; 这意味着失败（或用户）可以终止机器并且kops系统将会进行自我修复。

## nodeup：从image到kubelet

nodeup是为Kubelet安装软件包并设置操作系统的组件。 核心要求是：

- 必须安装Docker。 nodeup将安装Docker 1.13.1，这是Docker与Kubernetes 1.8一起测试的版本。
- Kubelet，并且安装了systemd服务。

另外，nodeup还会安装：

- Protokube，这是一个kops专用组件

## /etc/kubernetes/manifests

由/etc/kubernetes/manifest 中的文件控制kubelet启动pods。这些文件由nodeup和protokube（理想情况下全部由protokube创建，但目前是由两者分开的）创建。

这些pod使用标准k8s manifests声明，就像它们存储在API中一样。 但是它们被用来打破我们核心组件的循环依赖，比如etcd和kube-apiserver。

在master上：

- kube-apiserver
- kube-controller-manager (运行各种控制器)
- kube-scheduler (它将pod分配给node)
- etcd (这实际上是由protokube创建的)
- dns-controller

在node上：

- 配置iptables使k8s网络工作

## kubelet start

Kubelet启动/重启）/etc/kubernetes/manifest 中的所有容器。

Kubelet还尝试联系API server（master kubelet本身最终会启动），注册该节点。一旦节点注册完成，kube-controller-manager将为其分配一个PodCIDR，它是分配范围在k8s网络IP内。 kube-controller-manager更新节点，设置PodCIDR字段。 一旦kubelet看到这个分配，它将用这个CIDR建立本地网桥，允许docker启动。在这之前，只有具有hostNetwork的pod才能工作 - 因此所有“核心”容器都以hostNetwork=true运行。

## api-server启动

api-server将在主服务器上监听localhost:8080。 这是一个不安全的端点，但只能从主服务器访问，并且仅适用于使用hostNetwork = true运行的Pod。 这就是像kube-scheduler和kube-controller-manager这样的组件可以在不需要令牌的情况下到达API。