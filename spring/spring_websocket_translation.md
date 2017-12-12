---
categories: spring

tags: 
  - Spring
  - Java
  - WebSocket


title: Spring WebSocket支持（翻译）

date: 2017-03-29
---

## 前言

如果建立实时性非常高的应用，我们可以使用WebSocket，Spring Framework 实现了WebSocket 应用级封装，本文是对Spring Framework WebSocket部分的翻译，如果对WebSocket不太了解可以看看我在16年与老师合著出版的书《[Web异步与实时交互 iframe AJAX WebSocket开发实战](https://item.jd.com/11976922.html)》中我负责的WebSocket部分。
![Web异步与实时交互 iframe AJAX WebSocket开发实战](http://blog.geekidentity.com/images/spring_websocket_translation/iframe_ajax_websocket_action.jpg)

## 26. Spring WebSocket支持

文档的这一部分涵盖了Spring框架对Web应用程序中WebSocket风格消息传递的支持，包括使用STOMP作为应用程序级WebSocket子协议。

* “介绍”确立了思考WebSocket的一个思路，涵盖了使用WebSocket的挑战，设计考虑以及什么场景适合使用WebSocket的想法。
* “WebSocket API”回顾了服务器端的Spring WebSocket API。
* “SockJS Fallback Options”解释了SockJS协议，并展示了如何配置和使用它。
* 
* 
    * “Overview of STOMP”介绍STOMP消息协议。
    * “Enable STOMP over WebSocket”演示了如何在Spring中配置STOMP支持。
    * “Annotation Message Handling”和以下部分介绍了如何编写注释消息处理方法，发送消息，选择消息代理选项，以及与特定“用户”进行消息传输。
    * “Testing Annotated Controller Methods”列出了测试STOMP / WebSocket应用程序的三种方法。

## 26.1 介绍

WebSocket协议[RFC 6455](https://tools.ietf.org/html/rfc6455)为Web应用程序定义了一个重要的新功能：客户端和服务器之间的**全双工，双向**通信。 这是一个令人兴奋的新功能，在漫长的技术历史上，使Web更具交互性，包括Java Applet，XMLHttpRequest，Adobe Flash，ActiveXObject，各种Comet技术，服务器发送的事件等。

对WebSocket协议的完整的介绍超出了本文档的范围。 但是，至少要了解，在WebSocket建立连接时，HTTP仅用于初始握手，这依赖于内置于HTTP中的机制来请求协议升级（或在这种情况下为协议开关），如果服务器同意，它可以使用HTTP状态101对其进行响应 （切换协议）。 假设握手成功，HTTP升级请求后其底层的TCP套接字保持打开，这样客户端和服务器都可以使用这个套接字来相互发送消息。

Spring Framework 4包括一个全新的WebSocket支持的spring-websocket模块。 它与Java WebSocket API标准（[JSR-356](https://jcp.org/en/jsr/detail?id=356)）兼容，并且还提供额外的功能，如在其余介绍中所述。

### 26.1.1 WebSocket 后备方案

使用WebSocket的一个重要挑战是在某些浏览器中缺少对WebSocket的支持。 值得注意的是，支持WebSocket的第一个Internet Explorer版本是版本10（请参阅http://caniuse.com/websockets可查看浏览器版本是否支持WebSocket）。 此外，一些限制性代理可以配置为阻止尝试执行HTTP升级或者在一段时间后断开连接，因为它已经保持打开太久。 Peter Lubbers在InfoQ的文章["How HTML5 Web Sockets Interact With Proxy Servers"](http://www.infoq.com/articles/Web-Sockets-Proxy-Servers) 中提供了对此主题的一个很好的概述。

因此，为了构建一个WebSocket应用程序，需要后备方案才能在浏览器不支持WebSocket时模拟WebSocket API。 Spring Framework提供了基于[SockJS协议](https://github.com/sockjs/sockjs-protocol)的透明后备方案。 这些方案可以通过配置启用，不需要修改应用程序。

### 26.1.2 消息架构

除了时间上的挑战之外，使用WebSocket可以提出重要的设计注意事项，这对于早期的认识至关重要，特别是与我们今天建立Web应用程序相关的知识。

今天使用REST架构构建Web应用程序已经是被广泛接受，理解和支持的。 这是一种依赖于许多URL（名词），少数HTTP方法（动词）以及诸如使用超媒体（链接），remaining stateless 等原则的架构。

相比之下，WebSocket应用程序可以仅使用单个URL进行初始化HTTP握手。 此后，所有消息共享并在相同的TCP连接上传输。 这是一个完全不同的，异步的，事件驱动的消息架构。 更接近于传统消息传递应用（例如JMS，AMQP）。

Spring Framework 4 包括一个新的spring-messaging 模块，其中包含Spring Integration项目的关键抽象，如Message，MessageChannel，MessageHandler等，可以作为这样的消息架构的基础。 该模块还包括一组用于将消息映射到方法的注释，类似于基于Spring MVC注释的编程模型。

### 26.1.3 WebSocket中的子协议支持

WebSocket确实是一个消息架构，但它并不要求使用任何特定的消息协议。 WebSocket对TCP层做了一个非常薄封装，将字节流转换为消息流（文本或二进制），没有其他更多东西的支持。 由应用程序来解释消息的含义。

不同于HTTP（它是应用程序级协议），在WebSocket协议中，框架或容器的传入消息中没有足够的信息来知道如何路由或处理它。 因此，WebSocket可以说是级别很低，只是一个非常简单的应用程序。但我们可以在WebSocket顶部创建一个框架。 这与目前使用Web框架而不是单独使用Servlet API的大多数Web应用程序相当。

为此，WebSocket RFC定义了[子协议](https://tools.ietf.org/html/rfc6455#section-1.9)的使用。 在握手期间，客户端和服务器可以使用头部Sec-WebSocket-Protocol 来同意子协议，即使用更高的应用级协议。 子协议不是必须使用的，即使不使用子协议，应用程序仍然需要选择客户端和服务器都可以理解的消息格式。 该格式可以是自定义，框架特定或标准消息传递协议。

Spring Framework提供对使用[STOMP](https://stomp.github.io/stomp-specification-1.2.html#Abstract)的支持 - STOMP是一种简单的消息传递协议，最初创建用于受HTTP启发的脚本语言。 STOMP被广泛支持，非常适合在WebSocket和Web上使用。

### 26.1.4 我应该使用WebSocket吗？

有关使用WebSocket的所有设计考虑中，“什么时候使用？”是合理的。

WebSocket最适合在Web应用程序中，客户端和服务器需要以高频率和低延迟交换事件。 首要候选包括但不限于在金融，游戏，合作等方面的应用。 这种应用对时间延迟非常敏感，并且还需要以高频率交换各种各样的消息。

即使在延迟至关重要的情况下，如果消息量相对较低（例如监控网络故障），则长轮询的使用应被视为一种相对简单的替代方案，其工作可靠，并且在效率方面是可比较的 （假设消息量相对较低）。

低延迟和高频率的消息可以使WebSocket协议的使用成为关键。 即使在这样的应用程序中，选择仍然是所有客户端 - 服务器通信是否应该通过WebSocket消息完成，而不是使用HTTP和REST。 答案将因应用而异; 然而，有可能某些功能可以通过WebSocket和REST API来暴露，以便为客户提供替代方案。 此外，REST API调用可能需要向通过WebSocket连接的感兴趣的客户端广播消息。

Spring Framework允许@Controller和@RestController类具有HTTP请求处理和WebSocket消息处理方法。 此外，Spring MVC请求处理方法或任何应用方法可以轻松地向所有感兴趣的WebSocket客户端或特定用户广播消息。

## 26.2 WebSocket API

Spring Framework提供了一个适用于各种WebSocket引擎的WebSocket API。目前，该列表包括WebSocket Runtime环境，如Tomcat 7.0.47+，Jetty 9.1+，GlassFish 4.1+，WebLogic 12.1.3+和Undertow 1.0+（和WildFly 8.0+）。随着更多的WebSocket Runtime可用，可能会添加额外的支持 

>如26.1.3 WebSocket中的子协议支持中所述，直接使用WebSocket API的对于应用程序而言是级别太低了 -除非对消息的格式做出假设，这样框架可以通过注释来解释消息或路由它们。这就是为什么应用程序应该考虑使用一个子协议和Spring的WebSocket STOMP支持。当使用更高级别的协议时，WebSocket API的细节变得不那么相关，就像TCP通信的细节在使用HTTP时不会暴露给应用程序一样。 不过，本节将介绍直接使用WebSocket的详细信息。
    
### 26.2.1 创建并配置WebSocketHandler

创建WebSocket服务器与实现WebSocketHandler一样简单，或者更寻址方式去扩展类TextWebSocketHandler或BinaryWebSocketHandler

```Java
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.handler.TextWebSocketHandler;
import org.springframework.web.socket.TextMessage;

public class MyHandler extends TextWebSocketHandler {

	@Override
	public void handleTextMessage(WebSocketSession session, TextMessage message) {
		// ...
	}

}
```
有专门的WebSocket Java-config和XML命名空间支持将上述WebSocket处理程序映射到特定的URL：

```Java
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myHandler(), "/myHandler");
    }

    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }

}
```
XML的等效配置：

```XML
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        http://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers>
        <websocket:mapping path="/myHandler" handler="myHandler"/>
    </websocket:handlers>

    <bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```
以上配置是用于Spring MVC应用程序中的，应该包含在[DispatcherServlet](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-servlet)的配置中。但是Spring的WebSocket支持不依赖于Spring MVC。在WebSocketHttpRequestHandler的帮助下，将WebSocketHandler集成到其他HTTP服务环境中相对比较简单的。

### 26.2.2 自定义WebSocket握手

自定义初始HTTP WebSocket握手请求的最简单的方法是通过HandshakeInterceptor，它将暴露出握手方法的“before”和“after”。这样的拦截器可以用于阻止握手或向WebSocketSession中添加属性。 例如，有一个内置拦截器将HTTP会话属性传递给WebSocket会话：

```Java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(new MyHandler(), "/myHandler")
            .addInterceptors(new HttpSessionHandshakeInterceptor());
    }

}
```
XML的等效配置：

```XML
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        http://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers>
        <websocket:mapping path="/myHandler" handler="myHandler"/>
        <websocket:handshake-interceptors>
            <bean class="org.springframework.web.socket.server.support.HttpSessionHandshakeInterceptor"/>
        </websocket:handshake-interceptors>
    </websocket:handlers>

    <bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```
通过扩展执行WebSocket握手步骤的DefaultHandshakeHandler，还可以做一些更高级功能，包括验证客户端，协商子协议等。如果需要可以配置自定义RequestUpgradeStrategy以适应不支持的WebSocket服务器引擎和版本（有关此主题的更多信息，请参见[第26.2.4节“部署注意事项”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#websocket-server-deployment)）。Java-config和XML命名空间都可以配置自定义HandshakeHandler。

### 26.2.3 WebSocketHandler装饰器

Spring提供了一个WebSocketHandlerDecorator基类，可用于使用附加行为来装饰WebSocketHandler。在使用WebSocket Java-config或XML命名空间时，默认情况下提供并添加了日志记录和异常处理实现。ExceptionWebSocketHandlerDecorator捕获从任何WebSocketHandler方法引发的所有未捕获的异常，并关闭WebSocket Session，状态置为1011，表示服务器错误。

### 26.2.4 部署注意事项

Spring WebSocket API很容易集成到Spring MVC应用程序中，其中DispatcherServlet用于HTTP WebSocket握手以及其他HTTP请求。通过调用WebSocketHttpRequestHandler也很容易集成到其他HTTP处理场景中。 这样很方便而且容易理解。 但是，关于JSR-356运行时可以做一些特殊的考虑。

Java WebSocket API（JSR-356）提供了两个部署机制。 第一个涉及启动时的Servlet容器类路径扫描（Servlet 3功能）; 另一个是在Servlet容器初始化时使用的注册API。 这些机制都不可能对所有HTTP处理（包括WebSocket握手和所有其他HTTP请求）使用单个“前端控制器(front controller)，例如Spring MVC的DispatcherServlet。

即使在JSR-356 Runtimes 运行，通过提供特定于服务器的RequestUpgradeStrategy，Spring的WebSocket支持的JSR-356的一个重大限制(significant limitation )。

>已经创建了克服Java WebSocket API中上述限制的请求，并可在[WEBSOCKET_SPEC-211](https://java.net/jira/browse/WEBSOCKET_SPEC-211)中执行。
还要注意的是，Tomcat和Jetty已经提供了本地API替代方案，可以轻松克服这个限制。 我们希望更多的服务器将遵循他们的例子，无论Java WebSocket API何时被解决。

第二个考虑因素是具有JSR-356支持的Servlet容器预计将执行ServletContainerInitializer（SCI）扫描，这可能会减慢应用程序启动速度，在某些情况下启动速度会显著降低。 如果在升级到支持JSR-356的Servlet容器版本之后观察到对启动速度有重大影响，则可以通过使用web.xml中的<absolute-ordering />元素来选择性地启用或禁用Web fragments （和SCI扫描）：


```XML
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://java.sun.com/xml/ns/javaee
        http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

    <absolute-ordering/>

</web-app>
```
如果需要，您可以通过名称有选择地启用Web fragments 以提供对Servlet 3 Java初始化API的支持 ，例如Spring自己的SpringServletContainerInitializer：

```XML
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://java.sun.com/xml/ns/javaee
        http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

    <absolute-ordering>
        <name>spring_web</name>
    </absolute-ordering>

</web-app>
```
### 26.2.5 配置WebSocket引擎

每个底层WebSocket引擎都会公开控制runtime 的配置属性，例如消息缓冲区大小，空闲超时等。

对于Tomcat，WildFly和GlassFish，在您的WebSocket Java配置中添加一个ServletServerContainerFactoryBean：

```Java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Bean
    public ServletServerContainerFactoryBean createWebSocketContainer() {
        ServletServerContainerFactoryBean container = new ServletServerContainerFactoryBean();
        container.setMaxTextMessageBufferSize(8192);
        container.setMaxBinaryMessageBufferSize(8192);
        return container;
    }

}
```
或WebSocket XML命名空间：

```XML
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        http://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <bean class="org.springframework...ServletServerContainerFactoryBean">
        <property name="maxTextMessageBufferSize" value="8192"/>
        <property name="maxBinaryMessageBufferSize" value="8192"/>
    </bean>

</beans>
```

>对于客户端WebSocket配置，您应该使用WebSocketContainerFactoryBean(XML配置)或ContainerProvider.getWebSocketContainer() （Java配置）。

对于Jetty，您需要提供一个预配置的Jetty WebSocketServerFactory，并通过WebSocket Java配置将其插入到Spring的DefaultHandshakeHandler中：

```Java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(echoWebSocketHandler(),
            "/echo").setHandshakeHandler(handshakeHandler());
    }

    @Bean
    public DefaultHandshakeHandler handshakeHandler() {

        WebSocketPolicy policy = new WebSocketPolicy(WebSocketBehavior.SERVER);
        policy.setInputBufferSize(8192);
        policy.setIdleTimeout(600000);

        return new DefaultHandshakeHandler(
                new JettyRequestUpgradeStrategy(new WebSocketServerFactory(policy)));
    }

}
```
或WebSocket XML命名空间：

```XML
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        http://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers>
        <websocket:mapping path="/echo" handler="echoHandler"/>
        <websocket:handshake-handler ref="handshakeHandler"/>
    </websocket:handlers>

    <bean id="handshakeHandler" class="org.springframework...DefaultHandshakeHandler">
        <constructor-arg ref="upgradeStrategy"/>
    </bean>

    <bean id="upgradeStrategy" class="org.springframework...JettyRequestUpgradeStrategy">
        <constructor-arg ref="serverFactory"/>
    </bean>

    <bean id="serverFactory" class="org.eclipse.jetty...WebSocketServerFactory">
        <constructor-arg>
            <bean class="org.eclipse.jetty...WebSocketPolicy">
                <constructor-arg value="SERVER"/>
                <property name="inputBufferSize" value="8092"/>
                <property name="idleTimeout" value="600000"/>
            </bean>
        </constructor-arg>
    </bean>

</beans>
```

### 26.2.6 配置跨域

从Spring Framework 4.1.5开始，WebSocket和SockJS的默认行为是仅接受相同的源请求。 也可以允许所有或指定的Origin列表。 此检查主要是为浏览器客户端设计的。 我们不能阻止其他类型的客户端修改Origin标头值（有关详细信息，请参阅[RFC 6454：Web Origin概念](https://tools.ietf.org/html/rfc6454)）。

3种可能的行为是：

* 只允许相同的origin请求（默认）：在此模式下，当启用SockJS时，Iframe HTTP响应头X-Frame-Options设置为SAMEORIGIN，并且禁用JSONP传输，因为它不允许检查请求的来源 。 因此，启用此模式时，不支持IE6和IE7。
* 允许指定的起始列表：每个提供的允许origin必须以http://或https://开头。 在此模式下，当启用SockJS时，基于IFrame和JSONP的传输均被禁用。 因此，启用此模式时，不支持IE6至IE9。
* 允许所有来源：启用此模式，您应该提供*作为允许的origin值。 在这种模式下，所有的传输都可用。

WebSocket和SockJS允许的origin可以像如下进行配置：

```Java
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myHandler(), "/myHandler").setAllowedOrigins("http://mydomain.com");
    }

    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }

}
```
XML的等效配置：

```XML
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        http://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers allowed-origins="http://mydomain.com">
        <websocket:mapping path="/myHandler" handler="myHandler" />
    </websocket:handlers>

    <bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```
## 26.3 SockJS后备方案

如26.1.1 WebSocket 后备方案中所解释的，WebSocket并不是在所有的浏览器中都支持，而且还有可能会被限制性网络代理排除。 这就是为什么Spring提供了基于[SockJS协议](https://github.com/sockjs/sockjs-protocol)（0.3.3版）尽可能接近的WebSocket API的备用方案。

### 26.3.1 SockJS概述

SockJS的目标是让应用程序使用WebSocket API，但在运行时如果需要可以回到非WebSocket的替代方案，即无需更改应用程序代码。

SockJS 包括：

* [SockJS 协议](https://github.com/sockjs/sockjs-protocol)以可执行的 [narrated tests](https://sockjs.github.io/sockjs-protocol/sockjs-protocol-0.3.3.html) 的形式定义。
* [SockJS JavaScript客户端](https://github.com/sockjs/sockjs-client/) - 浏览器的客户端库。
* SockJS服务器实现，在Spring Framework spring-websocket模块中。
* 截至4.1 spring-websocket还提供了一个SockJS Java客户端。

SockJS 专为浏览器而设计。 它使用各种技术来支持广泛的浏览器版本。 有关 SockJS 传输类型和浏览器的完整列表，请参阅 SockJS 客户端页面。 传输分为三大类：WebSocket，HTTP Streaming 和 HTTP 长轮询。 有关这些类别的概述，请参阅[此博客](https://spring.io/blog/2012/05/08/spring-mvc-3-2-preview-techniques-for-real-time-updates/)。

SockJS 客户端通过发送“GET / info”开始从服务器获取基本信息。 之后，它必须决定使用什么传输工具。 如果可能则使用 WebSocket。 如果没有，在大多数浏览器中至少有一个 HTTP 流选项，如果不是，则使用 HTTP（长）轮询。

所有传输请求都具有以下URL结构：

```
http://host:port/myApp/myEndpoint/{server-id}/{session-id}/{transport}
```
* {server-id} - 对集群中的路由请求有用，其他情况没有用。
* {session-id} - 关联属于SockJS会话的HTTP请求。
* {transport} - 表示传输类型，例如： “websocket”，“xhr-streaming”等

WebSocket传输只需要一个HTTP请求来执行WebSocket握手。 之后的所有消息都在该套接字上传输。

HTTP传输需要更多的请求。 例如，Ajax/XHR流依赖于一个长时间运行的服务器到客户端消息的请求以及客户端到服务器消息的其他HTTP POST请求。 长轮询类似，除了在每个服务器到客户端发送之后结束当前请求。

SockJS增加了最小的消息框架。 例如，服务器最初发送字母o（"open"帧），消息作为[“message1”，“message2”]（JSON编码数组）发送，字母h（“heartbeat”帧）默认25秒内如果没有消息流 ，发送字母c（“close”框）关闭会话。

要了解更多信息，请在浏览器中运行示例并观察HTTP请求。 SockJS客户端允许修改传输列表，以便可以一次查看每个传输。 SockJS客户端还提供了一个调试标志，可在浏览器控制台中输出有用的消息。 在服务器端启用org.springframework.web.socket的TRACE日志记录。 更详细的参考SockJS协议 [narrated test](https://sockjs.github.io/sockjs-protocol/sockjs-protocol-0.3.3.html)。

### 26.3.2 启用SockJS

SockJS通过Java配置启用：

```Java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myHandler(), "/myHandler").withSockJS();
    }

    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }

}
```
XML等效配置：

```XML
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        http://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers>
        <websocket:mapping path="/myHandler" handler="myHandler"/>
        <websocket:sockjs/>
    </websocket:handlers>

    <bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```
以上是用于Spring MVC应用程序，应该包含在 [DispatcherServlet](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-servlet) 的配置中。 但是Spring的WebSocket和SockJS支持不依赖于Spring MVC。 在 [SockJsHttpRequestHandler](http://docs.spring.io/spring-framework/docs/4.3.7.RELEASE/javadoc-api/org/springframework/web/socket/sockjs/support/SockJsHttpRequestHandler.html) 的帮助下将它集成到其他HTTP服务环境中比较简单。

在浏览器端，应用程序可以使用模拟W3C WebSocket API的 [sockjs-client](https://github.com/sockjs/sockjs-client/)（版本1.0.x），并与服务器通信，以根据运行的浏览器选择最佳的传输选项。查看 [sockjs-client](https://github.com/sockjs/sockjs-client/) 页面 浏览器支持的传输类型列表。 客户端还提供了几个配置选项，例如，指定要包含哪种传输。

### 26.3.3 IE 8, 9 中HTTP Streaming Ajax/XHR vs IFrame

万恶的IE 8和9在未来一段时间内还会有不少人使用。 他们是SockJS产生的主要原因。 本节介绍在这些(万恶的IE)浏览器中运行的重要注意事项。

SockJS客户端通过Microsoft的 [XDomainRequest](https://blogs.msdn.com/b/ieinternals/archive/2010/05/13/xdomainrequest-restrictions-limitations-and-workarounds.aspx) 支持IE 8和9中的Ajax/XHR流式传输。这可以跨域工作，但不支持发送Cookie。Cookie对Java应用程序来说至关重要。但是，由于SockJS客户端可以与许多服务器类型（不仅仅是Java类型）一起使用，所以需要知道cookie是否重要。 如果是这样，SockJS客户端更倾向于Ajax/XHR进行流式传输，否则依赖于基于iframe的技术。

来自SockJS客户端的第一个"/info"请求将会影响客户端选择传输信息类型。其中一个细节是服务器应用程序是否依赖于Cookie，例如 用于认证目的或使用粘性会话进行聚类。Spring的SockJS支持包括一个名为sessionCookieNeeded的属性。它默认启用，因为大多数Java应用程序依赖于JSESSIONID cookie。如果您的应用程序不需要它，您可以关闭此选项，SockJS客户端一般在IE 8和9中会选择 xdr-streaming 进行传输。

如果您使用基于iframe的传输，通过将HTTP响应头X-Frame-Options设置为DENY，SAMEORIGIN，可以指示浏览器阻止在指定页面上使用IFrames ，或ALLOW-FROM <origin>。 这用于防止点击劫持(clickjacking)。

> Spring Security 3.2+支持在每个响应中设置X-Frame-Options。 默认情况下，Spring Security Java配置将其设置为DENY。在3.2中，Spring Security XML 中并默认并没有配置X-Frame-Options，但可以自己配置并作为默认值。
见 [第7.1节 "Default Security Headers"](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#headers)。
有关如何配置X-Frame-Options标题的设置的详细信息，请参阅Spring Security文档的“默认安全性头文件”。 您还可以查看或观看 [SEC-2501](https://jira.spring.io/browse/SEC-2501) 的额外背景。

如果您的应用程序添加 X-Frame-Options 响应头（应该是！），并且依赖于基于iframe的传输，则需要将头值设置为 SAMEORIGIN 或 ALLOW-FROM <origin>。 除此之外，Spring SockJS 还需要知道SockJS客户端的位置，因为它是从iframe加载的。默认情况下，iframe设置为从CDN位置下载SockJS客户端。最好是将此选项配置为与应用程序相同来源的URL。

在Java中可以如下所示完成配置。 XML 中通过<websocket:sockjs>元素提供了类似的选项：

```Java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/portfolio").withSockJS()
                .setClientLibraryUrl("http://localhost:8080/myapp/js/sockjs-client.js");
    }

    // ...

}
```
>在初始开发过程中，请启用SockJS客户端开发模式，以防止浏览器缓存SackJS请求（如iframe）。 有关如何启用它的详细信息，请参阅 [SockJS客户端](https://github.com/sockjs/sockjs-client/) 页面。

### 26.3.4 心跳消息

SockJS协议要求服务器发送心跳消息以阻止代理结束连接。Spring SockJS配置有一个名为heartbeatTime的属性，可用于自定义设置频率。默认情况下，如果在该连接上没有发送其他消息，则在25秒后发送心跳。这25秒的值符合以下 [IETF](https://tools.ietf.org/html/rfc6202) 对互联网应用的建议。

>当 WebSocket/SockJS 使用STOMP时，如果STOMP客户端和服务器协商要心跳交换，则SockJS心跳被禁用。

Spring SockJS 还允许配置TaskScheduler用于安排心跳任务。 任务调度程序由一个线程池支持，其线程数默认值为可用处理器数量。 应用程序应考虑根据具体需要自定义设置。

### 26.3.5 Servlet 3异步请求

SockJS传输使用 HTTP 流和HTTP长轮询连接时需要比平常更长时间。 有关这些技术的概述，请参阅[此博客](https://spring.io/blog/2012/05/08/spring-mvc-3-2-preview-techniques-for-real-time-updates/)。

在Servlet 3 对异步支持完成，允许正在退出Servlet容器的线程处理一个请求(request)并将响应(response)写入另一个线程。

一个特定的问题是Servlet API不会为已经离开的客户端提供通知，请参阅 [SERVLET_SPEC-44](https://java.net/jira/browse/SERVLET_SPEC-44)。但是，Servlet容器在后续尝试写入响应时引发异常。由于Spring的SockJS服务支持服务器发送心跳（默认为每25秒），这意味着如果更频繁地发送心跳，通常会在该时间段内或更早的时间内检测到客户端断开连接。

>因此，网络IO错误可能仅仅因为客户端已断开，日志中可能会有大量IO错误堆栈跟踪信息。Spring尽最大努力确定那些代表客户端断开连接（具体到每个服务器）的网络故障，并使用AbstractSockJsSession中定义的专用日志类别DISCONNECTED_CLIENT_LOG_CATEGORY记录最小消息。如果需要查看堆栈跟踪，请将该日志类别设置为TRACE

26.3.6 SockJS 的 CORS 头

如果允许跨域请求（参见[第26.2.6节“配置允许的域”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#websocket-server-allowed-origins)），则SockJS协议在XHR流传输和轮询传输中使用CORS进行跨域支持。 因此，除非检测到响应中存在CORS头，否则自动添加CORS头。 因此，如果应用程序已经配置为提供CORS支持，例如 通过一个Servlet过滤器，Spring的SockJsService将跳过自动添加CORS头。

也可以通过Springs SockJs Service中的suppressors属性禁用添加这些CORS头。

以下是SockJS属性列表：

* "Access-Control-Allow-Origin" - 从“Origin”请求头的值初始化。
* "Access-Control-Allow-Credentials" - 总是设置为true。
* "Access-Control-Request-Headers" - 从等效请求头的值初始化。
* "Access-Control-Allow-Methods" - 支持的HTTP方法（请参阅TransportType枚举）
* "Access-Control-Max-Age" - 设定为31536000（1年）。

具体的实现，请参阅AbstractSockJsService中的addCorsHeaders以及源代码中的TransportType枚举。

或者，如果CORS配置允许，会考虑使用SockJS端点(endpoint )前缀排除URL，从而使Spring的SockJsService处理它。

### 26.3.7 SockJS客户端

Spring 提供了SockJS Java客户端，可以不使用浏览器连接到远程SockJS端点(endpoint)。当需要在公共网络上的2台服务器之间进行双向通信时，即网络代理可能排除使用WebSocket协议时，这一点尤其有用。 SockJS Java客户端对于测试目的也非常有用，例如模拟大量并发用户。

SockJS Java客户端支持“websocket”，“xhr-streaming”和“xhr-polling”传输。 其余的只有在浏览器中使用才有意义。

WebSocketTransport可以配置为：

* StandardWebSocketClient 在JSR356 的runtime
* JettyWebSocketClient使用Jetty 9+本地 WebSocket API
* Spring WebSocketClient的任何实现

按照定义，XhrTransport支持“xhr-streaming”和“xhr-polling”，因为从客户端的角度看，除了用于连接到服务器的URL之外，没有什么区别。 目前有两个实现：

* RestTemplateXhrTransport使用Spring的RestTemplate 进行 HTTP请求。
* JettyXhrTransport使用Jetty的HttpClient进行HTTP请求。

下面的示例显示了如何创建SockJS客户端并连接到SockJS端点(endpoint)：

```Java
List<Transport> transports = new ArrayList<>(2);
transports.add(new WebSocketTransport(new StandardWebSocketClient()));
transports.add(new RestTemplateXhrTransport());

SockJsClient sockJsClient = new SockJsClient(transports);
sockJsClient.doHandshake(new MyWebSocketHandler(), "ws://example.com:8080/sockjs");
```
>SockJS 使用JSON格式的数组传递消息，默认使用Jackson 2。或者，您可以配置SockJsMessageCodec的自定义实现并在SockJsClient上进行配置。

要使用SockJsClient来模拟大量并发用户，您需要配置底层HTTP客户端（对于XHR传输）来允许足够数量的连接和线程。 例如Jetty：

```Java
HttpClient jettyHttpClient = new HttpClient();
jettyHttpClient.setMaxConnectionsPerDestination(1000);
jettyHttpClient.setExecutor(new QueuedThreadPool(1000));
```
还要考虑配置这些服务器端的SockJS相关属性（有关详细信息，请参阅Javadoc）：

```Java
@Configuration
public class WebSocketConfig extends WebSocketMessageBrokerConfigurationSupport {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/sockjs").withSockJS()
            .setStreamBytesLimit(512 * 1024)
            .setHttpMessageCacheSize(1000)
            .setDisconnectDelay(30 * 1000);
    }

    // ...

}
```
## 26.4 WebSocket消息传递架构上的STOMP

WebSocket协议定义了两种类型的消息：文本和二进制，但是它们的内容是未定义的（没有纯文本/二进制，没有任何语义）。 一般我们认为客户端和服务器会同意使用子协议（即更高级协议）来定义消息语义。虽然我们并不强制使用WebSocket的子协议，但客户端和服务器都需要同意某种协议来帮助解释消息。

### 26.4.1 STOMP概述

[STOMP](https://stomp.github.io/stomp-specification-1.2.html#Abstract) 是一种简单的面向文本的消息协议，最初是为脚本语言（如Ruby，Python和Perl）创建的，用于连接企业消息代理。它旨在解决常用消息传递模式的一个子集。STOMP可以通过任何可靠的双向流网络协议（如TCP和WebSocket）来使用。虽然STOMP是面向文本的协议，但消息的有效内容可以是文本或二进制文件。

STOMP是一种基于帧(frame)的协议，其框架是基于HTTP建模的。 STOMP框架的结构：

```
COMMAND
header1:value1
header2:value2

Body^@
```
客户端可以沿着"destination"头使用SEND或SUBSCRIBE命令发送或订阅消息，"destination"头描述消息是什么以及谁应该接收消息。这使得能够使用简单的**发布订阅机制**来将消息通过代理发送到其他连接的客户端，或者向服务器发送消息以请求执行某些工作。

以下是客户端订阅接收股票报价的示例，服务器会周期性地发布股票报价。 通过计划的任务通过SimpMessagingTemplate发送消息到代理：

```
SUBSCRIBE
id:sub-1
destination:/topic/price.stock.*

^@
```
以下是客户端发送交易请求的示例，服务器可以通过@MessageMapping方法处理，在执行之后，向客户端广播交易确认消息和详细说明：

```
SEND
destination:/queue/trade
content-type:application/json
content-length:44

{"action":"BUY","ticker":"MMM","shares",44}^@
```
目的地(destination )的含义在STOMP规范中有意地不透明。它可以是任何字符串，完全由STOMP服务器来定义服务器支持的目的地的语义和语法。但通常目的地是路径形式的字符串，其中“/ topic / ..”表示发布 - 订阅（一对多），“/ queue /”意味着点对点（一到一）消息交换。

STOMP服务器可以使用MESSAGE命令向所有用户广播消息。 以下是向订阅客户端发送股票报价的服务器端示例：

```
MESSAGE
message-id:nxahklf6-1
subscription:sub-1
destination:/topic/price.stock.MMM

{"ticker":"MMM","price":129.45}^@
```
要知道一个非常重要的是服务器不能发送未经请求(unsolicited)的消息。 来自服务器的所有消息必须响应于特定的客户端订阅，并且服务器消息的“subscription-id”头部必须与客户端订阅的“id”头匹配。

上述概述提供对STOMP协议的最基本的了解。建议全面检查[STOMP协议规范](https://stomp.github.io/stomp-specification-1.2.html)。

使用STOMP作为WebSocket子协议的好处：

* 不需要自己规定消息格式
* 在浏览器中使用现有的 [stomp.js](https://github.com/jmesnil/stomp-websocket) 客户端
* 基于目的地(destination)路由消息的能力
* 可以选择使用RabbitMQ，ActiveMQ等专门的消息代理进行广播

最重要的是，使用STOMP（相比较 只用底层 WebSocket 接口）可以使Spring框架提供WebSocket应用程序级编程模型该模型与 Spring MVC提供的基于HTTP的编程模型方式相同。

### 26.4.2 通过WebSocket启用STOMP

Spring Framework通过Spring-messaging和spring-websocket模块提供了在WebSocket上使用STOMP的支持。以下是以URL path/portfolio  发送STOMP WebSocket/SockJS 端点的示例，其中目的地以“/ app”开头的消息会被路由到消息处理方法（即应用程序处理），以“/ topic”或“/ queue”开头的消息将被路由到消息代理（即广播到其他连接的客户端）：

```Java
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/portfolio").withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.setApplicationDestinationPrefixes("/app");
        config.enableSimpleBroker("/topic", "/queue");
    }

}
```
XML：

```XML
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        http://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:message-broker application-destination-prefix="/app">
        <websocket:stomp-endpoint path="/portfolio">
            <websocket:sockjs/>
        </websocket:stomp-endpoint>
        <websocket:simple-broker prefix="/topic, /queue"/>
    </websocket:message-broker>

</beans>
```
> 这里前缀“/app”是任意的。您可以选择任何前缀。这只是为了区分消息，将不同的消息路由到不同的消息处理方法，以便将应用程序工作与消息路由到代理程序以广播到订阅的客户端。
“/ topic”和“/ queue”前缀取决于正在使用的代理。 在简单的内存代理的情况下，前缀没有任何特殊的含义; 它只是一个表示目的地如何使用的公约（pub-sub将许多订阅者或点对点消息通常针对单个收件人定位）。 在使用专用代理的情况下，大多数经纪人使用“/ topic”作为具有pub-sub语义的目的地的前缀，并且对于具有点对点语义的目的地使用“/ queue”。 检查代理的STOMP页面以查看其支持的目标语义。

在浏览器端，可以使用 [stomp.js](https://github.com/jmesnil/stomp-websocket) 和 [sockjs-client](https://github.com/sockjs/sockjs-client) 连接：

```Javascript
var socket = new SockJS("/spring-websocket-portfolio/portfolio");
var stompClient = Stomp.over(socket);

stompClient.connect({}, function(frame) {
}
```
或者通过WebSocket连接（没有SockJS）：

```Javascript
var socket = new WebSocket("/spring-websocket-portfolio/portfolio");
var stompClient = Stomp.over(socket);

stompClient.connect({}, function(frame) {
}
```
请注意，以上的stompClient不需要指定登录名和密码头。 即使指定登录名和密码头，也将被忽略，或者被覆盖，在服务器端。 有关认证的更多信息，请参[见第26.4.8节“连接到专业代理”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#websocket-stomp-handle-broker-relay-configure)和[第26.4.10节“安全认证”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#websocket-stomp-authentication)。

### 26.4.3 消息流程

当配置STOMP端点时，Spring应用程序将作为连接的客户端的STOMP代理。 本节将介绍消息在应用程序中的流动情况。

spring-messaging 模块为异步消息处理提供了基础。 它包含源自Spring Integration项目的许多抽象，旨在用作消息传递应用程序中的构建块：

* [Message](http://docs.spring.io/spring-framework/docs/4.3.7.RELEASE/javadoc-api/org/springframework/messaging/Message.html)  —  一个带头和内容的消息。
* [MessageHandler](http://docs.spring.io/spring-framework/docs/4.3.7.RELEASE/javadoc-api/org/springframework/messaging/MessageHandler.html)  —  消息处理器。
* [MessageChannel](http://docs.spring.io/spring-framework/docs/4.3.7.RELEASE/javadoc-api/org/springframework/messaging/MessageChannel.html)  —  发送消息的管道，使发送者和接收者之间耦合度能够降低。
* [SubscribableChannel](http://docs.spring.io/spring-framework/docs/4.3.7.RELEASE/javadoc-api/org/springframework/messaging/SubscribableChannel.html)  —  扩展MessageChannel并向注册的MessageHandler订阅者发送消息。
* [ExecutorSubscribableChannel](http://docs.spring.io/spring-framework/docs/4.3.7.RELEASE/javadoc-api/org/springframework/messaging/support/ExecutorSubscribableChannel.html)  —  可以通过线程池异步传递消息的SubscribableChannel的具体实现。

@EnableWebSocketMessageBroker Java配置和<websocket：message-broker> XML配置都组合了一个具体的消息流。 下面是使用简单的内存中间件时的部分流程图：
![image](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/images/message-flow-simple-broker.png)

上述配置包括3个消息通道：

* "clientInboundChannel" 来自WebSocket客户端的消息。
* "clientOutboundChannel" 发送到WebSocket客户端的消息。
* "brokerChannel" 从应用程序中向代理发送消息。

同样的三个通道(channel)也与专用的代理一起使用，除了"broker relay"取代简单的代理：
![image](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/images/message-flow-broker-relay.png.pagespeed.ce.uNbmXXwXgl.png)

“clientInboundChannel”上的消息可以由应用程序（例如股票交易执行请求）的注释方法处理，或者可以转发给代理（例如客户订阅股票报价）。STOMP目标地址用于简单的基于前缀的路由。例如，“/app”前缀可以将消息路由到注释方法，而“/topic”和“/queue”前缀可以将消息路由到代理(broker)。

当消息处理(message-handling)注释方法具有返回类型时，其返回值作为Spring消息的有效内容发送到“brokerChannel”。代理(broker)又向客户广播信息。通过使用消息模板，也可以从应用程序中的任何地方发送消息到目标地址。例如， HTTP POST 处理方法可以向连接的客户端广播消息，或者服务组件可以周期性地广播股票报价。

以下是一个简单的例子来说明消息流：

```Java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/portfolio");
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.setApplicationDestinationPrefixes("/app");
        registry.enableSimpleBroker("/topic");
    }

}

@Controller
public class GreetingController {

    @MessageMapping("/greeting") {
    public String handle(String greeting) {
        return "[" + getTimestamp() + ": " + greeting;
    }

}
```
以下说明上述示例的消息流程：

* WebSocket客户端连接到“/portfolio”的WebSocket端点。
* “/topic/greeting”的订阅消息通过“clientInboundChannel”并转发给代理。
* 发送到“/app/greeting”的问候通过“clientInboundChannel”并转发给GreetingController。 控制器添加当前时间，返回值通过“brokerChannel”作为消息传递到“/topic/greeting”（目的地是根据惯例选择的，但可以通过@SendTo覆盖）。