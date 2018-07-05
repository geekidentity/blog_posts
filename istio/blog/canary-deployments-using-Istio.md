[Istio](https://istio.io/)项目的一个好处是它提供了金丝雀部署所需的一切。金丝雀部署的想法是让一小部分用户流量测试新版本的服务，然后如果一切顺利，可以逐渐增加新版本百分比，同时逐步淘汰旧版本。 如果在此过程中出现任何问题，将中止升级并回滚到之前的版本。在最简单的形式中，发送到金丝雀版本的流量是随机选择的请求百分比，但在更复杂的方案中，它可以基于请求的region，用户或其他属性来决定用哪些用户流量测试新版本。

我们为什么需要Istio对金丝雀部署的支持，像Kubernetes这样的平台已经提供了进行[滚动升级](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)和[金丝雀部署](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#canary-deployments)的方法。 但问题好像并没有完全解决。虽然Kubernetes可以进行简单的部署，但它非常有限，特别是在大规模云环境中接收大量（特别是不同数量的）流量，需要进行自动弹性伸缩。

## 在Kubernetes中金丝雀部署

举个例子，假设我们有一个已部署的服务helloworld v1版本，我们想要测试或发布新版本v2。 使用Kubernetes，可以通过简单地更新service相应[Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) 中的image 并[自动进行部署](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)来发布新版本的helloworld服务。 如果我们在仅启动一个或两个v2副本之后[暂停](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#pausing-and-resuming-a-deployment)发布同时有足够的v1副本运行，我们可以保持金丝雀部署对系统的影响非常小。 然后我们可以在继续观察我们的v2版本应用，以决定是继续升级或者在必要时进行[回滚](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment)。 最重要的是，我们甚至可以将一个[水平pod autoscaler](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment) 附加到Deployment ，如果在部署过程中还需要向上或向下扩展副本以处理流量负载，使用它将保持副本比率一致。

尽管它的功能很好，但这种方法适用于已经测试过的新版本，也就是说，这样更多的是蓝绿部署。 实际上，对于没用充分测试的金丝雀版本，Kubernetes中的金丝雀部署将使用具有[相同pod标签](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#using-labels-effectively)的两个Deployments来完成。 在这种情况下，我们不能再使用autoscalers，因为它现在由两个独立的autoscalers完成，每个Deployment，复本比率（百分比）可能与所需的比率不同，完全取决于负载。

无论我们使用一个deployment 还是两个deployment ，使用Docker，Mesos / Marathon或Kubernetes等容器编排平台的部署功能进行的canary管理都存在一个基本问题：使用实例扩展来管理流量; 流量版本分发和副本部署在这些系统中不是独立的。 所有副本容器，无论版本如何，在kube-proxy线程池中都被视为相同的，因此管理特定版本接收的流量的唯一方法是控制副本比率。 以小百分比维持金丝雀流量需要许多复制品（例如，1％将需要至少100个复本）。 即使我们忽略了这个问题，部署方法仍然非常有限，因为它只支持简单（百分比）金丝雀部署方法。 相反，如果我们想根据某些特定标准限制金丝雀对请求的可见性，我们仍然需要另一种解决方案。

## 进入 Istio

使用Istio，流量路由和副本部署是两个完全独立的功能。 实现服务的pod的数量可以根据流量负载自由扩展和缩小，与版本流量路由的控制完全正交。 这使得在存在自动缩放的情况下管理金丝雀版本是一个更简单的问题。 事实上，autoscalers 可以响应由流量路由变化引起的负载变化，但它们仍然独立运行，与负载因其他原因而发生变化时没有区别。

Istio的路由规则也提供了其他重要的功能; 你可以轻松控制细粒度流量百分比（例如，路由1％的流量而无需100个容器），你可以使用其他条件控制流量（例如，将特定用户的流量路由到canary版本）。 为了说明，让我们看一下部署helloworld服务。

我们首先定义helloworld服务，和其他Kubernetes服务一样，如下所示：

```yaml
apiVersion: v1
kind: Service
metadata:
name: helloworld
labels:
  app: helloworld
spec:
  selector:
    app: helloworld
  ...

```

然后我们添加2个Deployments，每个版本一个（v1和v2），这两个版本都包含service selector’s 的app: helloworld标签：

```yaml
kind: Deployment
metadata:
  name: helloworld-v1
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: helloworld
        version: v1
    spec:
      containers:
      - image: helloworld-v1
        ...
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: helloworld-v2
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: helloworld
        version: v2
    spec:
      containers:
      - image: helloworld-v2
        ...

```

请注意，这与使用普通Kubernetes进行[canary部署](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#canary-deployments)的方式完全相同，但在这种情况下，我们需要调整每个Deployment的副本数以控制流量分配。 例如，要将10％的流量发送到canary版本（v2），v1和v2的副本可以分别设置为9和1。

但是，由于我们要在[启用Istio](https://istio.io/docs/setup/)的群集中部署服务，我们需要做的就是设置路由规则来控制流量分配。 例如，如果我们想将10％的流量发送到金丝雀，我们可以使用[istioctl](https://istio.io/docs/reference/commands/istioctl/)命令设置如下的路由规则：

```yaml
cat <<EOF | istioctl create -f -
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: helloworld
spec:
  hosts:
    - helloworld
  http:
  - route:
    - destination:
        host: helloworld
        subset: v1
      weight: 90
    - destination:
        host: helloworld
        subset: v2
      weight: 10
EOF

```

设置此规则后，Istio将确保只有十分之一的请求将发送到canary版本，无论每个版本的运行副本数量是多少。

## 自动伸缩deployments

因为我们不再需要维护副本比例，所以我们可以安全地添加horizontal pod autoscalers来管理两个版本部署的副本：