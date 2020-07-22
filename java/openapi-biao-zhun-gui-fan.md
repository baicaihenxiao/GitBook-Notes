# OpenAPI 标准规范

[https://mp.weixin.qq.com/s/Ow7tkcnpY37faHYHd12ENQ](https://mp.weixin.qq.com/s/Ow7tkcnpY37faHYHd12ENQ)

## **什么是API规范**

API 是模块或者子系统之间交互的接口定义。好的系统架构离不开好的 API 设计，而一个设计不够完善的 API 则注定会导致系统的后续发展和维护非常困难。在关键环节制定明确的API规范有助于 Service 对内提高产品间互通的效率，对外提供一致的使用体验，也有助于更好地被集成。

对于API规范，比较知名的是 **OpenAPI Specfication**\[1\] 和 **Google API Design Guide**\[2\]。前者针对 RESTful API 设计在细节层面给出了非常具体的规定，已经成为 RESTful API 设计领域的事实标准，而后者则主要从云厂商的角度提出许多最佳实践性质的规范与建议，这些原则不仅仅适用于 RESTful API，也适合其他类型API设计。

虽然RESTful设计风格曝光率很高，但并不是所有云服务商都选择了完全遵循RESTful，例如AWS和 **阿里云**\[3\] RPC 风格反而占了大多数，Google和Azure则RESTful居多。

RESTful API的优势是HTTP具备更好的易用性，让异构系统更容易集成，且开发执行效率比较高，面向资源要求也比较高。而RPC API可以使用更广泛的框架和方案，技术层面更底层也更为灵活，设计起来相对简单，掌握起来有一定门槛，架构上更加复杂。RESTful 与 RPC 模式对比如下：

|  | RESTful API | RPC API |
| :--- | :--- | :--- |
| 是否有统一规范 | HTTP | 无 |
| 面向资源 | 是 | 不确定 |
| 性能 | 中 | 高 |
| 通用性 | 高 | 弱 |
| 复杂度 | 中 | 高 |

如果强制统一风格，有些适合 RESTful 风格的服务非要使用RPC的话，看起来就会比较丑陋，如果只是一个过程化的服务调用，往 RESTful 资源化设计方向去靠会比较困难。但如果不强制使用统一风格，会造成针对API的体系化支持会更加复杂，例如为兼容两种风格SDK的自动化支持需要两套代码。

选择API风格时要考虑几个问题：

* 选择支持哪种风格，才能更好地体现业务特性，让客户操作起来更加方便；
* 设计API时能否面向资源设计，相应的工程人员是否具备做这种设计的能力；
* 针对这种风格工具链的支持是否到位，投入产出比如何；
* 业界流行的趋势如何，是否需要考虑与其他系统体系的互操作。

用户使用API来访问 Service，本质上是想通过对某种资源执行特定的操作来完成一个业务动作。对于资源有两个关键点：一是要有统一的资源模型；二是要明确资源关系。统一的资源模型对 Service 的帮助是巨大的：

* 它可以使API具有更清晰的结构，帮助用户理解；
* 它可以帮助对比API与后台实体关系模型，更容易提供更完整的API服务；
* 它可以使产品协作更加顺畅，对资源的操作也更加规范化；
* 它可以使云服务底层平台实现起来更统一、更方便；
* 它可以使围绕API的生态集成起来更加简单、高效。

确定了设计模式和资源模型后，就需要考虑 API的设计细节了，诸如API名称、参数名、属性名称、数据格式、错误码之类的信息。除此之外，还要考虑以下一些问题：

* 在API命名的时候，遵循什么样的范式来确保大体风格相似？动词、名词、介词如何组合才能保持API风格看起来比较统一，降低理解成本？
* 对于类似的操作，有没有使用规范？有哪些公共的标准词汇使得同类型的操作可以比较容易理解，避免使用晦涩奇怪的词汇（例如读操作，Read/Query/Describe/List/Get中都在什么场合使用什么动词）？
* 被广泛使用的参数如何尽可能保持一致，避免不同产品的表达混乱的情况（例如分页参数用PageNumber还是PageNum）？
* 对于常用的场景，例如幂等、分页、异步API的设计有没有统一的规范，避免使用体验不一致？
* 错误码应该怎么设计？公共错误码怎么统一，业务错误码怎么表达？

上述问题都是实际研发过程中要注意的，要全部罗列的话远不止这些。API的用词描述是 Service 展现给外部用户的第一印象，绝非随意写就。对人员有一定规模，内部有多条产品线的组织来说，如何协调组织的各个部分对外具有统一的体验是个很大挑战。

Service 在管理API时应该考虑一些具体的规范，对命名规则、标准词汇、最佳实践模式、错误码等信息都有明确的规定，同时用系统化、平台化的手段来管理API，确保不走偏。设计风格不是云服务API设计中致命的问题，但是它关乎云服务外表形象，不可不察。

API是后端服务的外部表达，是服务就有可能出现问题，无论这个问题是可预期的还是不可预期的。如果只考虑功能本身功能特性，而忽视对异常情况的设计，当问题出现的时候业务本身可能无法感知造成服务异常，更重要的是站在客户角度去看，不能有效获取错误原因是非常痛苦的，很多时候只能束手无策，降低云服务提供商的整体口碑，甚至损害营收。

## **OpenAPI规范**

本规范基于 RESTful 风格的架构设计准则，广泛参考 GitHub、Azure、Google API Design Guide、腾讯云、阿里云等公开资料，兼顾现有实际情况和未来发展做一个概括性记录总结。

### 一、协议

API 与用户的通信协议，总是使用 HTTPS 协议。这个和 RESTful API 本身没有很大的关系，但是对于增加网站的安全是非常重要的。特别如果你提供的是公开 API，用户的信息泄露或者被攻击会严重影响网站的信誉。

### 二、版本（Version）

关于版本的设计有3种形式：

1. 将 API 的版本号放入 URL 中，如：`http://api.example.com/v1`，这样方便和直观；
2. 将版本号记录在 url query中，如：`http://api.example.com?param1=val&version=1.0`中的 version 参数。
3. 将版本号放在 HTTP 头信息中，基于的准则是：不同的版本，可以理解成同一种资源的不同形式，所以应该采用同一个URL。如：`Accept: application/json; version=1.0`，可以参考**Github API Design**\[4\]和 **Versioning REST Services**\[5\]；

根据现有的实际情况，如果是为了兼容已存在的服务接口，可以采用对应的形式。如果是新构建的体系结构，建议采用第三种。

### 三、Schema

URI的格式定义如下：`URI = scheme "://" authority "/" path \[ "?" query \] \[ "#" fragment \]`

URL 是 URI 的一个子集\(一种具体实现\)，对于 REST API 来说一个资源一般对应一个唯一的 URI（URL）。在 URL 的设计中，我们会遵循一些规则，使接口看起透明易读，方便使用者调用。

"/"分隔符一般用来对资源层级的划分。对于 RESTful API 来说，"/"只是一个分隔符，并无其他含义。为了避免混淆，"/"不应该出现在URL的末尾。

**URL 中尽量使用连字符"-"代替下划线"\_"的使用。** 连字符"-"一般用来分割 URL 中出现的字符串\(单词\)，来提高 URL 的可读性，例如：`http://api.example.restapi.org/blogs/mark-masse/entries/this-is-my-first-post`。使用下划线"_"来分割字符串\(单词\)可能会和链接的样式冲突重叠，而影响阅读性。但实际上，"-"和"_"对URL 中字符串的分割语意上还是有些差异的："-"分割的字符串\(单词\)一般各自都具有独立的含义，可参见上面的例子。而"\_"一般用于对一个整体含义的字符串做了层级的分割，方便阅读，例如你想在 URL 中体现一个 IP 地址的信息：210\_110\_25\_88 . \(欢迎关注：朱小厮的博客\)

**URL应该统一使用小写字母**。

**URL中不要包含文件\(脚本\)的扩展名**。例如 `.json` 之内的就不要出现了，对于接口来说没有任何实际的意义。如果是想对返回的数据内容格式标示的话，通过 HTTP Header 中的 Content-Type 字段更好一些。

对于响应返回的格式，JSON 因为它的可读性、紧凑性以及多种语言支持等优点，成为了 HTTP API 最常用的返回格式。因此，最好采用 JSON 作为返回内容的格式。如果用户需要其他格式，比如 `xml`，应该在请求头部 `Accept` 中指定。对于不支持的格式，服务端需要返回正确的 `status code`，并给出详细的说明。

**JSON中的所有字段都应该用小写的蛇形命名形式，而不是采用驼峰命名。**

### 四、以资源为中心的 URL 设计

资源是 `Restful API` 的核心元素，所有的操作都是针对特定资源进行的。而资源就是 `URL`（Uniform Resoure Locator）表示的，所以简洁、清晰、结构化的 URL 设计是至关重要的。Github 可以说是这方面的典范，下面我们就拿 `repository` 来说明：

```text
/users/:username/repos
/users/:org/repos
/repos/:owner/:repo
/repos/:owner/:repo/tags
/repos/:owner/:repo/branches/:branch
```

我们可以看到几个特性：

* 资源分为单个文档和集合，尽量使用复数来表示资源，单个资源通过添加 id 或者 name 等来表示
* 一个资源可以有多个不同的 URL
* 资源可以嵌套，通过类似目录路径的方式来表示，以体现它们之间的关系

**最常见的一种设计错误，就是URL包含动词。** 因为"资源"表示一种实体，所以应该是名词，URL 不应该有动词，动词应该放在 HTTP Method （参考下一条）中。举例来说，某个 URL 是 `/users/show/1`，其中 `show`是动词，这个 URL 就设计错了，正确的写法应该是 `/users/1`，然后用 HTTP GET 方法表示 show。

如果某些动作是HTTP 动词表示不了的，你可以把动作看成是一种资源。比如网上汇款，从账户1向账户2汇款500元，错误的 URL 是：

```text
POST /accounts/1/transfer/500/to/2
```

正确的写法是把动词 transfer 改成transaction，资源不能是动词，但是可以是一种服务：

```text
POST  /transactoin HTTP/1.1
HOST: 127.0.0.1

from=1&to=2&amount=500
```

### 五、正确使用 HTTP Method

有了资源的 URI 设计，所有针对资源的操作都是使用 HTTP 方法指定的。比较常用的 HTTP/1.1 动词有下面5个：

* GET：从服务器取出资源（一项或多项）。
* POST：在服务器新建一个资源。
* PUT：在服务器更新资源（客户端提供改变后的完整资源）。
* PATCH：在服务器更新资源（更新资源的部分属性）。
* DELETE：从服务器删除资源。

还有4个不常用的 HTTP/1.1 动词：

* HEAD：只获取某个资源的头部信息。比如只想了解某个文件的大小，某个资源的修改日期等
* OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的。
* TRACE：追踪路径。不建议使用。
* CONNECT：要求用隧道协议连接代理。不建议使用。

举例：

```text
GET /repos/:owner/:repo/issues
GET /repos/:owner/:repo/issues/:number
POST /repos/:owner/:repo/issues
PATCH /repos/:owner/:repo/issues/:number
DELETE /repos/:owner/:repo
```

这里顺带探讨一下，HTTP 协议涉及到的一种重要性质：幂等性\(Idempotence\)。在 HTTP/1.1 规范中幂等性的定义是：

> Methods can also have the property of "idempotence" in that \(aside from error or expiration issues\) the side-effects of N &gt; 0 identical requests is the same as for a single request.

从定义上看，HTTP 方法的幂等性是指一次和多次请求某一个资源应该具有同样的副作用。幂等性属于语义范畴，正如编译器只能帮助检查语法错误一样，HTTP 规范也没有办法通过消息格式等语法手段来定义它，这可能是它不太受到重视的原因之一。但实际上，幂等性是分布式系统设计中十分重要的概念，而 HTTP 的分布式本质也决定了它在 HTTP 中具有重要地位。

安全方法是指不修改资源的 HTTP 方法。譬如，当使用 GET 或者 HEAD 作为资源 URL，都必须不去改变资源。然而，这并不全准确。意思是：它不改变资源的表示形式。对于安全方法，它仍然可能改变服务器上的内容或资源，但这必须不导致不同的表现形式。

|  | HTTP Method | 幂等 | 安全 |
| :--- | :--- | :--- | :--- |
| 1 | OPTIONS | yes | yes |
| 2 | GET | yes | yes |
| 3 | HEAD | yes | yes |
| 4 | PUT | yes | no |
| 5 | POST | no | no |
| 6 | DELETE | yes | no |
| 7 | PATCH | no | no |

实际上接口的幂等或者安全与否取决于接口的实现，只是 HTTP Method 语义上我们约定俗成地认为实现的过程会参照上表所示。对于幂等的接口，客户端就可以放心地多次调用，网关层面也可以重试。

### 六、状态码 （Status Code）

HTTP 应答中，需要带一个很重要的字段：`status code`。它说明了请求的大致情况，是否正常完成、需要进一步处理、出现了什么错误，对于客户端非常重要。状态码都是三位的整数，大概分成了几个区间：

* `2XX`：请求正常处理并返回
* `3XX`：重定向，请求的资源位置发生变化
* `4XX`：客户端发送的请求有错误
* `5XX`：服务器端错误

在 HTTP API 设计中，经常用到的状态码以及它们的意义如下表：

| Status Code | 语义 | 说明 |
| :--- | :--- | :--- |
| 200 | OK | 请求已成功 |
| 201 | Created | 请求已完成，并导致了一个或者多个资源被创建，最常用在 POST 创建资源的时候 |
| 202 | Accepted | 请求已经接收并开始处理，但是处理还没有完成。一般用在异步处理的情况，响应 body 中应该告诉客户端去哪里查看任务的状态 |
| 204 | No Content | 请求已经处理完成，但是没有信息要返回，经常用在 PUT 更新资源的时候（客户端提供资源的所有属性，因此不需要服务端返回）。如果有重要的 metadata，可以放到头部返回 |
| 301 | Moved Permanently | 请求的资源已经永久性地移动到另外一个地方，后续所有的请求都应该直接访问新地址。服务端会把新地址写在 `Location` 头部字段，方便客户端使用。允许客户端把 POST 请求修改为 GET。 |
| 302 | Moved Temporarily | 临时重定向 |
| 304 | Not Modified | 请求的资源和之前的版本一样，没有发生改变。用来缓存资源，和条件性请求（conditional request）一起出现 |
| 307 | Temporary Redirect | 目标资源暂时性地移动到新的地址，客户端需要去新地址进行操作，但是 **不能** 修改请求的方法。 |
| 308 | Permanent Redirect | 和 301 类似，除了客户端 **不能** 修改原请求的方法 |
| 400 | Bad Request | 1.语义有误，当前请求无法被服务器理解; 2. 请求参数有误。 |
| 401 | Unauthorized | 当前请求需要身份验证。 |
| 403 | Forbidden | 服务器已经理解请求，但是拒绝执行它。与401响应不同的是，身份验证并不能提供任何帮助，而且这个请求也不应该被重复提交。如果这不是一个 HEAD 请求，而且服务器希望能够讲清楚为何请求不能被执行，那么就应该在实体内描述拒绝的原因。当然服务器也可以返回一个404响应，假如它不希望让客户端获得任何信息。 |
| 404 | Not Found | 请求失败，请求所希望得到的资源未被在服务器上发现。 |
| 405 | Method Not Allowed | 请求行中指定的请求方法不能被用于请求相应的资源。该响应必须返回一个Allow 头信息用以表示出当前资源能够接受的请求方法的列表。鉴于 PUT，DELETE 方法会对服务器上的资源进行写操作，因而绝大部分的网页服务器都不支持或者在默认配置下不允许上述请求方法，对于此类请求均会返回405错误。 |
| 406 | Not Acceptable | 请求的资源的内容特性无法满足请求头中的条件，因而无法生成响应实体。 |
| 409 | Conflict | 由于和被请求的资源的当前状态之间存在冲突，请求无法完成。 |
| 429 | Too Many Requests | 资源配额不足或达到速率限制。 |
| 499 | Client Closed Request | 请求被客户端取消。 |
| 500 | Internal Server Error | 服务器内部错误 |
| 501 | Not Implemented | 服务器不支持当前请求所需要的某个功能。当服务器无法识别请求的方法，并且无法支持其对任何资源的请求。 |
| 502 | Bad Gateway | 作为网关或者代理工作的服务器尝试执行请求时，从上游服务器接收到无效的响应。 |
| 503 | Service Unavailable | 由于临时的服务器维护或者过载，服务器当前无法处理请求。这个状况是临时的，并且将在一段时间以后恢复。 |
| 504 | Gateway Timeout | 作为网关或者代理工作的服务器尝试执行请求时，未能及时从上游服务器（URI标识出的服务器，例如HTTP、FTP、LDAP）或者辅助服务器（例如DNS）收到响应。 |
| 505 | HTTP Version Not Supported | 服务器不支持，或者拒绝支持在请求中使用的 HTTP 版本。 |

上面这些状态码覆盖了 API 设计中大部分的情况，如果对某个状态码不清楚或者希望查看更完整的列表，可以参考 **HTTP Status Code**\[6\] 、**维基百科-状态码**\[7\]或者 **RFC7231 Response Status Codes**\[8\] 的内容。

### 七、错误处理（Error Handling）

如果出错，应该在 response body 中通过 `message` 给出明确的错误信息（一般来说，返回的信息中将 message 作为键名，出错详情作为键值即可）。比如客户端发送的请求有错误，一般会返回 `4XX Bad Request` 结果。这个结果很模糊，给出错误 `message` 的话，能更好地让客户端知道具体哪里有问题，进行快速修改。

```text
{
 "message":"错误详情"
}
```

错误详情应该可以帮助用户轻松快捷地理解和解决API 错误。通常，在编写错误详情时请考虑以下准则：

* 不要假设用户是您 API 的专家用户。用户可能是客户端开发人员、操作人员、IT 人员或应用的最终用户。
* 不要假设用户了解有关服务实现的任何信息，或者熟悉错误的上下文（例如日志分析）。
* 如果可能，应构建错误详情，以便技术用户（但不一定是 API 开发人员）可以响应错误并改正。
* 确保错误详情内容简洁。如果需要，请提供一个链接，便于有疑问的读者提问、提供反馈或详细了解错误详情中不方便说明的信息。此外，可使用详细信息字段来提供更多信息。

### 八、命名规则

为了能够长时间在众多 API 中为开发者提供一致的体验，API 使用的所有名称都应该具有以下特点：

* 简单
* 直观
* 一致

这包括接口、资源、集合、方法和消息的名称。

由于很多开发者不是以英语为母语，所以这些命名惯例的目标之一是确保大多数开发者可以轻松理解 API。对于方法和资源，我们鼓励使用简单、直观和少量的词汇来命名。

* API 名称 **应该** 使用正确的美式英语。例如，使用美式英语的 license、color，而非英式英语的 licence、colour。
* 为了简化命名， **可以** 使用已被广泛接受的简写形式或缩写。例如，API 优于 Application Programming Interface。
* 尽量使用直观、熟悉的术语。例如，如果描述移除（和销毁）一个资源，则删除优于擦除。
* 使用相同的名称或术语命名同样的概念，包括跨 API 共享的概念。
* 避免名称过载。使用不同的名称命名不同的概念。
* 避免在 API 的上下文以及更大的 API 生态系统中使用含糊不清且过于笼统的名称。这些名称可能导致对 API 概念的误解。相反，应选择能准确描述 API 概念的名称。这对定义一阶 API 元素（例如资源）的名称尤其重要。没有要避免使用的名称的明确列表，因为每个名称都必须放在其他名称的上下文中进行评估。实例、信息和服务是过去有问题的名称的示例。所选择的名称应清楚地描述 API 概念（例如：什么的实例？），并将其与其他相关概念区分开（例如：“alert”是指规则、信号还是通知？）。
* 仔细考虑是否使用可能与常用编程语言中的关键字相冲突的名称。您可以使用这些名称，但在 API 审核期间可能会触发额外的审查。因此应谨慎使用。

### 九、认证和授权（Authentication & Authorization）

一般来说，让任何人随意访问公开的 API 是不好的做法。验证和授权是两件事情：

* 验证（Authentication）是为了确定用户是其申明的身份，比如提供账户的密码。不然的话，任何人伪造成其他身份（比如其他用户或者管理员）是非常危险的
* 授权（Authorization）是为了保证用户有对请求资源特定操作的权限。比如用户的私人信息只能自己能访问，其他人无法看到；有些特殊的操作只能管理员可以操作，其他用户有只读的权限等等

如果没有通过验证（提供的用户名和密码不匹配，token 不正确等），需要返回 **401 Unauthorized**\[9\]状态码，并在 body 中说明具体的错误信息；而没有被授权访问的资源操作，需要返回 **403 Forbidden**\[10\] 状态码，还有详细的错误信息。

> NOTES: 借鉴于 Github，它对某些用户未被授权访问的资源操作返回 **404 Not Found**\[11\]，目的是为了防止私有资源的泄露（比如黑客可以自动化试探用户的私有资源，返回 403 的话，就等于告诉黑客用户有这些私有的资源）。

### 十、限流（RateLimit）

如果对访问的次数不加控制，很可能会造成 API 被滥用，甚至被 **DDOS 攻击**\[12\]。根据使用者不同的身份对其进行限流，可以防止这些情况，减少服务器的压力。

对用户的请求限流之后，要有方法告诉用户它的请求使用情况，本文档推荐使用的三个相关的头部：

* `X-RateLimit-Limit`: 用户每个小时允许发送请求的最大值
* `X-RateLimit-Remaining`：当前时间窗口剩下的可用请求数目
* `X-RateLimit-Rest`: 时间窗口重置的时候，到这个时间点可用的请求数量就会变成 `X-RateLimit-Limit` 的值

举例：

```text
X-Ratelimit-Limit: 18000
X-Ratelimit-Remaining: 17995
X-Ratelimit-Reset: 1590570990
```

如果允许没有登录的用户使用 API（可以让用户试用），可以把 `X-RateLimit-Limit` 的值设置得很小，比如 `60`。没有登录的用户是按照请求的 IP 来确定的，而登录的用户按照认证后的信息来确定身份。

对于超过流量的请求，可以返回 **429 Too many requests**\[13\] 状态码，并附带错误信息。

### 十一、编写优秀的文档

API 最终是给人使用的，不管是公司内部，还是公开的 API 都是一样。即使我们遵循了上面提到的所有规范，设计的 API 非常优雅，用户还是不知道怎么使用我们的 API。最后一步，但非常重要的一步是：为你的 API 编写优秀的文档。

对每个请求以及返回的参数给出说明，最好给出一个详细而完整地示例，提醒用户需要注意的地方……反正目标就是用户可以根据你的文档就能直接使用 API，而不是要发邮件给你，或者跑到你的座位上问你一堆问题。

### 参考资料

\[1\]OpenAPI Specfication: [https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md)\[2\]Google API Design Guide: [https://cloud.google.com/apis/design](https://cloud.google.com/apis/design)\[3\]阿里云: [https://bytedance.feishu.cn/docs/doccnrP6Ud3YTavonOlxds9lzad\#I1tZbK](https://bytedance.feishu.cn/docs/doccnrP6Ud3YTavonOlxds9lzad#I1tZbK)\[4\]Github API Design: [https://developer.github.com/v3/\#current-version](https://developer.github.com/v3/#current-version)\[5\]Versioning REST Services: [https://www.informit.com/articles/article.aspx?p=1566460](https://www.informit.com/articles/article.aspx?p=1566460)\[6\]HTTP Status Code: [https://httpstatuses.com/](https://httpstatuses.com/)\[7\]维基百科-状态码: [https://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81](https://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81)\[8\]RFC7231 Response Status Codes: [https://tools.ietf.org/html/rfc7231\#section-6](https://tools.ietf.org/html/rfc7231#section-6)\[9\]401 Unauthorized: [https://httpstatuses.com/401](https://httpstatuses.com/401)\[10\]403 Forbidden: [https://httpstatuses.com/403](https://httpstatuses.com/403)\[11\]404 Not Found: [https://httpstatuses.com/404](https://httpstatuses.com/404)\[12\]DDOS 攻击: [https://en.wikipedia.org/wiki/Denial-of-service\_attack](https://en.wikipedia.org/wiki/Denial-of-service_attack)\[13\]429 Too many requests: [https://httpstatuses.com/429](https://httpstatuses.com/429)

