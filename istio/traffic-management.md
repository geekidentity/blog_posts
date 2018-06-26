# 综述

本页面概述了Istio中流量管理的工作原理，包括流量管理原则的优点。我们假定你已经阅读了[什么是Istio](https://istio.io/docs/concepts/what-is-istio/overview/)？ 并熟悉Istio的高层架构。 您可以在本节的其他指南中找到有关流量管理功能的更多信息。

## Pilot 和Envoy

Istio中流量管理的核心组件是[Pilot](https://istio.io/docs/concepts/traffic-management/pilot/)，它管理和配置部署在Istio服务网格中的所有Envoy代理实例。它允许你指定要使用哪些规则在Envoy代理之间路由流量，并配置故障恢复功能，例如超时，重试和断路器。 它还维护网格中所有服务的规范模型，并使用它来让Envoys 通过其服务发现了解网格中的其他实例。

每个Envoy实例根据从Pilot获取的信息和其负载平衡池中其他实例的定期运行状况检查获取的信息来维护负载平衡信息，从而使其能够在遵循其指定的路由规则的情况下智能分配目标实例之间的流量。

## 流量管理的好处

使用Istio的流量管理模型基本上将交通流量和基础设施扩展分离，让运营商通过Pilot指定他们希望流量遵循哪些规则，而不是哪些特定的Pod / VM应该接收流量 - 试点和智能Envoy代理负责监督剩余流量。 因此，例如，您可以通过Pilot指定您希望特定服务的5％流量转换为金丝雀版本，而不考虑金丝雀部署的大小，或根据请求内容将流量发送到特定版本。

![Traffic Management with Istio](https://istio.io/docs/concepts/traffic-management/img/pilot/TrafficManagementOverview.svg)