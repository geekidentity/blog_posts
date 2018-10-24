---
categories: Kubernetes

tags: 
  - k8s
  - kubernetes

title: 使用Kubefed安装集群Federation 

date: 2018-06-24

---

# 使用Kubefed安装集群Federation 

Kubernetes1.5及以上版本包含一个名为[kubefed](https://kubernetes.io/docs/admin/kubefed/)的命令行工具，可帮助你管理Federation 集群。`kubefed` 可帮助你部署Kubernetes集群federation控制台，并将集群添加到现有federation控制台或从现有federation控制台删除集群。

本指南介绍了如何使用kubefed管理Kubernetes集群federation。

注意：Kubernetes 1.6中kubefed是测试功能。

## 先决条件

假设你有一个正在运行的Kubernetes集群。 请参阅平台安装说明的[入门指南](https://kubernetes.io/docs/setup/)。

## 获取kubefed

下载与发行版相对应的客户端tarball，并提取tarball中的二进制文件：

请注意，在kubernetes1.8.x之前，federation 项目作为[kubernetes](https://github.com/kubernetes/kubernetes)仓库的一部分进行维护。在kubernetes发布1.8.0和1.9.0之间的某个时间点时，它成为一个单独项目[federation](https://github.com/kubernetes/federation)，现在在那里维护。 在此之后，federation 的release信息可在[此处](https://github.com/kubernetes/federation/releases)的release页面上找到。

### 对于k8s 1.8.x和更低版本：

```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/${RELEASE-VERSION}/kubernetes-client-linux-amd64.tar.gz
tar -xzvf kubernetes-client-linux-amd64.tar.gz
```

请注意，变量**RELEASE-VERSION**应该被替换为所需的实际版本。

```bash
sudo cp kubernetes/client/bin/kubefed /usr/local/bin
sudo chmod +x /usr/local/bin/kubefed
```

### 对于k8s 1.9.x及以上版本：

```bash
curl -LO https://storage.cloud.google.com/kubernetes-federation-release/release/${RELEASE-VERSION}/federation-client-linux-amd64.tar.gz
tar -xzvf federation-client-linux-amd64.tar.gz
```

请注意，变量**RELEASE-VERSION**应该被替换为[release](https://github.com/kubernetes/federation/releases)页面提供的其中一个发行版本。。

```bash
sudo cp kubernetes/client/bin/kubefed /usr/local/bin
sudo chmod +x /usr/local/bin/kubefed
```

### 安装kubectl

你可以按照[kubectl安装页面](https://kubernetes.io/docs/tasks/tools/install-kubectl/)上的说明安装kubectl的对应版本。

## 选择一个主机集群。

你需要选择一个已有Kubernetes集群作为主机集群。 主机集群上将会运行federation控制台的组件。确保你的本地kubeconfig中有与主机集群相对应的kubeconfig条目。 你可以通过运行下面命令以验证是否具有所需的kubeconfig条目：

```bash
kubectl config get-contexts
```

输出应该包含与你的主机集群相对应的条目，类似于以下内容：

```bash
CURRENT   NAME                                          CLUSTER                                       AUTHINFO                                      NAMESPACE
*         gke_myproject_asia-east1-b_gce-asia-east1     gke_myproject_asia-east1-b_gce-asia-east1     gke_myproject_asia-east1-b_gce-asia-east1
```

在部署federation控制台时，你需要为主机集群提供kubeconfig上下文（在上面的条目中的name值）。

## 部署federation控制台

运行kubefed init命令在主机集群上部署federation控制台。 kubefed init必须提供以下参数：

- Federation name
- `--host-cluster-context`, 主机集群的`kubeconfig` 上下文
- `--dns-provider`,  `'google-clouddns'`, `aws-route53`或`coredns`之一
- `--dns-zone-name`, federation服务的域名后缀

如果你的主机集群运行在非云环境或不支持常见云基本功能的环境（如负载均衡器）中，则可能需要额外的参数。 请参阅下面的[本地主机集群](https://kubernetes.io/docs/tasks/federation/set-up-cluster-federation-kubefed/#on-premises-host-clusters)部分。

以下示例命令将部署名为fellowship，主机集群上下文为rivendell和域名后缀为example.com的federation控制台：

```bash
kubefed init fellowship \
    --host-cluster-context=rivendell \
    --dns-provider="google-clouddns" \
    --dns-zone-name="example.com."
```

--dns-zone-name中指定的域后缀必须你管理的现有域名，并且可由你的DNS提供商提供。它必须以点结束。

一旦federation控制台初始化，就可以查询命名空间：

```bash
kubectl get namespace --context=fellowship
```

如果你没有看到列出的default命名空间（一个[BUG](https://github.com/kubernetes/kubernetes/issues/33292)）。 使用以下命令自行创建它：

```bash
kubectl create namespace default --context=fellowship
```

主机集群中的计算机必须具有相应的权限才能对正在使用的DNS服务进行使用。 例如，如果你的集群正在Google Compute Engine上运行，则必须为你的项目启用Google Cloud DNS API。

Google Kubernetes Engine集群中的机器默认情况下创建时不包含Google Cloud DNS API范围。 如果你要将Google Kubernetes Engine集群用作federation主机，则必须使用带有--scopes字段中适当值的gcloud命令来创建它。 你无法直接修改Google Kubernetes Engine集群以添加此范围，但可以为集群创建新节点池并删除旧集群。 请注意，这将导致集群中的pods 重新计划。

要添加新节点池，请运行：

```bash
scopes="$(gcloud container node-pools describe --cluster=gke-cluster default-pool --format='value[delimiter=","](config.oauthScopes)')"
gcloud container node-pools create new-np \
    --cluster=gke-cluster \
    --scopes="${scopes},https://www.googleapis.com/auth/ndev.clouddns.readwrite"
```

要删除旧节点池，请运行：

```bash
gcloud container node-pools delete default-pool --cluster gke-cluster
```

kubefed init在主机集群中设置联帮控制台，并在本地kubeconfig中为federationAPI服务器添加条目。 请注意，在Kubernetes 1.6的测试版中，kubefed init不会自动将当前上下文设置为新部署的federation。 你可以通过运行手动设置当前上下文：

```bash
kubectl config use-context fellowship
```

fellowship是federation的名称。

### Basic和令牌认证支持

kubefed init默认仅生成TLS证书和密钥，以便与federationAPI服务器进行身份验证并将它们写入本地kubeconfig文件。 如果你希望为调试启用Basic身份验证或令牌身份验证，则可以通过传递--apiserver-enable-basic-auth标志或--apiserver-enable-token-auth标志来启用它们。

```bash
kubefed init fellowship \
    --host-cluster-context=rivendell \
    --dns-provider="google-clouddns" \
    --dns-zone-name="example.com." \
    --apiserver-enable-basic-auth=true \
    --apiserver-enable-token-auth=true
```

### 将命令行参数传递给federation组件

kubefed init使用federationAPI服务器和federation controller manager的缺省参数引导federation控制台。 其中一些参数来自kubefed init的标志。但是，你可以通过通过覆盖标志来覆盖这些命令行参数。

你可以通过将它们传递给--apiserver-arg-overrides来覆盖federationAPI服务器参数，并通过将federation controller manager参数传递给--controllermanager-arg-overrides来覆盖它们。

```bash
kubefed init fellowship \
    --host-cluster-context=rivendell \
    --dns-provider="google-clouddns" \
    --dns-zone-name="example.com." \
    --apiserver-arg-overrides="--anonymous-auth=false,--v=4" \
    --controllermanager-arg-overrides="--controllers=services=false"
```

### 配置DNS提供商

federation服务控制器编程一个DNS提供商，通过DNS名称公开federation服务。 如果主机集群的云提供商与DNS提供商相同，则某些云提供商会自动提供编制DNS提供商所需的配置。 在所有其他情况下，你必须将DNS提供程序配置提供给你的federation控制器管理器，并将其转交给federation服务控制器。 你可以将此配置提供给federation控制器管理器，方法是将其存储在文件中，并将文件的本地文件系统路径传递给kubefed init的--dns-provider-config标志。 例如，将配置保存在$ HOME / coredns-provider.conf中。

```bash
[Global]
etcd-endpoints = http://etcd-cluster.ns:2379
zones = example.com.
```

### 本地主机集群

#### API 服务器服务类型

kubefed init将federationAPI服务器公开为主机集群上的Kubernetes服务。 默认情况下，此服务作为负载均衡服务公开。 大多数内部和裸机环境以及一些云环境缺乏对负载均衡服务的支持。 kubefed init允许在此类环境中将federationAPI服务器公开为NodePort服务。 这可以通过传递--api-server-service-type=NodePort标志来完成。 你还可以通过传递--api-server-advertise-address = <IP-address>标志来指定首选地址以通告federationAPI服务器。 否则，将其中一个主机集群的节点地址选为默认值。

```bash
kubefed init fellowship \
    --host-cluster-context=rivendell \
    --dns-provider="google-clouddns" \
    --dns-zone-name="example.com." \
    --api-server-service-type="NodePort" \
    --api-server-advertise-address="10.0.10.20"
```

#### 为etcd配置存储

federation控制台将其状态存储在 [`etcd`](https://coreos.com/etcd/docs/latest/)中， [`etcd`](https://coreos.com/etcd/docs/latest/) 数据必须存储在永久性存储卷中，以确保federation控制台重新启动的正确操作。在支持[动态配置存储卷](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#dynamic)的主机集群上，kubefed init动态配置 [`PersistentVolume`](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes)并将其绑定到 [`PersistentVolumeClaim`](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)以存储etcd数据。 如果你的主机集群不支持动态配置，你也可以静态配置一个 [`PersistentVolume`](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes)。 kubefed init创建一个具有以下配置的 [`PersistentVolume`](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes)：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    volume.alpha.kubernetes.io/storage-class: "yes"
  labels:
    app: federated-cluster
  name: fellowship-federation-apiserver-etcd-claim
  namespace: federation-system
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

要设置固定的 [`PersistentVolume`](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes)，你必须确保你创建的 [`PersistentVolume`](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes)具有匹配的存储类，访问模式和至少与请求的 [`PersistentVolume`](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes)一样多的容量。

或者，你可以通过将--etcd-persistent-storage=false传递给kubefed init来完全禁用持久性存储。 但是，我们不建议这样做，因为你的federation控制台无法在此模式下重新启动。

```bash
kubefed init fellowship \
    --host-cluster-context=rivendell \
    --dns-provider="google-clouddns" \
    --dns-zone-name="example.com." \
    --etcd-persistent-storage=false
```

kubefed init仍不支持将现有的 [`PersistentVolumeClaim`](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)附加到引导的federation控制台。我们计划在未来版本的kubefed中支持这一点。

#### CoreDNS support支持

federation服务现在支持[CoreDNS](https://coredns.io/)作为DNS提供商之一。 如果你在无法访问基于云的DNS提供程序的环境中运行集群和federation身份验证，则可以运行自己的[CoreDNS](https://coredns.io/)实例并将federation服务DNS名称发布到该服务器。

你可以配置federation使用[CoreDNS](https://coredns.io/)，方法是将值传递给kubefed init的--dns-provider和--dns-provider-config标志

```bash
kubefed init fellowship \
    --host-cluster-context=rivendell \
    --dns-provider="coredns" \
    --dns-zone-name="example.com." \
    --dns-provider-config="$HOME/coredns-provider.conf"
```

更多信息请查看： [Setting up CoreDNS as DNS provider for Cluster Federation](https://kubernetes.io/docs/tasks/federation/set-up-coredns-provider-federation/)

## 将集群添加到federation

在部署federation控制面板之后，你需要使控制台知道它应该管理的集群。

要将集群加入federation：

1. 更改上下文：

```bash
kubectl config use-context fellowship
```

2. 如果你正在使用托管集群服务，请允许该服务访问集群。 为此，请为与集群服务关联的帐户创建一个集群绑定：

```bash
kubectl create clusterrolebinding <your_user>-cluster-admin-binding --clusterrole=cluster-admin --user=<your_user>@example.org --context=<joining_cluster_context
```

3. 使用kubefed连接将集群加入federation，并确保提供以下内容：

   1. 要加入federation的集群的名称
   2. `--host-cluster-context`，主机集群的kubeconfig上下文

   例如，此命令将集群`gondor` 全部添加到运行在主机集群rivendell上的federation：

```bash
 kubefed join gondor --host-cluster-context=rivendell
```

现在已将新的上下文添加到名为fellowship的kubeconfig中（在你的federation名称之后）。

注意：你提供给join命令的名称将用作federation中的加入集群的标识。 如果此名称符合[标识文档](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/)中描述的规则。 如果与加入集群相对应的上下文符合这些规则，则可以在join命令中使用相同的名称。 否则，你必须为集群选择一个不同的名称。

### 命名规则和自定义

为 `kubefed join`提供的集群名称必须是有效的[RFC 1035](https://www.ietf.org/rfc/rfc1035.txt)标签，并在[Identifiers文档](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/)中。

此外，federation控制台需要federation集群的凭证才能对其进行操作。 这些凭据是从本地kubeconfig获取的。 kubefed join使用指定的集群名称作为参数，以在本地kubeconfig中查找集群的上下文。 如果无法找到匹配的上下文，则会退出并显示错误。

如果federation中的每个集群的上下文名称不遵循[RFC 1035](https://www.ietf.org/rfc/rfc1035.txt)标签命名规则，则这可能会导致一些问题。 在这种情况下，你可以指定符合[RFC 1035](https://www.ietf.org/rfc/rfc1035.txt)标签命名规则的集群名称，并使用--cluster-context标志指定集群上下文。 例如，如果要加入的集群的上下文是gondor_needs-no_king，那么你可以通过运行以下命令来加入集群：

```bash
kubefed join gondor --host-cluster-context=rivendell --cluster-context=gondor_needs-no_king
```

#### Secret name

如上所述的federation控制面板所需的集群凭证作为密钥存储在主机集群中。 密钥的名称也来自集群名称。

但是，Kubernetes中的秘密对象的名称应符合RFC 1123中描述的DNS子域名规范。如果不是这种情况，则可以使用--secret-name标志将秘密名称传递给kubefed连接。 例如，如果群集名称是noldor，秘密名称是11kingdom，则可以通过运行以下命令来加入群集：

```bash
kubefed join noldor --host-cluster-context=rivendell --secret-name=11kingdom
```

注意：如果你的集群名称不符合DNS子域名称规范，则只需通过--secret-name标志提供密码名称即可。 kubefed加入自动为你创建秘密。

### `kube-dns` 配置

必须在每个加入群集中更新kube-dns配置以启用federation服务发现。 如果加入的Kubernetes集群版本为1.5或更高版本，而kubefed版本为1.6或更高版本，那么当使用kubefed join或unjoin命令加入或取消加入群集时，会自动管理此配置。

在所有其他情况下，你必须手动更新kube-dns配置，如管理指南的更新KubeDNS部分中所述。

## 从federation中移除集群

从federation中移除集群，请使用集群名称和federation的--host-cluster-context运行kubefed unjoin命令：

```bash
kubefed unjoin gondor --host-cluster-context=rivendell
```

## 关闭federation控制台

在这个kubefed的beta版本中，没有完全实施federation控制台的正确清理。 但是，暂时删除federation系统名称空间应该除去为federation控制面板的etcd动态设置的永久存储卷之外的所有资源。 你可以通过运行以下命令来删除federation身份验证名称空间：

```bash
kubectl delete ns federation-system --context=rivendell
```

请注意，rivendell是主机集群名称，请将其替换为配置中的相应名称。

