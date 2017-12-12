---
categories: spring

tags: 
  - Spring Cloud
  - Java
  - Spring


title: Spring Cloud 项目综述（技术栈一览）

date: 2017-04-10
---

## 前言

Spring Cloud 为构建分布式系统和微服务提供了一些通用的工具，例如：配置中心，服务注册与发现，熔断器，路由，代理，控制总线，一次性令牌，全局锁，leader选举，分布式 会话，集群状态等。

目前国内有很多公司还是使用dubbo做服务分解，但dubbo只提供了服务注册发现功能，要建立分布式系统还要自己找对应工具进行组合，当然这样定制性、灵活性高，但有些技术要摸着走，而且阿里已经停止了对dubbo的更新。

如果采用Spring Cloud技术栈，Spring Cloud提供了分布式系统和微服务中所需要的约大多数公共模块和功能，

Spring Cloud 下各项目都是基于 [Spring Boot](http://blog.geekidentity.com/spring/spring_boot_translation/) 的，所有要想用Spring Cloud做微服务开发，最好先掌握 [Spring Boot](http://blog.geekidentity.com/spring/spring_boot_translation/)。

下表是dubbox与Spring Cloud技术栈对比

功能 | Dubbox | Spring Cloud
---|---|---
服务注册中心 | Zookeeper | Spring Cloud Netflix Eureka
服务调用方式 | RPC/REST API | REST API
服务网关 | 无 | Spring Cloud Netflix Zuul
熔断器 | 不完善 | Spring Cloud Netflix Hystrix
分布式配置 | 无 | Spring Cloud Config
服务跟踪 | 无 | Spring Cloud Sleuth
消息总线 | 无 | Spring Cloud Bus
数据流 | 无 | Spring Cloud Stream
批量任务 | 无 | Spring Cloud Task

## [Spring Cloud Cluster](http://projects.spring.io/spring-cloud)

Zookeeper，Redis，Hazelcast，Consul的leader选举和公共的状态模式的抽象和实现。

### 简介

Spring Cloud为开发人员提供了快速构建分布式系统中的一些通用模式(patterns)的工具（例如配置管理，服务发现，熔断器，智能路由，微代理，控制总线(control bus)，一次性令牌，全局锁，leader选举，分布式 会话，集群状态）。 分布式系统的协调导致锅炉板模式(boiler plate patterns)，并且使用Spring Cloud开发人员可以快速开发出(stand up)实现这些模式的服务和应用程序。 程序将在任何分布式环境中都可以良好的运行，包括开发人员自己的笔记本电脑，裸机数据中心，以及像Cloud Foundry的托管平台。

Spring Cloud基于Spring Boot，通过提供的一组类库，可以在增强应用程序的行为。 您可以利用基本的默认行为(behaviour)/配置快速入门，然后在需要时，您可以配置或扩展以创建自定义解决方案。

### 快速开始

```XML
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.4.5.RELEASE</version>
</parent>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Camden.SR6</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
</dependencies>
```


### 功能

Spring Cloud致力于为典型的使用案例和扩展机制提供良好的开箱即用的体验。

* 分布式/版本化配置
* 服务注册和发现
* 路由
* 服务到服务(Service-to-service)的调用
* 负载均衡
* 熔断器(Circuit Breakers)
* 全局锁
* Leader选举和集群状态
* 分布式消息

Spring Cloud采用注解声明的方式，通常只需要一个类路径和(或)解释更改即可获得很多功能。作为发现客户端的示例应用程序：

```Java
@SpringBootApplication
@EnableDiscoveryClient
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

## [Spring Cloud Config](http://cloud.spring.io/spring-cloud-config)

由git仓库支持的统一配置管理。配置资源直接映射到Spring`Environment`，但如果需要，可以使用非Spring应用程序。

### 简介

Spring Cloud Config为分布式系统中的外部统一配置中心提供服务器和客户端支持。使用Config Server，您可以在所有环境中管理应用程序的外部(externalized)配置属性。客户端和服务器映射的概念与Spring Environment和PropertySource抽象相同，因此它们与Spring应用程序非常契合，但可以与任何语言的应用程序一起使用。伴随着应用程序通过从开发环境到测试环境和生产环境的部署过程，您可以管理这些环境之间的配置，并确定应用程序不同环境迁移时需要所有配置属性。服务器存储端的默认实现使用git，因此它可以轻松支持带标签版本的配置环境，以及可以访问用于管理的内容的各种工具。可以轻松添加替代实现，并使用Spring配置将其插入。

### 功能

Spring Cloud Config Server功能：

* 基于HTTP资源的外部配置API（名称/值对或等效的YAML内容）
* 对属性值加密和解密（对称或非对称）
* 可以使用@EnableConfigServer轻松嵌入Spring Boot应用程序

Config Client功能（适用于Spring应用程序）：

* 绑定到Config Server并使用远程的属性源初始化Spring Environment
* 对属性值加密和解密（对称或非对称）

### 快速开始

以使用Maven为项目依赖管理为例：

```XML
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config</artifactId>
            <version>1.3.1.BUILD-SNAPSHOT</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
</dependencies><repositories>
    <repository>
        <id>spring-snapshots</id>
        <name>Spring Snapshots</name>
        <url>https://repo.spring.io/libs-snapshot</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
```
只要Spring Boot Actuator和Spring Config Client类库在类路径中，任何Spring Boot应用程序将尝试连接http://localhost:8888（spring.cloud.config.uri的默认值）的配置服务器：

```Java
@Configuration
@EnableAutoConfiguration
@RestController
public class Application {

  @Value("${config.name}")
  String name = "World";

  @RequestMapping("/")
  public String home() {
    return "Hello " + name;
  }

  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }

}
```
示例中的config.name的值（或在Spring Boot以正常方式绑定的任何其他值）可以来自本地配置或远程Config Server。 默认情况下，Config Server将优先。要查看应用程序中的/env端点，请参阅configServer属性源。

要运行自己的服务器，请使用spring-cloud-config-server依赖项和@EnableConfigServer注解。如果您设置spring.config.name=configserver，则应用程序将在端口8888上运行，并从样本存储库(sample repository)提供数据。 您需要一个spring.cloud.config.server.git.uri来找到自己需要的配置数据（默认情况下，它是git存储库的位置，可以是本地url:.. URL）。

## [Spring Cloud Netflix](http://cloud.spring.io/spring-cloud-netflix)

集成各种Netflix OSS组件(Eureka, Hystrix, Zuul, Archaius等)。

### 简介

Spring Cloud Netflix通过自动配置、绑定到Spring Environment 和其他Spring编程模型语法来为Spring Boot应用程序提供Netflix OSS集成。通过几个简单的注解，您可以快速启用和配置应用程序中的常见模式，并使用经过考验的Netflix组件构建大型分布式系统。 提供的常见模式包括服务发现（Eureka），融断机制（Hystrix），智能路由（Zuul）和客户端负载平衡（Ribbon）。

### 功能

Spring Cloud Netflix功能：

* 服务发现：可以注册Eureka实例，客户端可以使用Spring管理的bean来发现实例
* 服务发现：可以使用声明式Java配置创建嵌入式Eureka服务器
* 融断机制：Hystrix客户端可以使用简单的注释驱动方法装饰器构建
* 融断机制：具有声明式Java配置的嵌入式Hystrix仪表板
* 声明性REST客户端：Feign创建了一个使用JAX-RS或Spring MVC注解装饰的接口的动态实现
* 客户端负载均衡器：Ribbon
* 外部配置：从Spring Environment 到Archaius的桥梁（使用Spring Boot约定启用Netflix组件的本地配置）
* 路由器和过滤器：Zuul过滤器的自动注册，以及反向代理创建的简单配置方法

### 快速开始

Maven：

```XML
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-netflix</artifactId>
            <version>1.3.1.BUILD-SNAPSHOT</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
</dependencies><repositories>
    <repository>
        <id>spring-snapshots</id>
        <name>Spring Snapshots</name>
        <url>https://repo.spring.io/libs-snapshot</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
```
只要Spring Cloud Netflix和Eureka Core类库在类路径中，任何具有@EnableEurekaClient的Spring Boot应用程序将尝试连接 http://localhost:8761 （eureka.client.serviceUrl.defaultZone的默认值）上的Eureka服务器：

```Java
@Configuration
@EnableAutoConfiguration
@EnableEurekaClient
@RestController
public class Application {

  @RequestMapping("/")
  public String home() {
    return "Hello World";
  }

  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }

}
```
要运行自己的服务器，请添加依赖spring-cloud-starter-eureka-server和@EnableEurekaServer注解。


## [Spring Cloud Bus](http://cloud.spring.io/spring-cloud-bus)

用于将服务和服务实例以及分布式消息传递链接的事件总线。 用于在集群中传播状态更改（例如配置更改事件）。

### 简介

Spring Cloud Bus将分布式系统的节点与轻量级消息代理连接起来。 这可以用于广播状态改变（例如配置改变）或其他管理指令。 目前唯一的实现是使用AMQP代理作为传输，但是相同的基本功能集（还有一些取决于传输）在其他传输的路线图上。

### 快速开始

Maven：

```XML
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-bus-parent</artifactId>
            <version>1.3.0.M1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bus-amqp</artifactId>
    </dependency>
</dependencies><repositories>
    <repository>
        <id>spring-milestones</id>
        <name>Spring Milestones</name>
        <url>https://repo.spring.io/libs-milestone</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
</repositories>
```
只要Spring Cloud Bus AMQP和RabbitMQ类库在类路径中，任何Spring Boot应用程序将尝试连接 localhost:5672上的RabbitMQ服务器（spring.rabbitmq.addresses的默认值）：

```Java
@Configuration
@EnableAutoConfiguration
@RestController
public class Application {

  @RequestMapping("/")
  public String home() {
    return "Hello World";
  }

  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }

}
```

## Spring Cloud for Cloud Foundry

将您的应用程序与Pivotal Cloudfoundry集成。 提供服务发现实现，并且还可以轻松实现SSO和OAuth2保护的资源，还可以创建Cloudfoundry服务代理。

### 简介

Spring Cloud for Cloudfoundry可以轻松在Cloud Foundry（平台即服务）中运行Spring Cloud应用程序。 Cloud Foundry有一个“服务”的概念，它是“绑定”到应用程序的中间件，本质上为其提供包含凭据的环境变量（例如，用于服务的位置和用户名）。

### 功能

spring-cloud-cloudfoundry-web项目为Cloud Foundry中的webapps的一些增强功能提供了基本支持：自动绑定到单点登录服务，并可选择启用粘性路由进行发现。

spring-cloud-cloudfoundry-discovery项目提供了Spring Cloud Commons DiscoveryClient的实现，因此您可以@EnableDiscoveryClient并将提供您的凭据spring.cloud.cloudfoundry.discovery.[email，password]，然后可以直接使用DiscoveryClient或通过 一个LoadBalancerClient使用（如果您没有连接到Pivotal Web Services，也是 *.url）。

> 注意：如果您正在寻找一种绑定到服务的方法，那么这是错误的库。 请查看Spring Cloud连接器。

### 快速开始

Maven：

```XML
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-cloudfoundry-web</artifactId>
        <version>1.0.2.BUILD-SNAPSHOT</version>
    </dependency>
</dependencies><repositories>
    <repository>
        <id>spring-snapshots</id>
        <name>Spring Snapshots</name>
        <url>https://repo.spring.io/libs-snapshot</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
```

## [Spring Cloud Cloud Foundry Service 代理](http://cloud.spring.io/spring-cloud-cloudfoundry-service-broker/)

提供构建管理Cloud Foundry管理服务的服务代理的起点。

### 简介

Spring Cloud Cloud Foundry Service Broker是构建Spring Boot应用程序的框架，用于实现Cloud Foundry Service Broker API并管理Cloud Foundry Marketplace中提供的服务。

Cloud Foundry管理的服务由服务代理管理，服务代理通知其服务提供的服务计划，并提供，销毁，绑定和解除绑定服务实例。 Spring Cloud Cloud Foundry Service Broker提供了一个基于Spring Boot的框架，使您可以在Cloud Foundry上为您自己的托管服务快速创建代理。

### 功能

* 目录和服务绑定/解除端点的默认配置
* 支持异步服务操作（Cloud Foundry Service Broker API 2.7）
* 支持提供给cf Command Line Interface工具的任意参数
* 支持Cloud Foundry路由服务

### 快速开始

Maven：

```XML
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-cloudfoundry-service-broker</artifactId>
        <version>1.0.0.RELEASE</version>
    </dependency>
</dependencies>
```
要启用Spring Cloud Cloud Foundry Service Broker框架的默认配置，您的代理应用程序只需要在其主应用程序类中使用@EnableAutoConfiguration或@SpringBootApplication注解：

```Java
@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```
有关实现API功能的接口以及提供的默认实现的更多信息，请参阅[项目文档](https://github.com/spring-cloud/spring-cloud-cloudfoundry-service-broker/blob/master/README.md)。 有关开发Cloud Foundry管理服务的更多信息，请参阅[Cloud Foundry文档](http://docs.cloudfoundry.org/services/)。

## [Spring Cloud Consul](http://cloud.spring.io/spring-cloud-consul/)

Hashicorp Consul服务发现和配置管理。

### 简介

该项目通过自动配置、绑定到Spring Environment和其他Spring编程模型语法来为Spring Boot应用程序提供Consul集成。 通过几个简单的注释，您可以快速启用和配置应用程序中的常见模式，并使用基于Consul的组件构建大型分布式系统。 提供的模式包括服务发现，控制总线和配置。 智能路由（Zuul）和客户端负载平衡（功能区），断路器（Hystrix）通过与Spring Cloud Netflix的集成提供。

## [Spring Cloud Security](http://cloud.spring.io/spring-cloud-security)

在Zuul代理中支持负载均衡的OAuth2 rest 客户端和认证头转发。

### 简介

Spring Cloud Security提供了一套用于构建安全的原语级应用程序和最小化服务。 可以从外部（或集中）高度配置的声明式模型适用于通常使用中央契约管理服务的大型合作远程组件系统的实现。 在像Cloud Foundry这样的服务平台上也很容易使用。基于Spring Boot和Spring Security OAuth2，我们可以快速创建实现单点登录，令牌中继和令牌交换等常见模式的系统。

### 功能

* 在Zuul代理中将SSO令牌从前端转发到后端服务
* 资源服务器之间的中继令牌
* 一个拦截器可以使一个Feign客户端的行为像OAuth2RestTemplate（获取令牌等）
* 在Zuul代理中配置下游认证

### 快速开始

Maven：

```XML
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-security</artifactId>
        <version>1.1.4.BUILD-SNAPSHOT</version>
    </dependency>
</dependencies><repositories>
    <repository>
        <id>spring-snapshots</id>
        <name>Spring Snapshots</name>
        <url>https://repo.spring.io/libs-snapshot</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
```
如果您的应用程序还有Spring Cloud Zuul嵌入式反向代理（使用@EnableZuulProxy），则可以要求它将OAuth2访问令牌转发到其代理的服务器。 因此，上述的SSO应用程序可以简单地增强：

```Java
@SpringBootApplication
@EnableOAuth2Sso
@EnableZuulProxy
class Application {

}
```
并且（除了记录用户并抓取令牌之外）还将将身份验证令牌下传到 /proxy/* 服务。 如果这些服务是用@EnableResourceServer实现的，那么他们将在正确的头中获取一个有效的token。

## [Spring Cloud Data Flow](http://cloud.spring.io/spring-cloud-dataflow)

现代的运行时可组合的微服务应用程序的本地云(cloud-native)编排服务。 易于使用的DSL，拖放式GUI和REST API一起简化了基于微服务的数据管道的整体编排。

### 简介

Spring Cloud Data Flow是一种针对现代的运行时可组合的微服务应用程序的本地云(cloud-native)编排服务。 通过Spring Cloud Data Flow，开发人员可以为数据采集，实时分析和数据导入/导出等常见用例创建和编排数据管道(pipelines)。

Spring Cloud Data Flow是Spring XD的本地云生重新设计，旨在简化Big Data应用程序的开发。 Spring XD的流和批处理模块分别作为基于Spring Boot的流和任务/批处理微服务应用程序进行重构。 这些应用程序现在是自主的部署单元，它们可以“天生地”运行在现代运行时环境，如Cloud Foundry，Apache YARN，Apache Mesos和Kubernetes。

Spring Cloud Data Flow提供了基于微服务的分布式流和任务/批处理数据流水线的一系列模式和最佳实践。

### 功能

* 使用DSL，REST-API，仪表板和拖放GUI - Flo开发
* 独立地进行创建、单元测试、故障排解和管理微服务应用程序
* 使用开箱即用的流和任务/批处理应用程序快速构建数据管道
* 将微服务应用程序用作maven或docker工件(artifacts)
* 缩放数据管道(pipelines)而不中断数据流
* 在各种现代运行时平台上协调数据中心的应用程序，包括Cloud Foundry，Apache YARN，Apache Mesos和Kubernetes
* 利用指标(metrics)，健康检查远程管理每个微服务应用程序

### 快速开始

**Step 1** - 下载Spring Cloud Data Flow的本地服务器和Shellüber-jar：

```shell
wget http://repo.spring.io/release/org/springframework/cloud/spring-cloud-dataflow-server-local/1.1.3.RELEASE/spring-cloud-dataflow-server-local-1.1.3.RELEASE.jar
```

```shell
wget http://repo.spring.io/release/org/springframework/cloud/spring-cloud-dataflow-shell/1.1.3.RELEASE/spring-cloud-dataflow-shell-1.1.3.RELEASE.jar
```
**Step 2** - [下载并启动Kafka 0.10](https://kafka.apache.org/quickstart) [用作消息中间件]

**Step 3** - 启动本地服务器

```shell
java -jar spring-cloud-dataflow-server-local-1.1.3.RELEASE.jar
```
**Step 4** - 在运行本地服务器的同一台机器上启动Shell

```shell
java -jar spring-cloud-dataflow-shell-1.1.3.RELEASE.jar
```
**Step 5** - 从Shell导入提供的应用程序

```shell
dataflow:>app import --uri http://bit.ly/Avogadro-SR1-stream-applications-kafka-10-maven
```
**Step 6** - 从Shell创建'ticktock'流

```shell
dataflow:>stream create ticktock --definition "time | log" --deploy
```

一旦部署了“ticktock”流，您将会注意到输出与本地服务器控制台中类似的内容[参考Step 3]。 例如，日志应用程序的日志将位于：/var/folders/...../ticktock.log目录。

```shell
2016-07-18 22:08:24.777  INFO 73058 --- [nio-9393-exec-9] o.s.c.d.spi.local.LocalAppDeployer       : deploying app ticktock.log instance 0
   Logs will be in /var/folders/c3/ctx7_rns6x30tq7rb76wzqwr0000gp/T/spring-cloud-dataflow-5011521526937452211/ticktock-1468904904769/ticktock.log
2016-07-18 22:08:25.081  INFO 73058 --- [nio-9393-exec-9] o.s.c.d.spi.local.LocalAppDeployer       : deploying app ticktock.time instance 0
   Logs will be in /var/folders/c3/ctx7_rns6x30tq7rb76wzqwr0000gp/T/spring-cloud-dataflow-5011521526937452211/ticktock-1468904905074/ticktock.time
```
**Step 7** - 验证'ticktock'日志

```shell
tail -f /var/folders/ ... /ticktock.log/stdout_0.log
```

**Step 8** - 查看本地服务器的仪表板功能：http://localhost:9393/dashboard

### Spring Cloud Data Flow实现

* Local Server
* Cloud Foundry Server
* Apache YARN Server
* Kubernetes Server
* Apache Mesos Server

关于版本请参考[这里](http://cloud.spring.io/spring-cloud-dataflow/)

### Spring Cloud Data Flow 社区版实现

[Spring Cloud Data Flow for HashiCorp Nomad](https://github.com/donovanmuller/spring-cloud-dataflow-server-nomad)

[Spring Cloud Data Flow for Red Hat OpenShift](https://github.com/donovanmuller/spring-cloud-dataflow-server-openshift)

### Spring Cloud Data Flow 构建模块

Spring Cloud Data Flow建立在多个项目的基础上，生态系统的顶层构建模块以下列可视化表示形式列出。 每个项目都代表着一个核心功能，它们独立发展，独立发布 - 根据链接可以查找有关每个项目的更多细节。

![image](http://blog.geekidentity.com/images/spring_cloud_overview/building_blocks_of_spring_cloud_data_flow.jpg)

## [Spring Cloud Stream](http://cloud.spring.io/spring-cloud-stream)

一个轻量级的事件驱动的微服务框架来快速构建可以连接到外部系统的应用程序。 使用Apache Kafka或RabbitMQ在Spring Boot应用程序之间发送和接收消息的简单声明模型。

### 简介

Spring Cloud Stream是构建消息驱动的微服务的框架。 Spring Cloud Stream建立在Spring Boot上，以创建DevOps友好的微服务应用程序和Spring Integration来提供与消息代理的连接。 Spring Cloud Stream提供了一个自己的消息代理配置，在多个中间件供应商中引入了持久的pub/sub语义，消费者组和分区的概念。 这个自带的配置为创建流处理应用程序提供了基础。

通过向主应用程序添加@EnableBinding，您可以立即连接到消息代理，并通过向方法添加@StreamListener，您将收到流处理事件。

### 快速开始

Maven：

```XML
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream-dependencies</artifactId>
            <version>Brooklyn.SR3</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-stream</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-stream-kafka</artifactId>
    </dependency>
</dependencies>
```
只要Spring Cloud Stream和Spring Cloud Stream binder在类路径中，任何具有@EnableBinding的Spring Boot应用程序将绑定到总线提供的外部代理（例如，Rabbit MQ或Kafka，具体取决于您选择的实现）。 示例：

转到 http://start.spring.io 并创建一个带有“Stream Kafka”依赖项目。 修改主类，如下所示：

```Java
@SpringBootApplication
@EnableBinding(Source.class)
public class StreamdemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(StreamdemoApplication.class, args);
    }

    @Bean
    @InboundChannelAdapter(value = Source.OUTPUT)
    public MessageSource<String> timerMessageSource() {
        return () -> new GenericMessage<>(new SimpleDateFormat().format(new Date()));
    }

}
```
运行应用程序时，确保Kafka正在运行。 您可以使用Kafka提供的kafka-console-consumer.sh 脚本程序来监视在输出主题上发送的消息。

## [Spring Cloud Stream App Starters](http://cloud.spring.io/spring-cloud-stream-app-starters)

Spring Cloud Stream App Starters应用程序启动器是基于Spring Boot的Spring集成应用程序，提供与外部系统的集成。

### 简介

Spring Cloud Stream应用程序启动器是基于Spring Boot的Spring集成应用程序，提供与外部系统的集成。 Spring Cloud Stream应用程序可以与Spring Cloud Data Flow一起使用来创建，部署和编排消息驱动的微服务应用程序。

Spring Cloud Stream Application Starters 是通过消息传递中间件（如Apache Kafka和RabbitMQ）进行通信的独立可执行应用程序。 这些应用程序可以在各种运行时平台上独立运行，包括：Cloud Foundry，Apache Yarn，Apache Mesos，Kubernetes，Docker，甚至您的笔记本电脑。

### 功能

* 作为Spring Boot应用程序独立运行
* 在Spring Cloud Data Flow中将微服务组全成管道流
* 将微服务应用程序用作maven或docker artifacts
* 通过命令行，环境变量或YAML文件覆盖配置参数
* 提供基础设施来隔离测试应用程序
* 下载版本[Spring Initializr](https://start-scs.cfapps.io/)作为启动器

### 可用应用程序

Source | Processor | Sink(水槽)
---|---|---
file | aggregator | aggregate-counter
ftp | bridge | cassandra
gemfire | filter | counter
gemfire-cq | groovy-filter | field-value-counter
http | groovy-transform | file
jdbc | httpclient | ftp
jms | pmml | gemfire
load-generator | scriptable-transform | gpfdist
loggregator | splitter | hdfs
mail | tasklaunchrequest-transform | hdfs-dataset
mongodb | tcp-client | jdbc
rabbit | transform | log
s3 | | mongodb
sftp | | pgcopy
syslog | |rabbit
tcp | |redis-pubsub
tcp-client | | router
time | | s3
trigger | | sftp
triggertask | | task-launcher-cloudfoundry
twitterstream | | task-launcher-local
| | task-launcher-yarn
| | tcp
| | throughput
| | websocket

### 快速开始

**Step 1** - [从这里下载](http://repo.spring.io/libs-release/org/springframework/cloud/stream/app/time-source-kafka-10)最新的基于Kafka 10的*Time Source*应用程序 [eg: /1.1.1.RELEASE/time-source-kafka-10-1.1.1.RELEASE.jar]

**Step 2** - [从这里下载](http://repo.spring.io/libs-release/org/springframework/cloud/stream/app/log-sink-kafka-10)最新的基于Kafka 10的*log-sink*应用程序 [eg: /1.1.1.RELEASE/log-sink-kafka-10-1.1.1.RELEASE.jar]

**Step 3** - 启动 Kafka 0.10.1.0

**Step 4** - 运行**Time Source**并绑定到ticktock 主题

```shell
java -jar time-source-kafka-***.jar --spring.cloud.stream.bindings.output.destination=ticktock
```

**Step 5** - 运行 **Log Sink** 并绑定到 ticktock Topic

```shell
java -jar log-sink-kafka-***.jar --spring.cloud.stream.bindings.input.destination=ticktock
```
**Step 6** - 在控制台验证Tickets Logs

### Stream App Starters and Spring Cloud Data Flow (**)

见[官网](http://cloud.spring.io/spring-cloud-stream-app-starters/)最后

## [Spring Cloud Task](http://cloud.spring.io/spring-cloud-task)

一种短期的微服务框架，用于快速构建执行有限数据处理的应用程序。 简单的声明，将功能和非功能特性添加到Spring Boot应用程序。

### 简介

Spring Cloud Task允许用户使用Spring Cloud开发和运行短期的微服务器，并在云端运行，甚至在Spring Cloud Data Flow中运行。 只需添加@EnableTask并运行您的应用程序作为一个Spring Boot应用程序（单个应用程序上下文）。

### 快速开始

Maven：

```XML
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-task-starter</artifactId>
        <version>1.1.2.RELEASE</version>
    </dependency>
</dependencies>
```
只要Spring Cloud Task在类路径中，任何具有@EnableTask的Spring Boot应用程序将记录引导应用程序的开始和结束以及配置的任务存储库中的任何未捕获的异常。 示例：

```Java
@SpringBootApplication
@EnableTask
public class ExampleApplication {

	@Bean
	public CommandLineRunner commandLineRunner() {
		return strings ->
				System.out.println("Executed at :" + 
				      new SimpleDateFormat().format(new Date()));
	}

	public static void main(String[] args) {
		SpringApplication.run(ExampleApplication.class, args);
	}
}
```

## [Spring Cloud Task App Starters](http://cloud.spring.io/spring-cloud-task-app-starters)

Spring Cloud Task 应用程序启动器是Spring Boot 应用程序，可能是任何进程，包括不会永久运行的Spring Batch作业，并且在有限的数据处理时间结束/停止。

### 简介

Spring Cloud任务应用程序启动器是Spring Boot应用程序，可能是任何进程，包括不会永久运行的Spring Batch作业，并且在某些时候结束/停止。 Spring Cloud Task应用程序可以与Spring Cloud Data Flow一起使用来创建，部署和编排短期数据微服务器。

Spring Cloud任务应用程序启动器是独立的可执行应用程序，可用于按需使用情况，如数据库迁移，机器学习和计划操作。 这些应用程序可以在各种运行时平台上独立运行，包括：Cloud Foundry，Apache Yarn，Apache Mesos，Kubernetes，Docker，甚至您的笔记本电脑。

### 功能

* 作为Spring Boot应用程序独立运行
* 协调为短时数据微服务
* 将数据微服务应用程序用作maven或docker artifacts
* 通过命令行，环境变量或YAML文件覆盖配置参数
* 提供基础设施来隔离测试应用程序
* 下载此版本的[Spring Initializr](https://start-scs.cfapps.io/)作为启动器

### 可用应用程序

* spark-client
* spark-cluster
* spark-yarn
* timestamp

### 快速开始

**Step 1** - [这里下载](http://repo.spring.io/libs-release/org/springframework/cloud/task/app/timestamp-task/)最新的*timestamp*应用程序 [eg: /1.1.0.RELEASE/timestamp-task-1.1.0.RELEASE.jar]

**Step 2** - 运行*timestamp*程序

```shell
java -jar timestamp-task-***.jar
```

**Step 3** - 验证控制台中的*timestamp*日志

**Step 4** - 验证*timestamp*应用程序关闭

### Task App Starters and Spring Cloud Data Flow (**)

[官网最下面](http://cloud.spring.io/spring-cloud-task-app-starters/)

## [Spring Cloud Zookeeper](http://cloud.spring.io/spring-cloud-zookeeper)

使用Apache Zookeeper进行服务发现和配置管理。

### 简介

Spring Cloud Zookeeper通过自动配置和绑定到Spring Environment和其他Spring编程模型语法来为Spring Boot应用程序提供Apache Zookeeper集成。 通过几个简单的注释，您可以快速启用和配置应用程序中的常见模式，并使用Zookeeper构建大型分布式系统。 提供的模式包括服务发现和分布式配置。

### 功能

* 服务发现：可以向Zookeeper注册实例，客户端可以使用Spring管理的bean来发现实例
* 通过Spring Cloud Netflix支持客户端负载均衡器Ribbon
* 通过Spring Cloud Netflix支持Zuul，动态路由器和过滤器
* 分布式配置：使用Zookeeper作为数据存储

### 快速开始

Maven：

```XML
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-zookeeper-dependencies</artifactId>
            <version>1.0.3.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-zookeeper-discovery</artifactId>
    </dependency>
</dependencies>
```
只要Spring Cloud Zookeeper，Apache Curator和Zookeeper Java Client都在类路径中，任何具有@EnableDiscoveryClient的Spring Boot应用程序将尝试连接 http://localhost:2181 （zookeeper.connectString的默认值）上的Zookeeper代理。

```Java
@Configuration
@EnableAutoConfiguration
@EnableDiscoveryClient
@RestController
public class Application {

  @RequestMapping("/")
  public String home() {
    return "Hello World";
  }

  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }

}
```

## [Spring Cloud for Amazon Web Services](https://cloud.spring.io/spring-cloud-aws)

### 简介

与托管的Amazon Web Services轻松集成。 它提供了一种方便的方式，使用众所周知的Spring 常用语法和API（如消息传递或缓存API）与AWS提供的服务进行交互。 开发人员可以围绕托管服务构建应用程序，而无需关心基础设施或维护。

### 功能

* SQS的Spring Messaging API实现。
* ElastiCache的Spring Cache API实现。
* SNS端点（HTTP）的基于注解的映射。
* 通过它们在CloudFormation栈中定义的逻辑名称访问资源。
* 基于RDS实例的逻辑名称创建自动JDBC DataSource。
* 用于S3桶ResourceLoader的[Ant风格路径匹配](http://blog.geekidentity.com/spring/ant_style_path_patterns/)。

**Annotation-based SQS Queue Listener**

```Java
@MessageMapping("logicalQueueName")
private void receiveMessage(Person person, @Header("SenderId") String senderId) {
    // ...
}
```
**Annotation-based SNS Listener**

```Java
@Controller
@RequestMapping("/sns/receive")
public class SnsEndpointController {

@NotificationMessageMapping
public void receiveNotification(@NotificationMessage String message, @NotificationSubject String subject) {
    // ...
}

@NotificationSubscriptionMapping
public void confirmSubscription(NotificationStatus notificationStatus) {
    notificationStatus.confirmSubscription();
}
```
**Messaging Templates**

```Java
snsTemplate.sendNotification("SnsTopic", "message", "subject");
sqsTemplate.convertAndSend("Queue", new Person("John", "Doe"));
```

### 快速开始

Maven：

```XML
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-aws</artifactId>
            <version>1.2.0.BUILD-SNAPSHOT</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-aws</artifactId>
    </dependency>
</dependencies><repositories>
    <repository>
        <id>spring-snapshots</id>
        <name>Spring Snapshots</name>
        <url>https://repo.spring.io/libs-snapshot</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
```

```XML
<!-- Define global credentials for all the AWS clients -->
<aws-context:context-credentials>
    <aws-context:instance-profile-credentials/>
    <aws-context:simple-credentials access-key="${accessKey:}"
                                    secret-key="${secretKey:}"/>
</aws-context:context-credentials>

<!-- Define global region -->
<aws-context:context-region region="EU_WEST_1"/>

<!-- Cloud Formation Stack -->
<aws-context:stack-configuration stack-name="StackName"/>
```

## [Spring Cloud Connectors](https://cloud.spring.io/spring-cloud-connectors/)

使PaaS应用程序在各种平台中轻松连接到后端服务，如数据库和消息代理（以前称为“Spring Cloud”）。

### 简介

Spring Cloud连接器简化了连接到服务的过程，并在Cloud Foundry和Heroku等云平台中获得了运行环境支持，特别是对于Spring应用程序。 它是为扩展性而设计的：您可以使用所提供的云连接器之一或为云平台编写一个连接器，您可以使用内置的常用服务支持（关系数据库，MongoDB，Redis，RabbitMQ）或扩展Spring 云连接器与您自己的服务一起工作。

### 功能

Spring Cloud连接器专注于为典型用例提供良好的开箱即用体验，并提供可扩展性机制来覆盖其他用户。

* 适用于Spring应用程序的Java和XML配置：为绑定到应用程序的服务创建bean的简单方法。
* 云平台可扩展性：云连接器的概念，允许您将Spring Cloud连接器的功能扩展到其他云平台。
* 服务信息和连接器可扩展性：您自己的Spring Cloud连接器扩展可以通过从应用程序操作环境提取服务连接信息将应用程序连接到任何服务，并且（可选）可以将信息转换为服务连接器。

### 组成项目

* [Spring Cloud Connectors Core](http://cloud.spring.io/spring-cloud-connectors/spring-cloud-connectors.html#_spring_cloud_connectors_core)：核心类库。 Cloud-agnostic，不依赖于Spring; 为选择以编程方式访问云服务和应用程序信息的应用程序开发人员提供了入门点，并为贡献cloud 连接器和服务连接器创建者提供了扩展机制。
* [Spring Cloud Service Connector](http://cloud.spring.io/spring-cloud-connectors/spring-cloud-spring-service-connector.html)：提供来自Spring Data项目的javax.sql.DataSource和各种连接工厂的服务连接器创建者的库。
* [Cloud Foundry Connector](http://cloud.spring.io/spring-cloud-connectors/spring-cloud-cloud-foundry-connector.html)：Cloud Foundry cloud连接器。
* [Heroku Connector](http://cloud.spring.io/spring-cloud-connectors/spring-cloud-heroku-connector.html)：Heroku的cloud连接器。

### 快速开始

Maven：

```XML
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-spring-service-connector</artifactId>
        <version>1.2.3.RELEASE</version>
    </dependency>
    <!-- If you intend to deploy the app on Cloud Foundry, add the following -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-cloudfoundry-connector</artifactId>
        <version>1.2.3.RELEASE</version>
    </dependency>
    <!-- If you intend to deploy the app on Heroku, add the following -->
    <!-- It is okay to add more than one cloud connector; the right one will
         be picked during runtime -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-heroku-connector</artifactId>
        <version>1.2.3.RELEASE</version>
    </dependency>
</dependencies>
```

然后，如果您正在使用Spring应用程序，请按照[Spring Cloud Spring Service Connector](http://cloud.spring.io/spring-cloud-connectors/spring-cloud-spring-service-connector.html)的说明进行操作。 如果您不使用Spring，请查看[Spring Cloud Connectors Core](http://cloud.spring.io/spring-cloud-connectors/spring-cloud-connectors.html#_spring_cloud_connectors_core)。

### 社区扩展

* Spring Cloud Connectors for SAP HANA Cloud Platform (HCP)
* Spring Cloud Connectors for IBM Bluemix
* Spring Cloud Connectors for Spring Cloud Services on Pivotal Cloud Foundry
* Spring Cloud Connectors for Pivotal Gemfire
* Spring Cloud Connectors for Amazon S3

## [Spring Cloud Starters](https://github.com/spring-cloud/spring-cloud-starters)

Spring Boot-style启动项目，以缓解Spring Cloud消费者的依赖管理。 （Angel.SR2之后停止作为项目，并与其他项目合并）。

## [Spring Cloud CLI](https://github.com/spring-cloud/spring-cloud-cli)

Spring Boot CLI插件用于在Groovy中快速创建Spring Cloud组件应用程序


既然看到最后了，一起来搞基吧：264133057