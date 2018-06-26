---
categories: Istio
tags:
  - Istio

title: Istio 综述
date: 2018-06-26
---

# 综述

本文档介绍Istio：一个用于连接，管理和保护微服务的开放式平台。Istio提供了一种简单的方法，通过负载均衡，服务到服务的认证，监控等为已部署服务的创建网络，并且无需对服务代码做任何更改。可以通过在整个环境中部署一个特殊的sidecar 代理来将Istio功能，sidecar截取微服务之间的所有网络通信，使用Istio的控制面板（control plane）功能进行配置和管理。

Istio目前支持在Kubernetes上部署服务，以及通过Consul或Eureka注册的服务以及在单个VM上运行的服务。

有关Istio组件的详细概念信息，请参阅其他[概念](https://istio.io/docs/concepts/)指南。

## 为什么使用Istio

Istio解决了从单一应用程序向分布式微服务架构过渡中开发和运维人员面临的许多挑战。 术语**service mesh** （服务网格）通常用于描述构成这些应用程序的微服务网络以及它们之间的交互。 随着服务网格的大小和复杂程度不断增加，可能会变得难以理解和管理。 Istio提供了包括服务发现，负载平衡，故障恢复，度量和监控，以及更复杂的操作要求，如A / B测试，金丝雀版本，流量限制，访问控制和端到端身份验证等。

Istio提供了一个完整的解决方案，通过对整个服务网格提供行为分析和操作控制来满足创建微服务应用程序的各种需求。 它在整个服务网络中统一提供许多关键功能：

- **流量管理**。控制业务之间的流量和API调用流量，使调用更可靠，并在面对不利条件时使网络更加健壮。
- **服务标识和安全**。 在网格中提供具有可验证身份的服务，并提供保护服务流量的能力，因为流量通过不同程度的可信性网络流动。
- **策略执行**。 将组织策略应用于服务之间的交互，确保访问策略得到执行，资源在消费者之间公平分配。 通过配置网格来进行策略更改，而不是通过更改应用程序代码。
- **Telemetry**。 了解服务之间的依赖关系以及它们之间数据传输的性质和流量，从而提供快速诊断问题的能力。

除了以上功能之外，Istio可扩展设计可以满足不同的部署需求：

- **平台支持**。 Istio旨在运行在各种环境中，包括跨云，On-premises 部署，Kubernetes，Mesos等。我们最初专注于Kubernetes，但正在努力支持其他环境。
- **整合和定制**。 策略实施组件可以进行扩展和定制，以与ACL，日志记录，监控，配额，审计等现有解决方案集成。

这些功能大大减少了应用程序代码，底层平台和策略之间的耦合。 这种耦合减少不仅使服务更实现起来更容易，而且还使运维人员能够更简单地将应用在不同环境之间部署或在新策略方案之间移动。 因此，应用程序本质上更加轻便。

## 架构

Istio服务网格逻辑上分为数据面板和控制面板。

- 数据面板由一系列智能代理（Envoy）组成，这些智能代理部署为sidecars （边车），用于调解和控制微服务之间的所有网络通信，以及通用策略和监控日志中枢（混合器）。
- 控制面板负责管理和配置代理以路由流量，并配置Mixers 以执行策略并收集监控日志数据。

下图显示了组成每个面板的不同组件：

![The overall architecture of an Istio-based application.](https://istio.io/docs/concepts/what-is-istio/img/overview/arch.svg)

### Envoy

Istio 是[Envoy](https://envoyproxy.github.io/envoy/)代理的扩展版本，这是一种使用C ++开发的高性能代理，用于调解服务网格中所有服务的所有入站和出站流量。 Istio利用Envoy的许多内置功能，如动态服务发现，负载平衡，TLS终止，HTTP/2 和gRPC代理，断路器，运行状况检查，基于百分比的流量的分阶段发布，故障注入以及丰富的metrics。

Envoy 以sidecar的形式部署在同一Kubernetes Pod相关服务中。这使Istio能够将大量有关流量行为的信号作为[属性](https://istio.io/docs/concepts/policies-and-telemetry/config/#attributes)提取出来，从而可以在[Mixer](https://istio.io/docs/concepts/policies-and-telemetry/overview/) 中使用它来执行策略决策，并将其发送到监控系统以提供有关整个网格行为的信息。 sidecar代理模型还允许将Istio功能添加到已部署服务，而无需重新构建或重写代码。 你可以阅读更多关于我们为什么在我们的[设计目标](https://istio.io/docs/concepts/what-is-istio/goals/)中选择此方法的信息。

### Mixer

[Mixer](https://istio.io/docs/concepts/policies-and-telemetry/overview/) 是一个独立于平台的组件，负责在整个服务网格中实施访问控制和使用策略，并收集来自Envoy代理和其他服务的遥测数据。代理提取请求级别[属性](https://istio.io/docs/concepts/policies-and-telemetry/config/#attributes)，将其发送到Mixer进行评估。有关此属性提取和策略评估的更多信息可以在[Mixer配置](https://istio.io/docs/concepts/policies-and-telemetry/config/)中找到。 Mixer包含灵活的插件模型，可以与各种主机环境和基础设施后端进行交互，从这些细节中抽象出Envoy代理和Istio管理的服务。

### Pilot

Pilot为Envoy sidecars提供服务发现，为智能路由（例如A / B测试，金丝雀部署等）和resiliency（超时，重试，断路器等）提供流量管理功能。它将控制流量行为的高级路由规则转换为特定的Envoy的配置，并在运行时将它们传播到sidecar。Pilot将平台特定的服务发现机制抽象化并将其合成为符合[Envoy数据平面API](https://github.com/envoyproxy/data-plane-api)的任何sidecar都可以使用的标准格式。这种松散耦合使得Istio能够在多种环境下运行（例如，Kubernetes，Consul / Nomad），同时用于流量管理的界面操作都是相同的。

### Citadel

[Citadel](https://istio.io/docs/concepts/security/)提供强大的服务到服务和终端用户身份验证，内置身份和凭证管理。它可用于升级服务网格中未加密的流量，并为运维人员提供基于服务标识而不是网络控制强制执行策略的能力。 从0.5版开始，Istio支持[基于角色的访问控制](https://istio.io/docs/concepts/security/rbac/)，以控制谁可以访问您的服务。

# 设计目标

Istio的体系结构由几个关键设计目标，这些目标对于使系统能够处理大规模和高性能的服务至关重要。

- **最大化透明**。 采用Istio，可以让运维人员或开发人员尽可能减少工作量，以便从系统中获得更多实际价值。 为此，Istio可以自动将自己注入到服务之间的所有网络路径中。Istio使用sidecar代理来捕获流量，并在可能的情况下自动编程联网层，以便通过这些代理路由流量，而不会对已部署的应用程序代码进行任何更改。 在Kubernetes中，代理被注入到pod中，通过编程iptables规则捕获流量。 一旦sidecar代理被注入并且编程好流量路由规则，Istio就能够调解所有的流量。 这一原则也适用于性能。 将Istio应用于部署时，运维人员应该看到Istio提供这些功能只需要很少的资源。 组件和API的设计必须考虑到性能和可伸缩。
- **渐进性**。 随着运维人员和开发人员越来越依赖Istio提供的功能，系统必须随着他们的需求而增长。 虽然我们希望自己继续添加新功能，但我们预计最大的需求是扩展策略系统，与其他策略和控制来源整合，并将有关网状行为的信号传播到其他系统进行分析。 策略运行时支持插入其他服务的标准扩展机制。 此外，它还允许扩展其词汇表，以便根据网格生成的新信号执行策略。
- **可移植性**。 Istio将使用的生态系统在很多方面都有所不同。 Istio必须以最小的努力在任何云环境或内部环境中运行。 将基于Istio的服务移植到新环境中的工作量是非常小的，并且应该可以使用Istio将一个服务部署到多个环境中（例如，在多个云上进行冗余）。
- **策略一致性**。 将策略应用于服务之间的API调用可以提供对网格行为的大量控制，但将策略应用于不一定在API级别表示的资源也同样重要。 例如，将配额应用于ML培训任务所消耗的CPU数量比将配额应用于启动工作的调用更有用。 为此，该策略体系作为一项独立的服务与其自己的API保持在一起，而不是被纳入代理/sidecar，从而允许服务根据需要与其直接整合。

