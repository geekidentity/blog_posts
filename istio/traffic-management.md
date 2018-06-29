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