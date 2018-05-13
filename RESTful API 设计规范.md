## RESTful API 设计规范

### 什么是RESTful API？

符合REST架构设计的API。REST全称是Representational State Transfer，中文意思是表述（编者注：通常译为表征）性状态转移。 它首次出现在2000年Roy Fielding的博士论文中，Roy Fielding是HTTP规范的主要编写者之一。 他在论文中提到："我这篇文章的写作目的，就是想在符合架构原理的前提下，理解和评估以网络为基础的应用软件的架构设计，得到一个功能强、性能好、适宜通信的架构。REST指的是一组架构约束条件和原则。" 如果一个架构符合REST的约束条件和原则，我们就称它为RESTful架构。

REST本身并没有创造新的技术、组件或服务，而隐藏在RESTful背后的理念就是使用Web的现有特征和能力， 更好地使用现有Web标准中的一些准则和约束。虽然REST本身受Web技术的影响很深， 但是理论上REST架构风格并不是绑定在HTTP上，只不过目前HTTP是唯一与REST相关的实例。 所以我们这里描述的REST也是通过HTTP实现的REST。

下面我们结合REST原则，围绕资源展开讨论，从资源的定义、获取、表述、关联、状态变迁等角度，列举一些关键概念并加以解释。

- 资源与URI
- 统一资源接口
- 资源的表述
- 资源的链接
- 状态的转移

#### 我们将按照下列步骤来设计面向资源的RESTful API

1.确定API提供的资源类型

2.查明不同资源间的关系

3.根据资源的类型和关系，决定资源名称的规范

4.决定资源的范式 (schema)

5.为资源加上方法的最小集合

### 1. 协议

API与用户的通信协议，总是使用HTTPs协议。

### 2.资源，域名及版本

应该尽量将API部署在专用域名之下。

[https://api.example.com](https://api.example.com/)

如果确定API很简单，不会有进一步扩展，可以考虑放在主域名下。

<https://example.org/api/>

有些API在URI里边带上版本号，例如:

- http://api.example.com/1.0/foo
- http://api.example.com/1.2/foo
- http://api.example.com/2.0/foo

如果我们把版本号理解成资源的不同表述形式的话，就应该只是用一个URL，并通过Accept头部来区分，还是以github为例，它的Accept的完整格式是:application/vnd.github[.version].param[+json]

对于v3版本的话，就是Accept: application/vnd.github.v3。对于上面的例子，同理可以使用使用下面的头部:

- Accept: vnd.example-com.foo+json; version=1.0
- Accept: vnd.example-com.foo+json; version=1.2
- Accept: vnd.example-com.foo+json; version=2.0

### 3. 路径

路径又称"终点"（endpoint），表示API的具体网址。

在RESTful架构中，每个网址代表一种资源（resource），**所以网址中不能有动词，只能有名词**，而且所用的名词往往与数据库的表格名对应。

一般来说，数据库中的表都是同种记录的"集合"（collection），**所以API中的名词也应该使用复数。**

举例来说，有一个API提供动物园（zoo）的信息，还包括各种动物和雇员的信息，则它的路径应该设计成下面这样。

<https://api.example.com/v1/zoos>

<https://api.example.com/v1/animals>

<https://api.example.com/v1/employees>

不要使用：

/getZoos /createNewAnimals /deleteAllEmployees

### 4. HTTP动词及对应状态码对应

对于资源的具体操作类型，由HTTP动词表示。

常用的HTTP动词有下面五个（括号里是对应的SQL命令）。

GET（SELECT）：从服务器取出资源（一项或多项）。

POST（CREATE）：在服务器新建一个资源。

PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。

PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）。

DELETE（DELETE）：从服务器删除资源。

还有两个不常用的HTTP动词。

HEAD：获取资源的元数据。

OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的。

下面列出了GET，DELETE，PUT和POST的典型用法:

#### GET

- 安全且幂等
- 获取表示
- 变更时获取表示（缓存）


- 200（OK） - 表示已在响应中发出


- 204（无内容） - 资源有空表示
- 301（Moved Permanently） - 资源的URI已被更新
- 303（See Other） - 其他（如，负载均衡）
- 304（not modified）- 资源未更改（缓存）
- 400 （bad request）- 指代坏请求（如，参数错误）
- 404 （not found）- 资源不存在
- 406 （not acceptable）- 服务端不支持所需表示
- 500 （internal server error）- 通用错误响应
- 503 （Service Unavailable）- 服务端当前无法处理请求

#### POST

- 不安全且不幂等
- 使用服务端管理的（自动产生）的实例号创建资源
- 创建子资源
- 部分更新资源
- 如果没有被修改，则不过更新资源（乐观锁）


- 200（OK）- 如果现有资源已被更改


- 201（created）- 如果新资源被创建
- 202（accepted）- 已接受处理请求但尚未完成（异步处理）
- 301（Moved Permanently）- 资源的URI被更新
- 303（See Other）- 其他（如，负载均衡）
- 400（bad request）- 指代坏请求
- 404 （not found）- 资源不存在
- 406 （not acceptable）- 服务端不支持所需表示
- 409 （conflict）- 通用冲突
- 412 （Precondition Failed）- 前置条件失败（如执行条件更新时的冲突）
- 415 （unsupported media type）- 接受到的表示不受支持
- 500 （internal server error）- 通用错误响应
- 503 （Service Unavailable）- 服务当前无法处理请求

#### PUT

- 不安全但幂等
- 用客户端管理的实例号创建一个资源
- 通过替换的方式更新资源
- 如果未被修改，则更新资源（乐观锁）


- 200 （OK）- 如果已存在资源被更改


- 201 （created）- 如果新资源被创建
- 301（Moved Permanently）- 资源的URI已更改
- 303 （See Other）- 其他（如，负载均衡）
- 400 （bad request）- 指代坏请求
- 404 （not found）- 资源不存在
- 406 （not acceptable）- 服务端不支持所需表示
- 409 （conflict）- 通用冲突
- 412 （Precondition Failed）- 前置条件失败（如执行条件更新时的冲突）
- 415 （unsupported media type）- 接受到的表示不受支持
- 500 （internal server error）- 通用错误响应
- 503 （Service Unavailable）- 服务当前无法处理请求

#### DELETE

- 不安全但幂等
- 删除资源


- 200 （OK）- 资源已被删除


- 301 （Moved Permanently）- 资源的URI已更改
- 303 （See Other）- 其他，如负载均衡
- 400 （bad request）- 指代坏请求
- 404 （not found）- 资源不存在
- 409 （conflict）- 通用冲突
- 500 （internal server error）- 通用错误响应
- 503 （Service Unavailable）- 服务端当前无法处理请求

### 5.结果过滤，排序，搜索

url最好越简短越好，对结果过滤、排序、搜索相关的功能都应该通过参数实现。

**过滤**：例如你想限制`GET /tickets` 的返回结果：只返回那些open状态的ticket， `GET /tickets?state=open` 这里的state就是过滤参数。

如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果。

下面是一些常见的参数。

- ?limit=10：指定返回记录的数量
- ?offset=10：指定返回记录的开始位置。
- ?page=2&per_page=100：指定第几页，以及每页的记录数。
- ?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
- ?animal_type_id=1：指定筛选条件

参数的设计允许存在冗余，即允许API路径和URL参数偶尔有重复。比如，GET /zoo/ID/animals 与 GET /animals?zoo_id=ID 的含义是相同的。

**排序**：和过滤一样，一个好的排序参数应该能够描述排序规则，而不和业务相关。复杂的排序规则应该通过组合实现。排序参数通过 `,` 分隔，排序参数前加 `-` 表示降序排列。

- GET /tickets?sort=-priority #获取按优先级降序排列的ticket列表
- GET /tickets?sort=-priority,created_at #获取按优先级降序排列的ticket列表，在同一个优先级内，先创建的ticket排列在前面。

**搜索**：有些时候简单的排序是不够的。我们可以使用搜索技术来实现

- GET /tickets?q=return&state=open&sort=-priority,create_at # 获取优先级最高且打开状态的ticket，而且包含单词return的ticket列表。

### 6. 错误处理

如果状态码是4xx，就应该向用户返回出错信息。一般来说，返回的信息中将error作为键名，出错信息作为键值即可。 使用详细的错误包装错误：

{

"errors": [

{

```
"userMessage": "Sorry, the requested resource does not exist",

"internalMessage": "No car found in the database",

"code": 34,

"more info": "http://dev.mwaysolutions.com/blog/api/v1/errors/12345"

```

}

]

}

### 7.状态的转移

有了上面的铺垫，再讨论REST里边的状态转移就会很容易理解了。

不过，我们先来讨论一下REST原则中的无状态通信原则。初看一下，好像自相矛盾了，既然无状态，何来状态转移一说?

其实，这里说的无状态通信原则，并不是说客户端应用不能有状态，而是指服务端不应该保存客户端状态。

#### 7.1 应用状态与资源状态

实际上，状态应该区分应用状态和资源状态，客户端负责维护应用状态，而服务端维护资源状态。

客户端与服务端的交互必须是无状态的，并在每一次请求中包含处理该请求所需的一切信息。

服务端不需要在请求间保留应用状态，只有在接受到实际请求的时候，服务端才会关注应用状态。

这种无状态通信原则，使得服务端和中介能够理解独立的请求和响应。

在多次请求中，同一客户端也不再需要依赖于同一服务器，方便实现高可扩展和高可用性的服务端。

但有时候我们会做出违反无状态通信原则的设计，例如利用Cookie跟踪某个服务端会话状态，常见的像J2EE里边的JSESSIONID。

这意味着，浏览器随各次请求发出去的Cookie是被用于构建会话状态的。

当然，如果Cookie保存的是一些服务器不依赖于会话状态即可验证的信息（比如认证令牌），这样的Cookie也是符合REST原则的。

#### 7.2 应用状态的转移

状态转移到这里已经很好理解了， "会话"状态不是作为资源状态保存在服务端的，而是被客户端作为应用状态进行跟踪的。客户端应用状态在服务端提供的超媒体的指引下发生变迁。服务端通过超媒体告诉客户端当前状态有哪些后续状态可以进入。

这些类似"下一页"之类的链接起的就是这种推进状态的作用——指引你如何从当前状态进入下一个可能的状态。

### 8.其他注意问题

设计规范应在设计进行过程中不断迭代，及时补充其他注意事项。