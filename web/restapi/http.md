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

