---
categories: Istio
tags:
  - Istio

title: Istio 流量管理
date: 2018-06-27
---

# 综述

本页面概述了Istio中流量管理的工作原理，包括流量管理原则的优点。我们假定你已经阅读了[什么是Istio](https://istio.io/docs/concepts/what-is-istio/overview/)？ 并熟悉Istio的高层架构。 您可以在本节的其他指南中找到有关流量管理功能的更多信息。

## Pilot 和Envoy

Istio中流量管理的核心组件是[Pilot](https://istio.io/docs/concepts/traffic-management/pilot/)，它管理和配置部署在Istio服务网格中的所有Envoy代理实例。它允许你指定要使用哪些规则在Envoy代理之间路由流量，并配置故障恢复功能，例如超时，重试和断路器。 它还维护网格中所有服务的规范模型，并使用它来让Envoys 通过其服务发现了解网格中的其他实例。

每个Envoy实例根据从Pilot获取的信息和其负载平衡池中其他实例的定期运行状况检查获取的信息来维护负载平衡信息，从而使其能够在遵循其指定的路由规则的情况下智能分配目标实例之间的流量。

## 流量管理的好处

使用Istio的流量管理模型基本上将交通流量和基础设施扩展分离，让运维通过Pilot为流量配置规则，而不是哪些特定的Pod/VM应该接收流量 - Pilot和智能Envoy代理负责监督其余流量。因此，例如，你可以通过Pilot指定您希望特定服务的5％流量转换为金丝雀版本，而不用考虑金丝雀版本的大小，或根据请求内容将流量发送到特定版本。

![Traffic Management with Istio](https://istio.io/docs/concepts/traffic-management/img/pilot/TrafficManagementOverview.svg)

将流量从基础架构中分离出来，使Istio能够提供各种流量管理功能，而且这些功能与应用代码分离。除了A/B测试，滚动部署和金丝雀发布之外，它还使用超时、重试和断路器处理故障恢复，以及故障注入，以测试跨服务的故障恢复策略的兼容性。 这些功能都是通过在服务网格中部署的Envoy sidecars/proxy 实现的。

# Pilot

Pilot负责整个Istio服务网格中部署的Envoy实例的生命周期。

![Pilot Architecture](https://istio.io/docs/concepts/traffic-management/img/pilot/PilotAdapters.svg)





如上图所示，Pilot维护一个独立于底层平台的服务网格，所有的服务都通过Envoy 加入服务风格中。Pilot中的Platform Adapter 负责适当地填充此规范模型。 例如，Pilot中的Kubernetes适配器实现必要的controllers，以观察Kubernetes API server以更改pod注册信息、ingress 资源和存储流量管理规则的第三方资源。 这些数据被翻译成规范的表示。Envoy的配置是基于规范表（canonical representation）示生成的。

Pilot公开了用于 [服务发现](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/sds) 的API，可动态更新[负载平衡池](https://www.envoyproxy.io/docs/envoy/latest/configuration/cluster_manager/cds)和[路由表](https://www.envoyproxy.io/docs/envoy/latest/configuration/http_conn_man/rds)。这些API将Envoy与平台特定的细微差别分离开来，简化了设计并提高了跨平台的可移植性。

运维可以通过[Pilot的规则API](https://istio.io/docs/reference/config/istio.routing.v1alpha1/)指定高级流量管理规则。 这些规则被转换成低级（low-level）配置并通过discovery API分发给Envoy实例。

# Request Routing（请求路由）

本节描述如何在Istio服务网格中的服务之间路由请求。

## 服务模型和服务版本

如Pilot所述，服务网格中服务的规范表示是由Pilot维护的。服务的Istio模型与其在底层平台（Kubernetes，Mesos，Cloud Foundry等）中的表现方式无关。平台相关的适配器负责使用平台中的元数据填充Istio内部模型。

Istio引入了服务版本的概念，该版本是按版本(`v1`, `v2`)或环境(`staging`, `prod`)细分服务实例的更细化的方式。 这些变体不一定是不同的API版本：它们可能是对同一服务的迭代更改，部署在不同的环境中（prod，staging，dev等）。 常用的场景包括A/B测试或金丝雀发布。 Istio的[流量路由规则](https://istio.io/docs/concepts/traffic-management/rules-configuration/)可以指服务版本，以提供对服务之间流量的额外控制。

## 服务之间的通信

![Showing how service versions are handled.](https://istio.io/docs/concepts/traffic-management/img/pilot/ServiceModel_Versions.svg)

如上图所示，client不知道服务的不同版本。他们可以继续使用服务的主机名/IP地址访问服务。 Envoy sidecar/proxy 拦截并转发client和服务之间的所有请求/响应。

Envoy 根据运运维人员使用Pilot指定的路由规则动态确定其实际服务版本。该模型使应用程序代码能够将其自身从其相关服务的演变中分离出来，同时还提供其他好处（请参[Mixer](https://istio.io/docs/concepts/policies-and-telemetry/overview/)）。 路由规则允许Envoy根据标准选择版本，例如headers，与source/destination关联的标签，分配给每个版本的权重。

Istio还为流量提供相同服务版本的多个实例的负载平衡。 您可以在[Discovery和Load-Balancing](https://istio.io/docs/concepts/traffic-management/load-balancing/)中找到更多关于此的信息。

Istio不提供DNS。 应用程序可以尝试使用底层平台中存在的DNS服务（kube-dns，mesos-dns等）来解析FQDN。

## Ingress 和 egress

Istio假设进入和离开服务网格的所有流量都通过Envoy代理。通过在服务前面部署Envoy代理，运维人员可以进行A/B测试，金丝雀部署服务等，以用于面向用户的服务。 同样，运维人员可以通过sidecar Envoy将流量路由到外部Web服务（例如，访问Maps API或视频服务API），从而增加故障恢复功能，例如超时，重试，断路器等，并获取详细信息 有关这些服务连接的度量标准。

![Ingress and Egress through Envoy.](https://istio.io/docs/concepts/traffic-management/img/pilot/ServiceModel_RequestFlow.svg)

# 服务发现和负载均衡

本节介绍Istio如何负载均衡服务网格中服务实例间的流量。

**服务注册**：Istio假设存在一个服务注册中心，以跟踪应用程序中服务的Pod/VM。它还假设新的服务实例会自动注册到服务注册中心，不健康的实例会被自动删除。Kubernetes，Mesos等平台已经为基于容器的应用程序提供了这样的功能。 基于VM的应用程序存在过多的解决方案。

**服务发现**：Pilot 使用来自服务注册中心的信息并提供与平台无关的服务发现接口。Mesh中的Envoy实例执行服务发现并相应地动态更新其负载均衡池。

![Discovery and Load Balancing](https://istio.io/docs/concepts/traffic-management/img/pilot/LoadBalancing.svg)

如上图所示，网格中的服务使用其DNS名称相互访问。 绑定到服务的所有HTTP流量都会通过Envoy自动重新路由。 Envoy 在负载均衡池中的实例之间分配流量。 虽然Envoy支持[多种复杂的负载均衡算法](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/load_balancing)，但Istio目前允许三种负载均衡模式：轮询，随机和权重最小请求。

除了负载平衡之外，Envoy还会定期检查负载均衡池中每个实例的健康状况。Envoy遵循断路器配置模式，根据健康检查API调用的故障率将实例归类为健康或不健康。 换句话说，当给定实例的状况检查失败次数超过预先指定的阈值时，它将从负载均衡池中移除。类似地，当运行状况检查数超过预先指定的健康阈值时，实例将被添加回负载均衡池。 你可以在[处理故障](https://istio.io/docs/concepts/traffic-management/handling-failures/)中找到有关Envoy故障处理功能的更多信息。

服务可以通过HTTP 503响应健康检查来积极减轻负载。 在这种情况下，服务实例将立即从调用者的负载平衡池中移除。

# 故障处理

Envoy提供了一套开箱即用的插拔式故障恢复功能，可以在应用程序中利用这些功能。功能包括：

1. 超时
2. 根据超时时间和重试次数、间隔等配置进行重试
3. 并发连接数和对上游服务的请求限制
4. 对负载均衡池的每个成员进行主动（定期）健康检查
5. 针对负载平衡池中针对每个实例应用的细粒度断路器（被动健康检查）

这些功能可以在运行时通过[Istio的流量管理规则](https://istio.io/docs/concepts/traffic-management/rules-configuration/)进行动态配置。

重试之间的抖动使重试对超负载的上游服务的影响最小，而超时设置可确保调用服务在可预测的时间范围内获得响应（成功/失败）。

主动和被动健康检查（上述4和5）的组合可最大限度地减少访问负载均衡池中不健康实例的机会。与平台级健康检查（如Kubernetes或Mesos支持的那些）结合使用时，应用程序可以确保不健康的pods/容器/虚拟机可以快速服务网格中清除，从而最大限度地减少请求失败并减少对延迟的影响。

这些功能一起使服务网格能够容忍失败的nodes ，并防止由于级联不稳定而导致本地故障影响到其他nodes。

## 微调

Istio的流量管理规则允许运维为每个服务/版本的故障恢复设置全局默认值。但是，服务的使用者也可以通过特殊的HTTP头提供请求级覆盖来覆盖超时和重试默认值。 通过Envoy代理实现，头文件分别是 `x-envoy-upstream-rq-timeout-ms` 和 `x-envoy-max-retries`。

## FAQ

**Q: 在Istio中运行时，应用程序是否还能处理失败？**

可以。 Istio提高了网格中服务的可靠性和可用性。但是，应用程序需要处理失败（错误）并采取适当的后备（fallback）操作。例如，当负载平衡池中的所有实例都失败时，Envoy将返回HTTP 503.应用程序负责处理来自上游服务的HTTP 503错误。

**Q: Envoy的故障恢复功能是否会中断已使用容错库（例如Hystrix）的应用程序？**

不会。Envoy 对应用是完全透明的。由Envoy返回的失败响应与由进行调用的上游服务返回的失败响应无法区分。

**Q: 在同时使用应用程序级类库和Envoy时如何处理失败？**

给同一个服务的两个故障恢复策略（例如，两个超时 - 一个设置在Envoy中，另一个设置在应用程序的库中），当故障发生时，两个限制中较强的将会被触发。 例如，如果应用程序为服务的API调用设置5秒的超时，而操作员配置了10秒的超时，则应用程序的超时将首先启动。 同样，如果Envoy的断路器在应用程序的断路器之前触发，API调用服务将从Envoy获得503。

# 故障注入

虽然Envoy sidecar/proxy为运行在Istio上的服务提供了大量[故障恢复机制](https://istio.io/docs/concepts/traffic-management/handling-failures/)，但仍然有必要测试整个应用程序的端到端故障恢复能力。 错误配置的故障恢复策略（例如，跨服务调用的不兼容/限制性超时）可能导致应用程序中关键服务持续不可用，从而导致用户体验不佳。

Istio支持将指定协议的故障注入到网络中，而不是杀死Pod，延迟或破坏TCP层的数据包。 我们的基本原理是，无论网络级别失败，应用层观察到的失败都是相同的，并且可以在应用层注入更有意义的失败（例如，HTTP错误代码）以增加应用程序的弹性。

运维人员可以配置故障注入到符合特定标准的请求中。运维可以进一步限制应该发生故障的请求的百分比。可以注入两种类型的故障：延迟和中止。延迟是定时失败，模仿网络延迟增加或上游服务超负荷。中止是模仿上游服务故障的崩溃故障。 中止通常以HTTP错误代码或TCP连接失败的形式出现。

有关更多详细信息，请参阅下一节[Istio的流量管理规则](https://istio.io/docs/concepts/traffic-management/rules-configuration/)。

# 规则配置

Istio提供了一个简单的配置模型来控制API调用和第4层流量如何在应用程序中的各种服务之间流动。配置模型允许运维配置服务级属性，如断路器，超时，重试，以及设置常见的持续部署任务，如金丝雀部署，A/B测试，基于百分比流量的分阶段发布等。

例如，可以使用以下配置将reviews服务的传入流量的100％发送到版本“v1”：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1

```

该配置表示发送到reviews服务（在`hosts` 字段中指定）的流量应该路由到reviews服务实例的v1子集中。 路由`subset` 在相应的目标规则配置中指定定义的子集的名称：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2

```

subset 指定一个或多个标识版本实例的标签。例如，在Istio的Kubernetes部署中，“version：v1”表示只有包含“version：v1”标签的pod才会接收流量。

可以使用[istioctl CLI](https://istio.io/docs/reference/commands/istioctl/)配置规则，也可以使用kubectl命令在Kubernetes deployment 中配置规则，但只有istioctl 会验证配置是否正确，我们建议使用istioctl 。 有关示例，请参阅[配置请求路由任务](https://istio.io/docs/tasks/traffic-management/request-routing/)。

Istio中有四种流量管理配置资源：VirtualService，DestinationRule，ServiceEntry和Gateway。 下面介绍使用这些资源的一些重要方面。详细信息请参阅[参考](https://istio.io/docs/reference/config/istio.networking.v1alpha3/)。

## Virtual Services

[VirtualService](https://istio.io/docs/reference/config/istio.networking.v1alpha3/#VirtualService)定义了控制服务请求如何在Istio服务网格中路由的规则。例如，virtual service可以将请求路由到不同版本的服务，或者实际上可以将请求路由到完全不同的服务。 请求可以根据请求源和目标、HTTP路径和header 字段以及与各个服务版本相关的权重进行路由。

### Rule destinations

路由规则对应于VirtualService配置中指定的一个或多个请求目标hosts。 这些hosts可能与实际的目标工作负载相同也可能不同，甚至可能没有对应网格中的实际可路由服务。例如，要使用其内部网格名称reviews 或通过hostbookinfo.com为请求评论服务定义路由规则，VirtualService可以有一个hosts字段，如下所示：

```yaml
hosts:
  - reviews
  - bookinfo.com
```
hosts字段隐式或显式指定一个或多个全限定域名（FQDN）。上面的短名称reviews将隐式扩展为FQDN。 例如，在Kubernetes环境中，reviews完整的域名为来自VirtualSevice的集群和名称空间（例如，reviews.default.svc.cluster.local）。

### 通过source/headers 来限定规则

Rules 可以选择性地限定为仅适用于匹配某些特定条件的请求，例如以下条件：

1. 限制为特定调用者。 例如，Rules 可以指定ratings仅适用于来自reviews 服务（pods）的的调用。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
      sourceLabels:
        app: reviews
    ...

```

`sourceLabels` 的值取决于service的实现。 例如，在Kubernetes中，它可能与相应Kubernetes 中pod选择器中使用的标签相同。

2. 限制为特定版本的调用者。 例如，以下规则将之前示例限定为仅允许*reviews* 服务“v2”版本的调用。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
    - sourceLabels:
        app: reviews
        version: v2
    ...

```

3. 根据HTTP headers选择规则。 例如，以下规则仅适用于包含字符串“user = jason”的“cookie”头的请求。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - match:
    - headers:
        cookie:
          regex: "^(.*?;)?(user=jason)(;.*)?$"
    ...

```

如果配置了多个header，则必须所有的header全部匹配才可以。

可以同时设置多个标准。 在这种情况下，根据嵌套情况，使用AND或OR语义。 如果多个条件嵌套在单个匹配子句中，则条件为ANDed。 例如，以下规则仅适用于请求的来源为“reviews:v2”且包含“user = jason”的“cookie”的情况。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
    - sourceLabels:
        app: reviews
        version: v2
      headers:
        cookie:
          regex: "^(.*?;)?(user=jason)(;.*)?$"
    ...

```

相反，如果条件显示在单独的匹配条件中，则有一个匹配即可（OR语义）：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
    - sourceLabels:
        app: reviews
        version: v2
    - headers:
        cookie:
          regex: "^(.*?;)?(user=jason)(;.*)?$"
    ...

```

### 在服务版本之间拆分流量

每条路由规则标识一个或多个加权后端，以便在规则激活时进行调用。每个后端对应于目标服务的特定版本，其中版本可以使用标签表示。如果多个已注册实例有相同的标签，则根据配置的负载平衡策略进行路由，默认为循环。

例如，以下规则将reviews 服务的25％流量路由到具有“v2”标签的实例，剩余流量（即75％）路由到“v1”。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 75
    - destination:
        host: reviews
        subset: v2
      weight: 25

```

### 超时和重试

默认情况下，http请求的超时时间为15秒，但是这可以在路由规则中配置，如下所示：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
    - ratings
  http:
  - route:
    - destination:
        host: ratings
        subset: v1
    timeout: 10s

```

也可以在路由规则中配置http请求的重试次数。 在超时期限内，最大尝试次数或尽可能多的尝试次数可以设置如下：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
    - ratings
  http:
  - route:
    - destination:
        host: ratings
        subset: v1
    retries:
      attempts: 3
      perTryTimeout: 2s

```

请注意，请求超时和重试也可以[基于每个请求](https://istio.io/docs/concepts/traffic-management/handling-failures#fine-tuning)进行配置。

请参阅[请求超时任务](https://istio.io/docs/tasks/traffic-management/request-timeouts/)以了解超时控制的示例。

### 注入请求路径中的错误

路由规则可以指定一个或多个故障注入（人为制造故障），同时将http请求转发到规则的相应请求目标。 故障可以是延迟或中止。

以下示例将向微服务ratings “v1”版本的10％请求中引入5秒延迟。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - fault:
      delay:
        percent: 10
        fixedDelay: 5s
    route:
    - destination:
        host: ratings
        subset: v1

```

另一种类型的故障，中止，可用于模拟请求中断故障。

以下示例会将ratings 服务“v1”版本10％请求的HTTP请求返回 400错误代码。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - fault:
      abort:
        percent: 10
        httpStatus: 400
    route:
    - destination:
        host: ratings
        subset: v1

```

有时延迟和中止故障一起使用。例如，以下规则将从reviews 服务“v2”到评ratings 服务“v1”的所有请求延迟5秒，然后中止10％：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
    - sourceLabels:
        app: reviews
        version: v2
    fault:
      delay:
        fixedDelay: 5s
      abort:
        percent: 10
        httpStatus: 400
    route:
    - destination:
        host: ratings
        subset: v1
```

要查看故障注入的实际情况，请参阅[故障注入任务](https://istio.io/docs/tasks/traffic-management/fault-injection/)。

### HTTP路由规则优先级

当给定destination有多个规则时，它们按照它们在VirtualService中出现的顺序进行评估，即列表中的第一个规则具有最高优先级。

**为什么优先级重要？** 如果服务的路由规则纯粹是基于权重的时候，就可以在单个规则中配置它。但是，当使用其他标准（例如，来自特定用户的请求）来路由流量时，需要多个规则来配置路由。这里必须仔细考虑规则优先级，以确保以正确的顺序执行规则。

通用路由规范的一种常见模式是提供一个或多个优先级更高的规则，以便通过source/headers来限定规则，然后提供一个没有匹配条件的单个基于权重的规则，最后为所有其他情况提供流量的加权分配。

例如，以下VirtualService包含2个规则，这些规则一起指定包含名为“Foo”且值为“bar”的header的reviews服务的所有请求都将发送到“v2”实例。 所有其余的请求将被发送到“v1”。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        Foo:
          exact: bar
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1

```

请注意，这里配置中基于header的规则具有更高的优先级。如果它较低，这些规则将不会如预期的那样工作，因为基于权重的规则（没有特定的匹配标准）将首先被评估，然后将所有流量简单地路由到“v1”，甚至包括匹配的“Foo “头。一旦找到适用于传入请求的规则，它将被执行并且规则评估过程将终止。 这就是为什么当存在多个规则时仔细考虑每个规则的优先级是非常重要的。

## Destination Rules

[DestinationRule](https://istio.io/docs/reference/config/istio.networking.v1alpha3/#DestinationRule)配置在虚拟服务路由发生后，应用于请求的一组策略。它们由服务所有者去配置定义，描述断路器，负载平衡器设置，TLS设置等。

DestinationRule还定义相应目的地主机的可寻址子集（即，版本名）。 在将流量发送到特定版本的服务时，这些子集用于VirtualService路由规范。

以下`DestinationRule` 配置reviews 服务的策略和subsets ：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  trafficPolicy:
    loadBalancer:
      simple: RANDOM
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
  - name: v3
    labels:
      version: v3

```

请注意，可以在单个DestinationRule配置中指定多个策略（例如，默认和v2）。

### 断路器

可以根据许多标准（例如连接和请求限制）设置简单的断路器。

例如，以下DestinationRule设置了与reviews 服务版本“v1”后端限制100个连接。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
    trafficPolicy:
      connectionPool:
        tcp:
          maxConnections: 100

```

查看断路器控制的[断路任务](https://istio.io/docs/tasks/traffic-management/circuit-breaking/)演示。

### DestinationRule evaluation