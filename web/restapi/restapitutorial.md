---
categories:  Web

tags: 
  - Web
  - REST API

title: REST API 权威指南

date: 2018-08-11
---

# 什么是 REST ？

REST架构风格描述了六个约束。应用于体系结构的这些约束最初由Roy Fielding在他的博士论文中提出（参见https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm），并定义了RESTful-style的基础 。

这六个约束是：

## Uniform Interface （统一接口）

统一接口约束定义了客户端和服务器之间的接口。 它简化并解耦了架构，使每个部件都能独立演变。 统一接口的四个指导原则是：

### Resource-Based （基于资源的）

使用URI作为资源标识符在请求中标识各个资源。 资源本身在概念上与返回给客户端的表示(representations)分开。 例如，服务器不直接发送其数据库内容，而是发送一些表示某些数据库记录的HTML，XML或JSON，例如，用芬兰语表示并以UTF-8编码，具体取决于请求的详细信息和服务器实现。

### Manipulation of Resources Through Representations

当客户端持有资源的表示（包括附加的任何元数据）时，它有足够的信息来修改或删除服务器上的资源，前提是它有权这样做。

### Self-descriptive Messages （自描述信息）

每条消息都包含足够的信息来描述如何处理该消息。 例如，要调用的解析器可以由Internet媒体类型 media type（以前称为MIME类型）指定。 响应还明确指出了它们是否可以被缓存。

### Hypermedia as the Engine of Application State (HATEOAS) （超媒体作为应用程序状态引擎）

客户端通过body 内容，查询字符串参数，请求header和请求的URI（资源名称）来提供状态。 服务通过正文内容，响应代码和响应标头向客户提供状态。 这在技术上被称为超媒体（或超文本中的超链接）。

除了上面的描述之外，HATEOS还意味着，在必要时，链接包含在返回的正文（或标题）中，以提供用于检索对象本身或相关对象的URI。 我们稍后会详细讨论这个问题。

任何REST服务必须提供的统一接口是其设计的基础

## Stateless （无状态的）

由于REST是REpresentational State Transfer的首字母缩写，statelessness （无状态）是关键。 这意味着要处理请求的状态必须包含在请求本身中，无论是作为URI，查询字符串参数，正文还是标题的一部分。 URI唯一标识资源，body包含该资源的状态（或状态变化）。 然后，在服务器进行处理之后，通过headers，状态和响应主体将适当的状态或重要状态的片断传送回客户端。

我们大多数已经在业界工作了一段时间的人习惯于在container（不是docker中的） 内编程，这为我们提供了“session”的概念，它在多个HTTP请求中维护状态。 在REST中，客户端必须包含服务器的所有信息以完成请求，如果该状态必须跨越多个请求，则根据需要重新发送状态。 无状态可以实现更高的可伸缩性，因为服务器不必维护，更新或传递该会话状态。 此外，负载均衡器不必担心无状态系统的会话亲和性。

那么状态(state)和资源(resource)之间的区别是什么？ 状态或应用程序状态是服务器关心的，以满足当前会话或请求所需的请求数据。 资源或资源状态是定义资源表示的数据 - 例如，存储在数据库中的数据。 将应用程序状态视为可能因客户端和每个请求而异的数据。 另一方面，资源状态在请求它的每个客户端都是不变的。

## Cacheable

与万维网一样，客户端可以缓存响应。 因此，响应必须隐式或显式地将自身定义为可缓存或不可缓存，以防止客户端重用陈旧或不适当的数据以影响进一步的请求。 管理良好的缓存部分或完全消除了一些客户端 - 服务器交互，进一步提高了可伸缩性和性能。

## Client-Server

统一接口将客户端与服务器分开。 这种关注点分离意味着，例如，客户端不关心数据存储，数据存储仍保留在每个服务器的内部，从而提高了客户端代码的可移植性。 服务器不关心用户界面或用户状态，因此服务器可以更简单，更具可伸缩性。 只要不改变接口，服务器和客户端也可以独立替换和开发。

## Layered System （分层系统）

客户端通常无法判断它是直接连接到终端服务器，还是中间服务器。 中间服务器可以通过启用负载平衡和提供共享缓存来提高系统可伸缩性。 Layers 也可以实施安全策略。

## Code on Demand (optional)

服务器能够通过向客户端传输可以执行的逻辑来临时扩展或自定义客户端的功能。 这样的示例可以包括编译的组件，例如Java applet和客户端脚本，例如JavaScript。

遵守这些约束，从而符合REST架构风格，将使任何类型的分布式超媒体系统具有理想的紧急(emergent)属性，例如性能，可伸缩性，简单性，可修改性，可见性，可移植性和可靠性。

**注意**：REST架构的唯一可选约束是code on demand。 如果服务违反任何其他约束，则严格来讲不能将其称为RESTful。

# REST API Quick Tips

无论技术上是不是REST（根据前面提到的六个约束条件），这里有一些推荐的类似REST的概念。 这六个快速提示将带来更好，更实用的服务。

## 使用HTTP动词使你的请求带有含意

API使用者能够发送GET，POST，PUT和DELETE请求，这极大地增强了给定请求的清晰度。

通常，四个主要的HTTP动词使用如下：

**GET** ：读取特定资源（通过标识符）或资源集合。

**PUT** ：更新特定资源（通过标识符）或资源集合。 如果资源标识符是事先已知的，也可以用于创建特定资源。

**DELETE** ：通过标识符删除指定资源。

**POST** ：创建一个新资源。 对于不适合其他类别的操作，也是一个万能的动词。

**注意** ：GET请求不得更改任何底层资源数据。 但可能会更新测量和跟踪数据，但URI标识的资源数据不应更改。

## 提供合理的资源名称

制作出色的API需要80％的艺术和20％的科学。 创建表示合理资源的URL层次结构是艺术部分。 拥有合理的资源名称（只是URL路径，例如/customers/12345/orders）可以提高给定请求的清晰度。

适当的资源名称为服务请求提供上下文，从而提高API的可理解性。 通过URI名称对资源进行分层查看，为消费者提供友好，易于理解的资源层次结构，以便在其应用程序中使用。

以下是一些URL路径（资源名称）设计的规则：

- 在你的网址中使用标识符，而不是在查询字符串中使用。 使用URL查询字符串参数非常适合过滤，但不适用于资源名称。
  - **Good:** /users/12345
  - **Poor:** /api?type=user&id=23
- 利用URL的分层特性来表示结构。
- 为你的客户而不是数据设计。
- 资源名称应为名词。 避免使用动词作为资源名称，以提高清晰度。 使用HTTP methods 指定请求的动词部分。
- 在URL段中使用复数形式，使用集合使API URI在所有HTTP方法中保持一致。
  - **Recommended:** /customers/33245/orders/8769/lineitems/1
  - **Not:** /customer/33245/order/8769/lineitem/1
- 避免在URL中使用集合词。 例如'customer_list'作为资源。 使用复数来隐含表示集合（例如，customers 代替customer_list）。
- 在URL段中使用小写，用下划线（'_'）或连字符（' - '）分隔单词。 有些服务器会忽略大小写，所以最好清楚。
- 保持URL尽可能短，尽可能少的分段。

## 使用HTTP响应代码指示状态

响应状态代码是HTTP规范的一部分。 它们中有很多可以gggg解决最常见的情况。 本着使RESTful服务包含HTTP规范的精神，我们的Web API应该返回相关的HTTP状态代码。 例如，当成功创建资源时（例如，来自POST请求），API应该返回HTTP状态代码201.这里有常用的HTTP状态代码列表，其列出了每个的详细描述。

“十大”HTTP响应状态代码的建议用法如下：

**200 OK** ：通用成功状态代码。 这是最常见的代码。 用于表示成功。  

**201 CREATED** ：成功创建（通过POST或PUT）。 将Location header设置为包含指向新创建的资源的链接（在POST上）。 响应body 内容可能存在也可能不存在。

**204 NO CONTENT** ：表示成功，但响应body中没有任何内容，通常用于DELETE和PUT操作。

**400 BAD REQUEST**  ：完成请求时的一般错误会导致无效状态。 例如域验证错误，缺少数据等。

**401 UNAUTHORIZED** ：丢失或无效的身份验证令牌。

**403 FORBIDDEN** ：当用户未被授权执行操作或资源由于某种原因（例如时间限制等）不可用时的错误代码。

**404 NOT FOUND** ：在找不到请求的资源时使用，不管是否不存在，或者是否是401或403，出于安全原因，服务需要屏蔽。

**405 METHOD NOT ALLOWED** ：表示请求的URL存在，但请求的HTTP方法不对。 例如，POST /users/12345，其API不支持以这种方式创建资源（使用提供的ID）。 返回405时必须设置Allow header表明支持的HTTP方法。 例如："Allow: GET, PUT, DELETE"

**409 CONFLICT** ：请求导致资源冲突时。 重复条目，例如尝试创建具有相同信息的两个客户，以及在不支持级联删除时删除根对象。

**500 INTERNAL SERVER ERROR** ：永远不要故意返回该状态码。 服务器端抛出异常时应使用catch-all捕捉。 仅将此用于客户端无法解决的错误（即服务器错误，client做啥也没有用，请联系后端人员解决）。

## 提供JSON和XML

一般只支持JSON就可以了，除非是高度标准化和受监管的行业，需要XML。 模式验证和命名空间，并提供JSON和XML，是非常高的。 理想情况下，让消费者使用HTTP Accept header在格式之间切换，或者只是在URL上将.xml的扩展名更改为.json。

请注意，一旦我们开始讨论XML支持，我们就会开始讨论用于验证，命名空间等的模式。除非你的行业需要，否则请尽量避免支持所有这些复杂性。 JSON旨在简化，简洁和实用。 如果可以的话，让你的XML看起来更简洁。

换句话说，使返回的XML更像JSON - 简单易读，不存在架构和命名空间细节，只有数据和链接。 如果它最终比这更复杂，那么XML的成本将是惊人的。 根据我的经验，过去几年没有人使用过XML响应，成本太昂贵了。

请注意，JSON-Schema提供了架构式验证功能。

## 创建细粒度资源

在开始时，最好创建模仿系统的底层应用程序域模型或数据库体系结构的API。 最终，聚合那些需要利用多个底层资源的服务来减少通信量。 但是，从单个资源创建更大的资源比从更大的聚合创建细粒度或单个资源要容易得多。 让自己轻松自如，从易于定义的小型资源开始，为这些资源提供CRUD功能。 你可以之后创建这些方面的用例，减少通信。

## 考虑连接性

REST的原则之一是连通性 - 通过超媒体链接（搜索HATEOAS）。 虽然没有超链接，服务仍然有用，但在响应中返回链接时，API会变得更具自我描述性和可发现性。 至少，“自我描述”的链接引用会告诉客户端如何检索数据。 此外，利用HTTP Location header 包含通过POST（或PUT）创建资源的链接。 对于在支持分页的响应中返回的集合，“first”，“last”，“next”和“prev”链接至少是非常有用的。

关于链接格式，有很多。 HTTP Web链接规范（[RFC5988](https://tools.ietf.org/search/rfc5988)）解释了如下链接：

链接是由Internationalised Resource Identifiers (IRIs) [[RFC3987](https://tools.ietf.org/search/rfc3987)]标识的两个资源之间的类型连接，包括：

- 上下文IRI，
- 链接关系类型
- 目标IRI，和
- 可选的目标属性。

链接可以被视为“{context IRI}在{target IRI}具有{relation type}资源的形式的声明，其具有{target attributes}。”



至少，按照规范中的建议放置HTTP链接头中的链接，或者在JSON表示中包含此HTTP链接样式的JSON表示（例如Atom样式链接，请参阅：[RFC4287](https://tools.ietf.org/search/rfc4287#section-4.2.7)）。 之后，随着REST API变得更加成熟，你可以在更复杂的链接样式中进行分层，例如[HAL + JSON](https://stateless.co/hal_specification.html)，[Siren](https://github.com/kevinswiber/siren)，[Collection + JSON ](https://amundsen.com/media-types/collection/) 或 [JSON-LD](https://json-ld.org/)等。

# Using HTTP Methods for RESTful Services

HTTP谓词构成了我们“统一接口”约束的主要部分，并为我们提供了与基于名词的资源相对应的动作。 主要或最常用的HTTP谓词（或方法，因为它们被正确调用）是POST，GET，PUT，PATCH和DELETE。 它们分别对应于创建，读取，更新和删除（或CRUD）操作。 还有许多其他动词，但使用频率较低。 在那些不常用的方法中，OPTIONS和HEAD比其他方法更频繁地使用。

下面的表总结了主要HTTP方法与资源URI结合使用时的建议返回值：

| HTTP 动词 | CRUD           | 整个集合（例如 /customers）                                  | 特定项目（例如 /customers/{id}）                             |
| --------- | -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| POST      | Create         | 201 (Created),'Location' header 包含指向新ID的 /customers/{id}的链接。 | 404 (Not Found), 409 (Conflict) if resource already exists.. |
| GET       | Read           | 200 (OK), list of customers. Use pagination, sorting and filtering to navigate big lists. | 200 (OK), single customer. 404 (Not Found), if ID not found or invalid. |
| PUT       | Update/Replace | 405 (Method Not Allowed), unless you want to update/replace every resource in the entire collection. | 200 (OK) or 204 (No Content). 404 (Not Found), if ID not found or invalid. |
| PATCH     | Update/Modify  | 405 (Method Not Allowed), unless you want to modify the collection itself. | 200 (OK) or 204 (No Content). 404 (Not Found), if ID not found or invalid. |
| DELETE    | Delete         | 405 (Method Not Allowed), unless you want to delete the whole collection—not often desirable. | 200 (OK). 404 (Not Found), if ID not found or invalid.       |

下面是对主要HTTP方法的更详细讨论。

## POST

POST动词最常用于创建新资源。 特别是，它用于创建从属资源。 也就是说，从属于其他一些（例如父）资源。 换句话说，在创建新资源时，POST负责将新资源与父资源相关联，分配ID（新资源URI）等。

成功创建后，返回HTTP status 201，返回Location header ，其中包含指向具有201 HTTP status 的新创建资源的链接。

POST既不安全也不是幂等。 因此，建议用于非幂等资源请求。 两个相同的POST请求最有可能导致两个资源包含相同的信息。

**Examples:**

- *POST http://www.example.com/customers*
- *POST http://www.example.com/customers/12345/orders*

## GET

HTTP GET方法用于读取（或检索）资源的表示。 在“正确”（或非错误）路径中，GET返回XML或JSON中的表示和HTTP响应代码200（OK）。 在错误情况下，它通常返回404（NOT FOUND）或400（BAD REQUEST）。

根据HTTP规范的设计，GET（以及HEAD）请求仅用于读取数据而不是更改它。 因此，当以这种方式使用时，它们被认为是安全的。 也就是说，可以调用它们而没有数据修改或损坏的风险 - 调用它一次具有与调用它10次相同的效果。 另外，GET（和HEAD）是幂等的，这意味着产生多个相同的请求最终会产生与单个请求相同的结果。

不要通过GET公开不安全的操作 - 它永远不应该修改服务器上的任何资源。

**Examples:**

- *GET http://www.example.com/customers/12345*
- *GET http://www.example.com/customers/12345/orders*
- *GET http://www.example.com/buckets/sample*

## PUT

PUT最常用于更新功能，与已知资源URI一起使用，请求body包含原始资源的需要更新的表示。

但是，在客户端而不是服务器选择资源ID的情况下，PUT也可用于创建资源。 换句话说，如果PUT是包含不存在的资源ID的值的URI。 同样，请求正文包含资源表示。 许多人认为这是令人费解和困惑的。 因此，如果有的话，应该谨慎使用这种方法创建资源。

或者，使用POST创建新资源并在正文表示中提供客户端定义的ID - 可能是不包含资源ID的URI（请参阅下面的POST）。

成功更新后，PUT返回200（如果未返回正文中的任何内容，则返回204）。 如果使用PUT进行创建，则在成功创建时返回HTTP状态201。 响应中的body是可选的 - 提供body会消耗更多带宽。 由于客户端已经设置了资源ID，因此无需在创建案例中通过Location header 返回链接。

PUT不是一个安全的操作，因为它修改（或创建）服务器上的状态，但它是幂等的。 换句话说，如果您使用PUT创建或更新资源，然后再次进行相同的调用，则资源仍然存在，并且仍然具有与第一次调用时相同的状态。

例如，如果在资源上调用PUT会增加资源中的计数器，则该调用不再是幂等的。 有时会发生这种情况，并且可能足以证明调用不是幂等的。 但是，建议保持PUT请求是幂等的。 强烈建议对非幂等请求使用POST。

**Examples:**

- *PUT http://www.example.com/customers/12345*
- *PUT http://www.example.com/customers/12345/orders/98765*
- *PUT http://www.example.com/buckets/secret_stuff*

## PATCH

PATCH用于修改数据。 PATCH请求只需要包含对资源的更改，而不是完整的资源。

这类似于PUT，但正文包含一组指令，描述当前驻留在服务器上的资源应如何修改以生成新版本。 这意味着PATCH body不应仅仅是资源的修改部分，而是某种补丁语言，如JSON Patch或XML Patch。

PATCH既不安全也不幂等。 然而，PATCH请求可以以幂等的方式发布，这也有助于防止在相似时间帧内在相同资源上的两个PATCH请求之间的冲突导致的不良结果。 来自多个PATCH请求的冲突可能比PUT冲突更危险，因为某些补丁格式需要从已知的基点操作，否则它们将破坏资源。 使用此类补丁应用程序的客户端应使用条件请求，以便在客户端上次访问资源后资源已更新时请求将失败。 例如，客户端可以在PATCH请求的If-Match标头中使用强ETag。

**Examples:**

- *PATCH http://www.example.com/customers/12345*
- *PATCH http://www.example.com/customers/12345/orders/98765*
- *PATCH http://www.example.com/buckets/secret_stuff*

## DELETE

DELETE很容易理解。 它用于删除由URI标识的资源。

成功删除后，返回HTTP状态200（OK）以及响应正文，可能是已删除项目的表示（通常需要太多带宽），或者包装响应（请参阅下面的返回值）。 要么是返回HTTP状态204（NO CONTENT）而没有响应正文。 换句话说，建议的响应是204状态，没有正文，或JSEND样式响应和HTTP状态200。

HTTP-spec-wise，DELETE操作是幂等的。 如果删除资源，则将其删除。 在该资源上反复调用DELETE最终结果相同：资源消失了。 如果调用DELETE说，递减计数器（在资源内），则DELETE调用不再是幂等的。 如前所述，只要没有资源数据被改变，就可以更新使用统计和测量，同时仍然考虑服务幂等。 建议使用POST进行非幂等资源请求。

但是，有一个关于DELETE幂等性的警告。 第二次在资源上调用DELETE通常会返回404（NOT FOUND），因为它已被删除，因此不再可查找。 根据一些观点，这使得DELETE操作不再是幂等的，但是，资源的最终状态是相同的。 返回404是可以接受的，并准确地传达调用的状态。

**Examples:**

- *DELETE http://www.example.com/customers/12345*
- *DELETE http://www.example.com/customers/12345/orders*
- *DELETE http://www.example.com/bucket/sample*

# Resource Naming

除了适当地利用HTTP动词之外，在创建易于理解，易于使用的Web服务API时，资源命名可以说是最有争议和最重要的概念。 当资源命名良好时，API直观且易于使用。 做得不好，相同的API可能会感觉到笨拙的并且难以使用和理解。 以下是为新API创建资源URI时的一些提示。

从本质上讲，RESTful API最终只是URI的集合，对这些URI的HTTP调用以及资源的一些JSON或XML表示，其中许多将包含关系链接。 URIs涵盖RESTful可寻址性原则。 每个资源都有自己的地址或URI-服务器可以提供的每条有用的信息都作为资源公开。 统一接口的约束部分通过URI和HTTP动词的组合来解决，并且根据标准和约定使用它们。

