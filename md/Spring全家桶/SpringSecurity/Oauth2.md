---
tags:
  - SpringBoot
  - SpringSecurity
  - Oauth2
---
## 简介

OAuth 是一个开放标准，该标准允许用户让第三方应用访问该用户在某一网站上存储的私密资源（如头像、照片、视频等），而在这个过程中无需将用户名和密码提供给第三方应用。实现这一功能是通过提供一个令牌（token），而不是用户名和密码来访问他们存放在特定服务提供者的数据。采用令牌（token）的方式可以让用户灵活的对第三方应用授权或者收回权限。

OAuth2 是 OAuth 协议的下一版本，但不向下兼容 OAuth 1.0。传统的 Web 开发登录认证一般都是基于 session 的，但是在前后端分离的架构中继续使用 session 就会有许多不便，因为移动端（Android、iOS、微信小程序等）要么不支持 cookie（微信小程序），要么使用非常不便，对于这些问题，使用 OAuth2 认证都能解决。

对于大家而言，我们在互联网应用中最常见的 OAuth2 应该就是各种第三方登录了，例如 QQ 授权登录、微信授权登录、微博授权登录、GitHub 授权登录等等。

### 四种角色

- Resource Owner：资源所有者，能够授予对受保护资源的访问权的实体。当资源所有者是一个人时，它被称为最终用户。
- Resource Server：资源服务器，托管受保护资源的服务器，能够使用访问令牌接收和响应受保护资源请求。
- Client：客户端（第三方应用），代表资源所有者并经其授权发出受保护资源请求的应用程序。术语“客户端”并不意味着任何特定的实现特征 (例如，应用程序是否在服务器、桌面或其他设备上执行)。
- Authorization Server：授权服务器，在成功验证资源所有者并获得授权后向客户端返回访问令牌。

授权服务器和资源服务器之间的交互超出了本规范的范围。授权服务器可以是与资源服务器相同的服务器，也可以是独立的实体。单个授权服务器可以颁发被多个资源服务器所接受的访问令牌。

### 协议流程

流程如下所示：[RFC 6749 - The OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/html/rfc6749#section-1.2)

```
+--------+                               +---------------+
|        |--(A)- Authorization Request ->|   Resource    |
|        |                               |     Owner     |
|        |<-(B)-- Authorization Grant ---|               |
|        |                               +---------------+
|        |
|        |                               +---------------+
|        |--(C)-- Authorization Grant -->| Authorization |
| Client |                               |     Server    |
|        |<-(D)----- Access Token -------|               |
|        |                               +---------------+
|        |
|        |                               +---------------+
|        |--(E)----- Access Token ------>|    Resource   |
|        |                               |     Server    |
|        |<-(F)--- Protected Resource ---|               |
+--------+                               +---------------+
```

上图所示的抽象 OAuth 2.0 流程描述了四个角色之间的交互，包括以下步骤:

1. 客户端向资源所有者申请授权。授权请求可以直接向资源所有者发出（如图所示），或者最好通过作为中介的授权服务器间接发出。
2. 客户端接收授权许可，这是代表资源所有者授权的凭证，使用本规范中定义的四种许可类型之一或使用扩展许可类型来表示。授权类型取决于客户端申请授权的方法以及授权服务器支持的类型。
3. 客户端通过与授权服务器进行身份验证并提供授权许可来申请访问令牌。
4. 授权服务器对客户端进行身份验证并验证授权，如果有效，则颁发访问令牌。
5. 客户端从资源服务器请求受保护的资源，并通过显示访问令牌进行身份验证。
6. 资源服务器验证访问令牌，如果有效，则为请求提供服务。

### 四种授权模式

授权许可是代表资源所有者的授权 (访问其受保护的资源) 的凭证，客户端使用该凭证来获取访问令牌。该规范定义了四种授权模式——**授权码模式**、**简化模式**、**密码模式**和**客户端模式**——以及用于定义其他类型的可扩展性机制。

#### 授权码模式

> [RFC 6749 - The OAuth 2.0 Authorization Framework | 4.1 Authorization Code Grant](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1)

> [!info]
>
> 授权码是通过使用授权服务器作为客户端和资源所有者之间的中介来获得的。客户端不是直接向资源所有者申请授权，而是将资源所有者引导到授权服务器 (通过 [RFC2616](https://datatracker.ietf.org/doc/html/rfc2616) 中定义的用户代理)，授权服务器又将资源所有者引导回带有授权码的客户端。在使用授权码将资源所有者引导回客户端之前，授权服务器对资源所有者进行身份验证并获得授权。因为资源所有者只向授权服务器进行身份验证，所以资源所有者的凭据永远不会与客户端共享。授权码提供了一些重要的安全优势，例如对客户端进行身份验证的能力，以及将访问令牌直接传输到客户端，而无需通过资源所有者的用户代理进行传递，也不会将其暴露给其他人，包括资源所有者。

授权码模式用于获取访问令牌和刷新令牌，并针对机密客户端进行了优化。因为这是一个基于重定向的流，所以客户端必须能够与资源所有者的用户代理 (通常是 web 浏览器) 进行交互，并且能够接收来自授权服务器的传入请求 (通过重定向)。如下图所示：[RFC 6749 - The OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1)

```
+----------+
| Resource |
|   Owner  |
|          |
+----------+
		^
		|
	 (B)
+----|-----+          Client Identifier      +---------------+
|         -+----(A)-- & Redirection URI ---->|               |
|  User-   |                                 | Authorization |
|  Agent  -+----(B)-- User authenticates --->|     Server    |
|          |                                 |               |
|         -+----(C)-- Authorization Code ---<|               |
+-|----|---+                                 +---------------+
 |    |                                          ^      v
(A)  (C)                                         |      |
 |    |                                          |      |
 ^    v                                          |      |
+---------+                                      |      |
|         |>---(D)-- Authorization Code ---------'      |
|  Client |          & Redirection URI                  |
|         |                                             |
|         |<---(E)----- Access Token -------------------'
+---------+       (w/ Optional Refresh Token)
```

> [!attention]
> 注意: 说明步骤 (A)、(B) 和 (C) 的线条在通过用户代理时分为两部分。

图上所示的流程包括以下步骤：

1. 客户端通过将资源所有者的用户代理定向到 **授权端点（`/oauth/authorize`）** 来启动流。客户端包括其<u>客户端标识符</u>、<u>请求的范围</u>、<u>本地状态</u>和一个<u>重定向 URI</u>，授权服务器在授予 (或拒绝) 访问权限后将向其发送用户代理。
2. 授权服务器对资源所有者进行身份验证 (通过用户代理)，并确定资源所有者是同意还是拒绝客户端的访问请求。
3. 假设资源所有者授予访问权限，授权服务器将使用先前提供的重定向 URI（在请求中或在客户端注册期间）将用户代理重定向回客户端。重定向 URI 包含一个**授权码**和客户端先前提供的任何本地状态。这一步是在客户端的后台的服务器上完成的，对用户不可见。
4. 客户端通过在上一步中获取到的授权码，向授权服务器的 **令牌端点（`/oauth/token`）** 请求（包含<u>重定向 URI</u> 以及<u>授权码</u>）访问令牌。
5. 授权服务器对客户端进行身份验证，验证授权码，并确保接收到的重定向 URI 与步骤 (C) 中用于重定向客户端的 URI 相匹配。如果有效，授权服务器将使用**访问令牌**和**刷新令牌 (可选)** 进行响应。

##### 授权请求

> [RFC 6749 - The OAuth 2.0 Authorization Framework | 4.1.1 Authorization Request](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1.1)

客户端通过使用 "application/x-www-form-urlencoded" 格式向授权端点 URI 的查询参数添加以下参数来构造请求 URI：

| 参数 | 描述 | 是否必须 |
| ---- | ---- | --------- |
| response_type | 值必须设置为 `code` | REQUIRED |
| client_id | 已在授权服务器中注册的客户端的唯一标识 [Section 2.2](https://datatracker.ietf.org/doc/html/rfc6749#section-2.2) | REQUIRED |
| redirect_uri |   重定向 URI [Section 3.1.2](https://datatracker.ietf.org/doc/html/rfc6749#section-3.1.2)   | OPTIONAL |
| scope | 授权范围，如果与客户端申请的范围相同，可省略 [Section 3.3](https://datatracker.ietf.org/doc/html/rfc6749#section-3.3) | OPTIONAL |
| state | 表示客户端的当前状态，为预防 CSRF 攻击，请务必设定此值并进行严格检查，认证服务器会原封不动地返回这个值 | RECOMMENDED |

> [!example]
> 客户端指示用户代理发出 GET 请求：[http://localhost:8080/oauth/authorize?client_id=client1&response_type=code&scope=scope1&redirect_uri=https://www.baidu.com&state=abc](http://localhost:8080/oauth/authorize?client_id=client1&response_type=code&scope=scope1&redirect_uri=https://www.baidu.com&state=abc)

授权服务器验证该请求，以确保所有必需的参数都存在并且有效。如果请求有效，授权服务器对资源所有者进行身份验证，并获得授权决定 (通过询问资源所有者或通过其他方式建立批准)。

一旦做出决定，授权服务器将使用 HTTP 重定向响应或通过用户代理提供的其他方式将用户代理直接发送到提供的客户端重定向 URI。

##### 授权响应

> [RFC 6749 - The OAuth 2.0 Authorization Framework | 4.1.2 Authorization Response](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1.2)

如果资源所有者同意访问请求，授权服务器将返回一个授权码，并通过使用 "application/x-www-form-urlencode" 格式将以下参数添加到重定向 URI 的查询参数中，将其传递给客户端。

| 参数  | 描述                                                         | 是否必须                                           |
| ----- | ------------------------------------------------------------ | -------------------------------------------------- |
| code  | 授权服务器生成的授权码。授权码必须在发布后不久过期，以降低泄露的风险。建议授权码的最长生存期为 10 分钟。客户不得多次使用授权代码。如果授权码被多次使用，授权服务器必须拒绝该请求，并且应该撤销 (如果可能的话) 先前基于该授权码颁发的所有令牌。授权码被绑定到客户端标识符和重定向 URI。 | REQUIRED                                           |
| state | 表示客户端的当前状态，为预防 CSRF 攻击，请务必设定此值并进行严格检查，认证服务器会原封不动地返回这个值 | REQUIRED 如果客户端授权请求中存在 "state" 参数的话 |

> [!example]
> 授权服务器通过发送以下 HTTP 响应重定向到用户代理： https://www.baidu.com/?code=KVXJFW&state=abc

###### 错误响应

> [RFC 6749 - The OAuth 2.0 Authorization Framework | 4.1.2.1 Error Response](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1.2.1)

如果由于重定向 URI 缺失、无效或不匹配而导致请求失败，或者如果客户端标识符缺失或无效，则授权服务器应将该错误通知资源所有者，并且不得自动将用户代理重定向到无效的重定向 URI。

> [!example]
> 当缺少必须参数 `reponse_type` 时，将返回 https://www.baidu.com/?error=unsupported_response_type&error_description=Unsupported%20response%20types:%20%5B%5D&state=abc

如果资源所有者拒绝访问请求，或者请求由于缺少或无效的重定向 URI 以外的原因而失败，授权服务器通过使用 "application/x-www-form-urlencode" 格式向重定向 URI 的查询参数添加以下参数来通知客户端：

| 参数              | 描述                                                         | 是否必须                                           |
| ----------------- | ------------------------------------------------------------ | -------------------------------------------------- |
| error             | 错误代码                                                     | REQUIRED                                           |
| error_description | 提供附加信息的可读 ASCII 文本，用于帮助客户端开发人员了解发生的错误 | OPTIONAL                                           |
| error_uri         | 用错误信息标识人类可读的网页的 URI，用于向客户端开发人员提供有关错误的附加信息 | OPTIONAL                                           |
| state             | 表示客户端的当前状态，为预防 CSRF 攻击，请务必设定此值并进行严格检查，认证服务器会原封不动地返回这个值 | REQUIRED 如果客户端授权请求中存在 "state" 参数的话 |

常见的错误如下所示：

- invalid_request：请求缺少必需的参数，包含无效的参数值，多次包含某个参数，或者格式不正确；
- unauthorized_client：客户端无权使用此方法申请授权码；
- access_denied：资源所有者或授权服务器拒绝请求；
- unsupported_response_type：授权服务器不支持使用此方法申请授权码；授权请求缺少 `response_type` 参数时会出现该错误！
- invalid_scope：请求的作用域无效、未知或格式不正确；当授权请求中的 `scope` 参数值与客户端在授权服务器注册时指定的 `scope` 参数值不一致时会出现该错误！
- server_error：授权服务器遇到意外情况，无法完成请求。(此错误代码是必需的，因为 500 内部服务器错误 HTTP 状态代码无法通过 HTTP 重定向返回给客户端。)
- temporarily_unavailable：由于服务器临时过载或维护，授权服务器当前无法处理该请求。(此错误代码是必需的，因为 503 服务不可用 HTTP 状态代码无法通过 HTTP 重定向返回给客户端。)

> [!example]
> 当资源所有者拒绝请求时，将返回 https://www.baidu.com/?error=access_denied&error_description=User%20denied%20access&state=abc

##### 访问令牌请求

> [RFC 6749 - The OAuth 2.0 Authorization Framework | 4.1.3 Access Token Request](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1.3)

客户端通过使用 "application/x-www-form-urlencode" 格式发送以下参数向令牌端点发出请求，在 HTTP 请求实体主体中的字符编码为 UTF-8：

| 参数          | 描述                                                         | 是否必须 |
| ------------- | ------------------------------------------------------------ | -------- |
| grant_type    | 值必须设置为 `authorization_code`                            | REQUIRED |
| code          | 从授权服务器获取到的**授权码**                               | REQUIRED |
| redirect_uri  | 重定向 URI，必须与授权请求中的该参数值保持一致                                                   | REQUIRED |
| client_id     | 客户端唯一标识                                               | REQUIRED |
| client_secret | 客户端密钥，需要带上该参数，用于验证客户端身份，否则会报错 { "error": "invalid_client","error_description": "Bad client credentials" } | REQUIRED |

> [!example]
> 客户端指示用户代理发出 POST 请求：[http://localhost:8080/oauth/token?client_id=client1&client_secret=123456&grant_type=authorization_code&code=3eLRrm&redirect_uri=https://www.baidu.com](http://localhost:8080/oauth/token?client_id=client1&client_secret=123456&grant_type=authorization_code&code=3eLRrm&redirect_uri=https://www.baidu.com)

授权服务器必须：

- 要求对机密客户端或任何已颁发客户端凭据的客户端进行客户端身份验证 (或具有其他身份验证要求) ；
- 如果包含客户端身份验证，则对客户端进行身份验证；
- 确保授权码已发给经过身份验证的机密客户端，或者如果客户端是公共的，则确保授权码已发给请求中的 "client_id"；
- 验证授权代码是否有效；
- 如果 "redirect_uri" 参数包含在第 [Section 4.1.1](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1.1) 节所述的初始授权请求中，则确保 "redirect_uri" 参数存在，并且如果包含了 "redirect_uri" 参数，则确保它们的值是相同的；

##### 访问令牌响应

如果访问令牌请求有效且经过授权，授权服务器将返回**访问令牌**和**刷新令牌**（可选），如 [Section 5.1](https://datatracker.ietf.org/doc/html/rfc6749#section-5.1) 节所述，如果请求客户端身份验证失败或无效，授权服务器将返回一个错误响应，如 [Section 5.2](https://datatracker.ietf.org/doc/html/rfc6749#section-5.2) 节所述。

一个成功的响应示例：

```
HTTP/1.1 200 
Cache-Control: no-store
Pragma: no-cache
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
X-Frame-Options: DENY
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Mon, 27 Nov 2023 13:08:38 GMT
Keep-Alive: timeout=60
Connection: keep-alive

{
  "access_token": "5bacb643-745e-42b0-b210-bd5c03d4c5d6",
  "token_type": "bearer",
  "refresh_token": "28ce9eb3-4129-4f42-8e6b-cb0be4d532c0",
  "expires_in": 3600,
  "scope": "scope1"
}
```

| 参数          | 描述                                                         | 是否必须   |
| ------------- | ------------------------------------------------------------ | ---------- |
| access_token  | 访问令牌                                                     | REQUIRED   |
| token_type    | 令牌类型，不区分大小写                                       | REQUIRED   |
| expires_in    | 访问令牌的过期时间（以秒为单位）。例如，值 "3600" 表示访问令牌将在响应生成后一小时过期。如果省略该参数，授权服务器应该通过其他方式设置过期时间或记录默认值。 | expires_in |
| refresh_token | 更新令牌，用于获取新的访问令牌而无需资源所有者再次授权       |            |
| scope         | 授权范围，如果与客户端申请的范围相同，可省略                 | OPTIONAL   |

#### 简化模式

> [RFC 6749 - The OAuth 2.0 Authorization Framework | 4.2 Implicit Grant](https://datatracker.ietf.org/doc/html/rfc6749#section-4.2)

简化模式是针对使用诸如 JavaScript 之类的脚本语言在浏览器中实现的客户端而优化的简化授权代码流。与授权码模式（客户端需要分别申请授权码和访问令牌）不同的是，在简化模式中，不是向客户端颁发授权码，而是直接向客户端颁发访问令牌（作为资源所有者授权的结果）。

由于这是一个基于重定向的流，客户端必须能够与资源所有者的用户代理 (通常是 Web 浏览器) 进行交互，并能够接收来自授权服务器的传入请求 (通过重定向)。

简化模式下，授权服务器不会对客户端进行身份验证，它依赖于资源所有者的存在和重定向 URI 的注册。由于访问令牌被编码到重定向 URI 中，因此它可能会向资源所有者和驻留在同一设备上的其他应用程序公开。

```
+----------+
| Resource |
|  Owner   |
|          |
+----------+
     ^
     |
    (B)
+----|-----+          Client Identifier     +---------------+
|         -+----(A)-- & Redirection URI --->|               |
|  User-   |                                | Authorization |
|  Agent  -|----(B)-- User authenticates -->|     Server    |
|          |                                |               |
|          |<---(C)--- Redirection URI ----<|               |
|          |          with Access Token     +---------------+
|          |            in Fragment
|          |                                +---------------+
|          |----(D)--- Redirection URI ---->|   Web-Hosted  |
|          |          without Fragment      |     Client    |
|          |                                |    Resource   |
|     (F)  |<---(E)------- Script ---------<|               |
|          |                                +---------------+
+-|--------+
 |    |
(A)  (G) Access Token
 |    |
 ^    v
+---------+
|         |
|  Client |
|         |
+---------+
```

> [!attention]
> 说明步骤（A）和（B）的线在通过用户代理时被分成两部分。

图上所示的流程包括以下步骤：

1. 客户端通过将资源所有者的用户代理定向到 **授权端点（`/oauth/authorize`）** 来启动流。客户端包括其<u>客户端标识符</u>、<u>请求的范围</u>、<u>本地状态</u>和一个<u>重定向 URI</u>，授权服务器在授予 (或拒绝) 访问权限后将向其发送用户代理。
2. 授权服务器对资源所有者进行身份验证 (通过用户代理)，并确定资源所有者是同意还是拒绝客户端的访问请求。
3. 假设资源所有者授予访问权限，授权服务器将使用先前提供的重定向 URI（在请求中或在客户端注册期间）将用户代理重定向回客户端。重定向 URI 将**访问令牌**包含在 URI 片段中。

> 摘抄自：[OAuth2.0 — 简化授权模式详解（ implicit grant type ） - Coding10](https://www.coding10.com/section/oauth2-implicit-grant-type)
>
> 关于简化授权模式的这部分我怎么都觉得很别扭，明明在 C 步骤已经获得了 Access Token，提取出来不就完了吗？后面的一堆流程就是请求了一段解析 Access Token 的脚本代码，也并没有在安全性、效率、功能以及认证方面有任何辅助性的操作。增加的这些流程只是增加了复杂度，降低了效率，所以完全可以取消这些流程，C 步骤结束后，自己写一段 Javascrip 解析代码提取 Access Token 即可。其实并不是说其他人都错了，大家只是忠实的翻译了一下 [RFC 6749]( https://www.rfcreader.com/#rfc6749 "OAuth 2 技术文档") 这篇技术文档中的流程罢了。
>
> 文档和规范终究也是人写的，简化授权模式的官方文档其实犯了一个错误，把技术人员想得太笨了，就像 OAuth 1.0 版本时犯的错误一样，把流程搞得复杂到令人发指，最后大家都不怎么买账，直到后来 OAuth 2.0 版本极大简化了授权机制和流程后，才得到广泛的认可和应用，但是这个模式 OAuth 2.0 还是把它搞复杂了，多了后续那些不必要的流程，如果有 OAuth 3.0，我想肯定不会再这么设计了。

##### 授权请求 ^implicit-authorize-request

> [RFC 6749 - The OAuth 2.0 Authorization Framework | 4.2.1 Authorization Request](https://datatracker.ietf.org/doc/html/rfc6749#section-4.2.1)

客户端通过使用 "application/x-www-form-urlencoded" 格式将以下参数添加到授权端点 URI 的查询参数来构建请求 URI：

| 参数          | 描述                                                         | 是否必须    |
| ------------- | ------------------------------------------------------------ | ----------- |
| response_type | 值必须为 `token`                                             | REQUIRED    |
| client_id     | 客户端唯一标识                                               | REQUIRED    |
| redirect_uri  | 重定向 URI                                                   | OPTIONAL    |
| scope         | 授权范围，如果与客户端申请的范围相同，可省略                                                     | OPTIONAL    |
| state         | 表示客户端的当前状态，为预防 CSRF 攻击，请务必设定此值并进行严格检查，认证服务器会原封不动地返回这个值。 | RECOMMENDED |

> [!example]
> 客户端指示用户代理发出 GET 请求：[http://localhost:8080/oauth/authorize?client_id=client1&response_type=token&scope=scope1&redirect_uri=https://www.baidu.com&state=abc](http://localhost:8080/oauth/authorize?client_id=client1&response_type=token&scope=scope1&redirect_uri=https://www.baidu.com&state=abc)

授权服务器验证请求，以确保所有必需的参数都存在且有效。授权服务器必须验证将访问令牌重定向到的重定向 URI 是否与客户端注册的重定向 URI 匹配，如 [Section 3.1.2](https://datatracker.ietf.org/doc/html/rfc6749#section-3.1.2) 节所述。

如果请求有效，授权服务器对资源所有者进行身份验证，并获得授权决定 (通过询问资源所有者或通过其他方式建立批准)。

一旦做出决定，授权服务器将使用 HTTP 重定向响应或通过用户代理提供的其他方式将用户代理直接发送到提供的客户端重定向 URI。

##### 授权响应

> [RFC 6749 - The OAuth 2.0 Authorization Framework | 4.2.2 Access Token Response](https://datatracker.ietf.org/doc/html/rfc6749#section-4.2.2)

如果资源所有者同意访问请求，则授权服务器返回一个访问令牌，并通过使用 "application/x-www-form-urlencoded" 格式将以下参数添加到重定向 URI 的片段部分，将该令牌传递给客户端。

| 参数         | 描述 | 是否必须                                           |
| ------------ | ---- | -------------------------------------------------- |
| access_token |  访问令牌    | REQUIRED                                           |
| token_type   |  令牌类型，不区分大小写    | REQUIRED                                           |
| expires_in   | 访问令牌的过期时间（以秒为单位）。例如，值 "3600" 表示访问令牌将在响应生成后一小时过期。如果省略该参数，授权服务器应该通过其他方式设置过期时间或记录默认值。 | expires_in                                         |
| scope        | 授权范围，如果与客户端申请的范围相同，可省略 | OPTIONAL                                           |
| state        | 表示客户端的当前状态，为预防 CSRF 攻击，请务必设定此值并进行严格检查，认证服务器会原封不动地返回这个值。 | REQUIRED 如果客户端授权请求中存在 "state" 参数的话 |

> [!important]
> 在此模式下授权服务器**不会生成刷新令牌**！

> [!example]
> 授权服务器通过发送以下 HTTP 响应重定向到用户代理：https://www.baidu.com/#access_token=209216a9-c49a-48cd-8105-0193f028b423&token_type=bearer&state=abc&expires_in=1909

#### 密码模式

> [RFC 6749 - The OAuth 2.0 Authorization Framework | 4.3 Resource Owner Password Credentials Grant](https://datatracker.ietf.org/doc/html/rfc6749#section-4.3)

> [!important]
> **密码模式适用于资源所有者与客户端具有信任关系的情况**

这种授权类型适用于能够获得资源所有者凭据 (用户名和密码，通常使用交互式表单) 的客户端。它还用于通过将存储的凭据转换为访问令牌，将使用 HTTP Basic 或 Digest 身份验证等直接身份验证方案的现有客户端迁移到 OAuth。

```
+----------+
| Resource |
|  Owner   |
|          |
+----------+
		v
		|    Resource Owner
	 (A) Password Credentials
		|
		v
+---------+                                  +---------------+
|         |>--(B)---- Resource Owner ------->|               |
|         |         Password Credentials     | Authorization |
| Client  |                                  |     Server    |
|         |<--(C)---- Access Token ---------<|               |
|         |    (w/ Optional Refresh Token)   |               |
+---------+                                  +---------------+
```

1. 资源所有者向客户端提供其用户名和密码；
2. 客户端通过从资源所有者接收的凭证，向授权服务器的 **令牌端点（`/oauth/token`）** 请求访问令牌。当发出请求时，客户端向授权服务器进行身份验证。
3. 授权服务器对客户端进行身份验证，并验证资源所有者凭证，如果有效，则颁发访问令牌；

##### 授权请求与响应

> [RFC 6749 - The OAuth 2.0 Authorization Framework | 4.3.1 Authorization Request and Response](https://datatracker.ietf.org/doc/html/rfc6749#section-4.3.1)

客户端获取资源所有者凭证的方法超出了本规范的范围。**一旦获得访问令牌，客户端必须丢弃凭证**。

##### 访问令牌请求 ^password-accesstoken-request

> [RFC 6749 - The OAuth 2.0 Authorization Framework | 4.3.2 Access Token Request](https://datatracker.ietf.org/doc/html/rfc6749#section-4.3.2)

客户端通过按照 "application/x- www-form-urlencoded" 格式在 HTTP 请求实体主体中添加以下参数，向 **令牌端点（`/oauth/token`）** 发出请求，字符编码为 UTF-8：

| 参数          | 描述                                                         | 是否必须         |
| ------------- | ------------------------------------------------------------ | ---------------- |
| grant_type    | 值必须为 `password`                                          | REQUIRED         |
| username      | 用户名                                                       | REQUIRED         |
| password      | 密码                                                         | REQUIRED         |
| scope         | 授权范围，如果与客户端申请的范围相同，可省略                 | OPTIONAL         |
| client_id     | 客户端唯一标识客户端唯一标识                                 | REQUIREDREQUIRED |
| client_secret | 客户端密钥，需要带上该参数，用于验证客户端身份，否则会报错 { "error": "invalid_client","error_description": "Bad client credentials" } | REQUIRED         |

如果客户端类型是保密的，或者客户端被颁发了客户端证书 (或者分配了其他身份验证要求) ，则客户端必须按照第 [Section 3.2.1](https://datatracker.ietf.org/doc/html/rfc6749#section-3.2.1) 节中的说明向授权服务器进行身份验证。可以在请求中添加 `client_id` 和 `client-secret` 两个参数。

> [!example]
> 客户端指示用户代理发出 POST 请求：[http://localhost:8080/oauth/token?client_id=client1&client_secret=123456&grant_type=password&username=admin&password=123456&scope=scope1](http://localhost:8080/oauth/token?client_id=client1&client_secret=123456&grant_type=password&username=admin&password=123456&scope=scope1)

授权服务器必须：

- 如果包含客户端身份验证，则对客户端进行身份验证；
- 使用其现有的密码验证算法来验证资源所有者密码凭据；

由于此访问令牌请求使用资源所有者的密码，授权服务器必须保护端点免受暴力攻击（例如，使用速率限制或生成警报）。

##### 访问令牌响应

> [RFC 6749 - The OAuth 2.0 Authorization Framework | 4.3.3 Access Token Response](https://datatracker.ietf.org/doc/html/rfc6749#section-4.3.3)

与 [授权码模式 | 访问令牌响应](#访问令牌响应) 内容一致，不再赘叙。

#### 客户端模式

> [RFC 6749 - The OAuth 2.0 Authorization Framework | 4.4 Client Credentials Grant](https://datatracker.ietf.org/doc/html/rfc6749#section-4.4)

```
+---------+                                  +---------------+
|         |                                  |               |
|         |>--(A)- Client Authentication --->| Authorization |
| Client  |                                  |     Server    |
|         |<--(B)---- Access Token ---------<|               |
|         |                                  |               |
+---------+                                  +---------------+
```

图上所示的流程包括以下步骤：

1. 客户端向授权服务器进行身份验证，并向 **令牌端点（`/oauth/token`）** 请求访问令牌；
2. 授权服务器对客户端进行身份验证，如果有效，则颁发访问令牌；

##### 授权请求与响应

> [RFC 6749 - The OAuth 2.0 Authorization Framework | 4.4.1 Authorization Request and Response](https://datatracker.ietf.org/doc/html/rfc6749#section-4.4.1)

因为客户端身份验证被用作授权许可，所以不需要额外的授权请求。

##### 访问令牌请求 ^client-credentials-accesstoken-request

> [RFC 6749 - The OAuth 2.0 Authorization Framework | 4.4.2 Access Token Request](https://datatracker.ietf.org/doc/html/rfc6749#section-4.4.2)

客户端通过按照 "application/x-www-form-urlencoded" 格式在 HTTP 请求实体主体中添加以下参数，向 **令牌端点（`/oauth/token`）** 发出请求，字符编码为 UTF-8：

| 参数          | 描述                                                         | 是否必须 |
| ------------- | ------------------------------------------------------------ | -------- |
| grant_type    | 值必须为 `client_credentials`                                | REQUIRED |
| scope         | 授权范围，如果与客户端申请的范围相同，可省略                 | OPTIONAL |
| client_id     | 客户端唯一标识客户端唯一标识                                 | REQUIRED |
| client_secret | 客户端密钥，需要带上该参数，用于验证客户端身份，否则会报错 { "error": "invalid_client","error_description": "Bad client credentials" } | REQUIRED |

> [!example]
> 客户端指示用户代理发出 POST 请求：[http://localhost:8080/oauth/token?client_id=client1&client_secret=123456&grant_type=client_credentials&scope=scope1](http://localhost:8080/oauth/token?client_id=client1&client_secret=123456&grant_type=client_credentials&scope=scope1)

##### 访问令牌响应

> [RFC 6749 - The OAuth 2.0 Authorization Framework | 4.3.3 Access Token Response](https://datatracker.ietf.org/doc/html/rfc6749#section-4.4.3)

与 [授权码模式 | 访问令牌响应](#访问令牌响应) 内容一致，只不过**不包含刷新令牌**而已！

### 标准接口

- **`/oauth/authorize`：授权端点**
- **`/oauth/token`：令牌端点**
- `/oauth/confirm_access`：用户确认授权提交端点
- `/oauth/error`：授权服务错误信息端点
- `/oauth/check_token`：用于资源服务访问的令牌解析端点
- `/oauth/token_key`：提供公钥的端点，如果使用 JWT 令牌的话

## 案例演示

| 模块                         | 端口 | 备注                                               |
| ---------------------------- | ---- | -------------------------------------------------- |
| spring-security-oauth2-study |      | 父模块，用于聚合子模块，管理版本依赖，提供公共依赖 |
| authorization-server         | 8080 | 授权服务器                                         |
| resource-server             | 8081 | 资源服务器                                         |

### 创建项目

创建 `spring-security-oauth2-study` 项目（父模块），确定 SpringBoot 版本为 2.2.4.RELEASE，SpringCloud 版本为 Hoxton.SR1，其 `pom.xml` 配置文件如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xmlns="http://maven.apache.org/POM/4.0.0"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
    <parent>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-parent</artifactId>  
        <version>2.2.4.RELEASE</version>  
        <relativePath/> <!-- lookup parent from repository -->  
    </parent>  
  
    <groupId>fun.xiaorang</groupId>  
    <artifactId>spring-security-oauth2-study</artifactId>  
    <version>1.0-SNAPSHOT</version>  
    <packaging>pom</packaging>  
    <modules>  
        <module>authorization-server</module>  
    </modules>  
  
    <properties>  
        <java.version>11</java.version>  
        <maven.compiler.source>${java.version}</maven.compiler.source>  
        <maven.compiler.target>${java.version}</maven.compiler.target>  
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>  
        <spring-cloud.version>Hoxton.SR1</spring-cloud.version>  
    </properties>  
  
    <dependencyManagement>  
        <dependencies>  
            <dependency>  
                <groupId>org.springframework.cloud</groupId>  
                <artifactId>spring-cloud-dependencies</artifactId>  
                <version>${spring-cloud.version}</version>  
                <type>pom</type>  
                <scope>import</scope>  
            </dependency>  
        </dependencies>  
    </dependencyManagement>  
</project>
```

### 授权服务器

创建 `authorization-server` 授权服务器模块，添加 `spring-boot-starter-web` 和 `spring-cloud-starter-oauth2` 依赖，其 `pom.xml` 配置如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xmlns="http://maven.apache.org/POM/4.0.0"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
    <parent>  
        <groupId>fun.xiaorang</groupId>  
        <artifactId>spring-security-oauth2-study</artifactId>  
        <version>1.0-SNAPSHOT</version>  
    </parent>  
  
    <artifactId>authorization-server</artifactId>  
  
    <dependencies>  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-web</artifactId>  
        </dependency>  
        <dependency>  
            <groupId>org.springframework.cloud</groupId>  
            <artifactId>spring-cloud-starter-oauth2</artifactId>  
        </dependency>  
    </dependencies>  
</project>
```

项目创建完成之后，首先提供一个 SpringSecurity 的基本配置 `SecurityConfig`：

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable().formLogin();
    }

    @Bean
    public UserDetailsService userDetailsService() {
        final InMemoryUserDetailsManager inMemoryUserDetailsManager = new InMemoryUserDetailsManager();
        final UserDetails admin = User
                .withUsername("admin")
                .password(passwordEncoder().encode("123456"))
                .roles("ADMIN")
                .build();
        inMemoryUserDetailsManager.createUser(admin);
        final UserDetails user = User
                .withUsername("user")
                .password(passwordEncoder().encode("123456"))
                .roles("USER")
                .build();
        inMemoryUserDetailsManager.createUser(user);
        return inMemoryUserDetailsManager;
    }
}
```

在上述代码中，为了方便测试，就不把 SpringSecurity 用户存到数据库中去了，直接存在内存中。这里我创建了 `admin` 和 `user` 两个用户，角色分别为 `ADMIN` 和 `USER`。同时还配置了表单登录。

#### 基于内存的客户端和令牌存储

首先咱们提供了一个 `TokenStore` 的实例，这个是指你生成的 Token 要往哪里存储，咱们可以存在 Redis 中，也可以存在内存中，也可以结合 JWT 等等，这里，咱们就先把它存在内存中，所以提供一个 `InMemoryTokenStore` 的实例即可。

```java
@Configuration  
public class TokenConfig {  
    @Bean  
    public TokenStore tokenStore() {  
        return new InMemoryTokenStore();  
    }  
}
```

接下来创建授权服务配置类 `AuthorizationServerConfig` 继承自 `AuthorizationServerConfigurerAdapter`，来对授权服务器做进一步的详细配置，类上记得加 `@EnableAuthorizationServer` 注解，表示开启授权服务器的自动化配置。

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
    private final PasswordEncoder passwordEncoder;

    public AuthorizationServerConfig(
            final PasswordEncoder passwordEncoder) {
        this.passwordEncoder = passwordEncoder;
    }

    @Override
    public void configure(final AuthorizationServerSecurityConfigurer security) throws Exception {
        security.checkTokenAccess("permitAll()")
                .allowFormAuthenticationForClients();
    }

    @Override
    public void configure(final ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("client1")
                .secret(passwordEncoder.encode("123456"))
                .resourceIds("res1")
                .authorizedGrantTypes("authorization_code", "refresh_token", "password", "implicit", "client_credentials")
                .scopes("scope1")
                .redirectUris("https://www.baidu.com");
    }

    @Override
    public void configure(final AuthorizationServerEndpointsConfigurer endpoints) throws Exception {

    }
}
```

- 在 `AuthorizationServerConfig` 配置类中，主要重写父类 `AuthorizationServerConfigurerAdapter` 中的 3 个 `configure` 方法：
	- `AuthorizationServerSecurityConfigurer` 用来配置令牌端点的安全约束，也就是这个端点谁能访问，谁不能访问。checkTokenAccess 是指 Token 校验的端点，这个端点咱们设置为可以直接访问（因为在后面，当资源服务器收到 Token 之后，需要去校验 Token 的合法性，就会访问这个端点）。
	- `ClientDetailsServiceConfigurer` 用来配置客户端的详细信息。在上面的简介当中已经讲过，授权服务器要做两方面的检验，一方面是**校验客户端**，另一方面则是**校验用户（咱们前面已经在 `SecurityConfig` 中配置过了）**，这里就是配置校验客户端。客户端的信息咱们可以存在数据库中，这其实也是比较容易的，和用户信息存到数据库中类似，但是这里为了简化代码，还是将客户端信息存在内存中，这里咱们分别配置了客户端的 id，secret、资源 id、授权模式、授权范围以及重定向 uri。授权模式已经在 [前面](#四种授权模式) 和大家一共讲了四种，四种之中并不包含 refresh_token 这种模式，但是在实际操作中，refresh_token 也被算作一种。
	- `AuthorizationServerEndpointsConfigurer` 这里用来配置令牌的访问端点和令牌服务。`AuthorizationCodeServices` 和 `AuthorizationServerTokenServices` 分别用来配置授权码和访问令牌的存储方式，默认情况下，它们俩都存储在内存中。

> [!todo]- 使用 `@EnableAuthorizationServer` 注解开启授权服务器的自动化配置，默认情况下到底都有哪些配置？
>
> ```java
> @Target(ElementType.TYPE)
> @Retention(RetentionPolicy.RUNTIME)
> @Documented
> @Import({AuthorizationServerEndpointsConfiguration.class, AuthorizationServerSecurityConfiguration.class})
> public @interface EnableAuthorizationServer {
> 
> }
> ```

现在启动授权服务器，测试 使用授权码模式**申请授权码**和**申请访问令牌**以及**刷新访问令牌**的整体流程，如下所示：

1. 申请授权码：
   1. 浏览器访问授权服务器的授权端点（/oauth/authorize），发送 [授权请求](#授权请求)： http://localhost:8080/oauth/authorize?client_id=client1&response_type=code&scope=scope1&redirect_uri=https://www.baidu.com&state=abc ，其中的 redirect_uri、scope 和 state 参数不是必须的；
   2. 出现如下所示 SpringSecurity 默认的登录表单，输入用户名 `admin` 和密码 `123456`，认证成功！<br /> ![](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202311241728359.png)
   3. 出现如下所示授权界面：选择 `Approve` 同意，然后点击 `Authorize` 授权按钮。<br /> ![](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202311241552057.png)
   4. 重定向到配置的百度首页 `https://www.baidu.com/?code=5vCHSO`，可以发现请求路径上带着返回的<span style="background:rgba(255, 183, 139, 0.55)">授权码</span> `code`，获取到授权码之后就可以拿着该授权码去申请<span style="background:rgba(240, 167, 216, 0.55)">访问令牌</span> ` token `。
2. 申请访问令牌：发送 [访问令牌请求](#访问令牌请求)： POST http://localhost:8080/oauth/token?client_id=client1&client_secret=123456&grant_type=authorization_code&code=8EsZtG&redirect_uri=https://www.baidu.com ，其中授权码 `code` 参数的值替换成上一步申请到的授权码即可。响应结果如下所示：

   ```json
   {
     "access_token": "6cac987e-d07b-459b-ad3d-26d7988d913a",
     "token_type": "bearer",
     "refresh_token": "06da0a50-4965-4e38-99ff-522ca4b6b8b5",
     "expires_in": 7199,
     "scope": "scope1"
   }
   ```

   > [!attention]
   >
   > **一个授权码只能使用一次！！！**如果再次使用该授权码去申请访问令牌的话，则会报 { "error": "invalid_grant", "error_description": "Invalid authorization code: k6zoSY" } 错误！

3. 刷新访问令牌：发送 POST 请求： http://localhost:8080/oauth/token?client_id=client1&client_secret=123456&grant_type=refresh_token&refresh_token=06da0a50-4965-4e38-99ff-522ca4b6b8b5 ，其中刷新令牌 `refresh_token` 参数的值替换成上面申请访问令牌响应中对应的值即可。响应结果如下所示：

   ```json
   {
     "error": "server_error",
     "error_description": "Internal Server Error"
   }
   ```

   抛出上述错误，控制台提示如下信息：

   ```
   2023-11-29 11:39:55.519  WARN 6520 --- [nio-8080-exec-2] o.s.s.o.provider.endpoint.TokenEndpoint  : Handling error: IllegalStateException, UserDetailsService is required.
   ```

   大家可以自行调试，将断点打在 `TokenEndpoint` 类中的 `postAccessToken()` 方法中，该方法用于生成和刷新访问令牌的。经调试发现，由于其中的 `defaultUserDetailsService` 为 NULL，才导致这个错误的发生！因此存在如下解决方案：

   修改授权服务配置类 `AuthorizationServerConfig` 中的第三个 `configure` 方法配置，添加 `UserDetailsService`。如下所示：

   ```java
   @Configuration
   @EnableAuthorizationServer
   public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
       ...
       
       private final UserDetailsService userDetailsService;
       
       ...
       
       @Override
       public void configure(final AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
           endpoints.userDetailsService(userDetailsService);
       }
   }
   ```

   重新启动项目，再次测试，响应结果如下所示：

   ```json
   {
     "access_token": "7157a890-8942-4078-9a67-3d0f76683d7d",
     "token_type": "bearer",
     "refresh_token": "06da0a50-4965-4e38-99ff-522ca4b6b8b5",
     "expires_in": 7200,
     "scope": "scope1"
   }
   ```

   > [!attention]
   >
   > **使用刷新令牌生成新的访问令牌之后，以前的访问令牌不再有效**！

---

> [!demo]+ 测试使用密码模式**申请访问令牌**和**刷新访问令牌**的整体流程
>
> 1. 申请访问令牌：发送 [访问令牌请求](#访问令牌请求%20password-accesstoken-request)：POST [http://localhost:8080/oauth/token?client_id=client1&client_secret=123456&grant_type=password&username=admin&password=123456&scope=scope1](http://localhost:8080/oauth/token?client_id=client1&client_secret=123456&grant_type=password&username=admin&password=123456&scope=scope1)，其中除了 `scope` 参数之外其他参数都是必须的。响应结果如下所示：
>
>    ```json
>    {
>      "error": "unsupported_grant_type",
>      "error_description": "Unsupported grant type: password"
>    }
>    ```
>
>    抛出上述错误，说不支持密码模式，该怎么办呢？完善原有代码使当前客户端支持密码模式：
>
>    1. 修改 SpringSecurity 基础配置类 `SecurityConfig`：将 `AuthenticationManager` 实例对象暴露出来，注册到容器中
>
>       ```java
>       @Configuration
>       public class SecurityConfig extends WebSecurityConfigurerAdapter {
>           ...
>
>           @Bean
>           @Override
>           public AuthenticationManager authenticationManagerBean() throws Exception {
>               return super.authenticationManagerBean();
>           }
>
>           ...
>       }
>       ```
>
>    2. 修改原来的授权服务配置类 `AuthorizationServerConfig`
>
>       ```java
>       @Configuration
>       @EnableAuthorizationServer
>       public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
>           ...
>                                                                                                                                                                                                                                                    
>           private final AuthenticationManager authenticationManager;
>                                                                                                                                                                                                                                                    
>           @Override
>           public void configure(final ClientDetailsServiceConfigurer clients) throws Exception {
>               clients.inMemory()
>                       .withClient("client1")
>                       .secret(passwordEncoder.encode("123456"))
>                       .resourceIds("res1")
>                       .authorizedGrantTypes("authorization_code", "refresh_token", "password")
>                       .scopes("scope1")
>                       .redirectUris("https://www.baidu.com");
>           }
>                                                                                                                                                                                                                                                    
>           @Override
>           public void configure(final AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
>               endpoints
>                       .userDetailsService(userDetailsService)
>                       .authenticationManager(authenticationManager);
>           }
>                                                                                                                                                                                                                                           
>                    ...
>       }
>       ```
>
>   重新启动项目，再次发送请求，发现可以正常返回访问令牌以及刷新令牌：
>
>   ```json
>    {
>      "access_token": "afff4bc7-7084-48b2-9744-ec7696616e42",
>      "token_type": "bearer",
>      "refresh_token": "5e8fe13a-a7c4-4f49-acd3-8675c4c37325",
>      "expires_in": 7199,
>      "scope": "scope1"
>    }
>   ```
>
>2. 刷新访问令牌：发送 POST 请求：[http://localhost:8080/oauth/token?client_id=client1&client_secret=123456&grant_type=refresh_token&refresh_token=5e8fe13a-a7c4-4f49-acd3-8675c4c37325](http://localhost:8080/oauth/token?client_id=client1&client_secret=123456&grant_type=refresh_token&refresh_token=5e8fe13a-a7c4-4f49-acd3-8675c4c37325)，其中刷新令牌 `refresh_token` 参数的值替换成上面申请访问令牌响应中对应的值即可。响应结果如下所示：
>
>   ```json
>    {
>      "access_token": "eb3e81cc-4107-431d-aaf3-f220df10b99a",
>      "token_type": "bearer",
>      "refresh_token": "5e8fe13a-a7c4-4f49-acd3-8675c4c37325",
>      "expires_in": 7200,
>      "scope": "scope1"
>    }
>   ```

> [!demo]+ 测试使用简化模式申请访问令牌的整体流程
>
> 在测试之前，首先使当前客户端支持简化模式，否则的话会抛出 `error=invalid_client&error_description=Unauthorized grant type: implicit` 错误！
>
> ```java
> @Override
> public void configure(final ClientDetailsServiceConfigurer clients) throws Exception {
>  clients.inMemory()
>          .withClient("client1")
>          .secret(passwordEncoder.encode("123456"))
>          .resourceIds("res1")
>          .authorizedGrantTypes("authorization_code", "refresh_token", "password", "implicit")
>          .scopes("scope1")
>          .redirectUris("https://www.baidu.com");
> }
> ```
>
> 1. 浏览器访问授权服务器的授权端点（/oauth/authorize），发送 [授权请求](#授权请求%20implicit-authorize-request)： [http://localhost:8080/oauth/authorize?client_id=client1&response_type=token&scope=scope1&redirect_uri=https://www.baidu.com&state=abc](http://localhost:8080/oauth/authorize?client_id=client1&response_type=token&scope=scope1&redirect_uri=https://www.baidu.com&state=abc) ，其中的 redirect_uri、scope 和 state 参数不是必须的；
>
> 2. 出现如下所示 SpringSecurity 默认的登录表单，输入用户名 `admin` 和密码 `123456`，认证成功！<br /> ![](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202311241728359.png)
> 3. 出现如下所示授权界面：选择 `Approve` 同意，然后点击 `Authorize` 授权按钮。<br /> ![](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202311241552057.png)
> 4. 重定向到配置的百度首页 `https://www.baidu.com/#access_token=1a99269f-aff4-4b48-9c01-cf0f61d25305&token_type=bearer&state=abc&expires_in=7199`，可以发现请求路径上带着返回的<span style="background:rgba(255, 183, 139, 0.55)">访问令牌</span> `access_token`。
>
> > [!attention]
> >
> > 与授权码模式不同的是，在简化模式中，不是向客户端颁发授权码，而是**直接向客户端颁发访问令牌**（作为资源所有者授权的结果），在此模式下授权服务器**不会生成刷新令牌**！

> [!demo]+ 测试使用客户端模式申请访问令牌的整体流程
>
> 在测试之前，首先使当前客户端支持客户端模式，否则的话会抛出 `error=invalid_client&error_description=Unauthorized grant type: client_credentials` 错误！
>
> ```java
> @Override
> public void configure(final ClientDetailsServiceConfigurer clients) throws Exception {
> clients.inMemory()
>       .withClient("client1")
>       .secret(passwordEncoder.encode("123456"))
>       .resourceIds("res1")
>       .authorizedGrantTypes("authorization_code", "refresh_token", "password", "implicit", "client_credentials")
>       .scopes("scope1")
>       .redirectUris("https://www.baidu.com");
> }
> ```
>
> 发送 [访问令牌请求](#访问令牌请求%20client-credentials-accesstoken-request)： POST [http://localhost:8080/oauth/token?client_id=client1&client_secret=123456&grant_type=client_credentials&scope=scope1](http://localhost:8080/oauth/token?client_id=client1&client_secret=123456&grant_type=client_credentials&scope=scope1)，其中除了 `scope` 参数之外其他参数都是必须的。响应结果如下所示：在此模式下授权服务器同样**不会生成刷新令牌**！
>
> ```json
> {
> "access_token": "6b3351e1-ba0a-4621-950b-8edf4760d349",
> "token_type": "bearer",
> "expires_in": 7199,
> "scope": "scope1"
> }
> ```

#### 基于数据库的客户端和令牌存储

在上面的案例中，`TokenStore` 以及 `ClientDetialService` 的默认实现分别为 `InMemoryTokenStore` 和 `InMemoryClientDetailsService`，均表示基于内存存储，如果想基于数据库进行存储，只需提供这些接口的数据库实现类即可，而框架已经为咱们写好了 `JdbcTokenStore` 和 `JdbcClientDetailsService` 实现类。

建表语句：[schema.sql](https://github.com/spring-attic/spring-security-oauth/blob/main/spring-security-oauth2/src/test/resources/schema.sql)

> [!attention]
> 需要使用 BLOB 替换语句中的 LONGVARBINARY 类型！如下所示：
>
> ```sql
> SET NAMES utf8mb4;
> SET FOREIGN_KEY_CHECKS = 0;
> 
> -- ----------------------------
> -- Table structure for clientdetails
> -- ----------------------------
> DROP TABLE IF EXISTS `clientdetails`;
> CREATE TABLE `clientdetails`  (
>   `appId` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
>   `resourceIds` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `appSecret` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `scope` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `grantTypes` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `redirectUrl` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `authorities` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `access_token_validity` int NULL DEFAULT NULL,
>   `refresh_token_validity` int NULL DEFAULT NULL,
>   `additionalInformation` varchar(4096) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `autoApproveScopes` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   PRIMARY KEY (`appId`) USING BTREE
> ) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;
> 
> -- ----------------------------
> -- Table structure for oauth_access_token
> -- ----------------------------
> DROP TABLE IF EXISTS `oauth_access_token`;
> CREATE TABLE `oauth_access_token`  (
>   `token_id` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `token` blob NULL,
>   `authentication_id` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
>   `user_name` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `client_id` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `authentication` blob NULL,
>   `refresh_token` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   PRIMARY KEY (`authentication_id`) USING BTREE
> ) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;
> 
> -- ----------------------------
> -- Table structure for oauth_approvals
> -- ----------------------------
> DROP TABLE IF EXISTS `oauth_approvals`;
> CREATE TABLE `oauth_approvals`  (
>   `userId` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `clientId` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `scope` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `status` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `expiresAt` timestamp NULL DEFAULT NULL,
>   `lastModifiedAt` timestamp NULL DEFAULT NULL
> ) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;
> 
> -- ----------------------------
> -- Table structure for oauth_client_details
> -- ----------------------------
> DROP TABLE IF EXISTS `oauth_client_details`;
> CREATE TABLE `oauth_client_details`  (
>   `client_id` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
>   `resource_ids` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `client_secret` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `scope` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `authorized_grant_types` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `web_server_redirect_uri` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `authorities` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `access_token_validity` int NULL DEFAULT NULL,
>   `refresh_token_validity` int NULL DEFAULT NULL,
>   `additional_information` varchar(4096) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `autoapprove` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   PRIMARY KEY (`client_id`) USING BTREE
> ) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;
> 
> -- ----------------------------
> -- Table structure for oauth_client_token
> -- ----------------------------
> DROP TABLE IF EXISTS `oauth_client_token`;
> CREATE TABLE `oauth_client_token`  (
>   `token_id` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `token` blob NULL,
>   `authentication_id` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
>   `user_name` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `client_id` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   PRIMARY KEY (`authentication_id`) USING BTREE
> ) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;
> 
> -- ----------------------------
> -- Table structure for oauth_code
> -- ----------------------------
> DROP TABLE IF EXISTS `oauth_code`;
> CREATE TABLE `oauth_code`  (
>   `code` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `authentication` blob NULL
> ) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;
> 
> -- ----------------------------
> -- Table structure for oauth_refresh_token
> -- ----------------------------
> DROP TABLE IF EXISTS `oauth_refresh_token`;
> CREATE TABLE `oauth_refresh_token`  (
>   `token_id` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
>   `token` blob NULL,
>   `authentication` blob NULL
> ) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;
> 
> SET FOREIGN_KEY_CHECKS = 1;
> ```
>
> 插入一条客户端信息（与上面基于内存存储案例中的客户端信息一致），如下所示：
>
> ```sql
> INSERT INTO `oauth_client_details` VALUES('client1', 'res1', '$2a$10$KwqBGLnnEIsp4pKay8mWHu.SLfwRNmMYDIpCfYk6dWXNCBROB4jNK', 'scope1', 'authorization_code,refresh_token,password,implicit,client_credentials', 'https://www.baidu.com', NULL, NULL, NULL, NULL, NULL);
> ```

添加依赖：

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

编写配置文件：

```yaml
server:
  port: 8080

spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://localhost:3306/oauth?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true
    driver-class-name: com.mysql.cj.jdbc.Driver
```

注释掉原来的授权服务配置类 `AuthorizationServerConfig`，编写一个新的使用数据库进行存储的配置类 `JdbcAuthorizationServerConfig`，代码实现如下所示：

```java
@Configuration
@EnableAuthorizationServer
public class JdbcAuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
    private final AuthenticationManager authenticationManager;
    private final DataSource dataSource;

    public JdbcAuthorizationServerConfig(
            final AuthenticationManager authenticationManager,
            final DataSource dataSource) {
        this.authenticationManager = authenticationManager;
        this.dataSource = dataSource;
    }

    @Override
    public void configure(final AuthorizationServerSecurityConfigurer security) throws Exception {
        security.checkTokenAccess("permitAll()")
                .allowFormAuthenticationForClients();
    }

    @Override
    public void configure(final ClientDetailsServiceConfigurer clients) throws Exception {
        clients.withClientDetails(clientDetailsService());
    }

    @Override
    public void configure(final AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints
                // 密码模式需要使用，否则会报错
                .authenticationManager(authenticationManager)
                // 授权码存储方式
                .authorizationCodeServices(authorizationCodeServices())
                // 令牌存储方式
                .tokenServices(tokenServices());
    }

    @Bean
    public ClientDetailsService clientDetailsService() {
        return new JdbcClientDetailsService(dataSource);
    }

    @Bean
    public AuthorizationCodeServices authorizationCodeServices() {
        return new JdbcAuthorizationCodeServices(dataSource);
    }

    @Bean
    public TokenStore tokenStore() {
        return new JdbcTokenStore(dataSource);
    }

    @Bean
    public AuthorizationServerTokenServices tokenServices() {
        final DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
        defaultTokenServices.setClientDetailsService(clientDetailsService());
        defaultTokenServices.setTokenStore(tokenStore());
        defaultTokenServices.setSupportRefreshToken(true);
        defaultTokenServices.setAccessTokenValiditySeconds(60 * 60 * 24 * 2);
        defaultTokenServices.setRefreshTokenValiditySeconds(60 * 60 * 24 * 7);
        return defaultTokenServices;
    }
}
```

启动项目，发现报错：在 `ClientDetailsServiceConfiguration` 配置类中存在一个名称相同的 `ClientDetailsService` 组件定义，提示需要组件修改名称或者配置 `spring.main.allow-bean-definition-overriding` 属性值为 `true`，允许覆盖组件定义。

> [!error]
>
> Description:
>
> The bean 'clientDetailsService', defined in class path resource [fun/xiaorang/oauth2/config/JdbcAuthorizationServerConfig.class], could not be registered. A bean with that name has already been defined in BeanDefinition defined in class path resource [org/springframework/security/oauth2/config/annotation/configuration/ClientDetailsServiceConfiguration.class] and overriding is disabled.
>
> Action:
>
> Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true

> [!faq]+ 错误原因分析&解决方案
>
> 咱们查看 `ClientDetailsServiceConfiguration` 配置类：发现其不仅将 `ClientDetailsServiceConfigurer` 配置器注册到 Spring 容器中，还通过该配置器构建出一个 `ClientDetailsService` 组件注册到 Spring 容器中。
>
> ```java
> @Configuration
> public class ClientDetailsServiceConfiguration {
> 
> 	@SuppressWarnings("rawtypes")
> 	private ClientDetailsServiceConfigurer configurer = new ClientDetailsServiceConfigurer(new ClientDetailsServiceBuilder());
> 
> 	@Bean
> 	public ClientDetailsServiceConfigurer clientDetailsServiceConfigurer() {
> 		return configurer;
> 	}
> 
> 	@Bean
> 	@Lazy
> 	@Scope(proxyMode=ScopedProxyMode.INTERFACES)
> 	public ClientDetailsService clientDetailsService() throws Exception {
> 		return configurer.and().build();
> 	}
> 
> }
> ```
>
> 再查看 `AuthorizationServerSecurityConfiguration` 配置类：发现在当前配置类中，
>
> 1. 会将所有 `AuthorizationServerConfigurer` 类型的组件（包括咱们自己写的授权服务配置类 `JdbcAuthorizationServerConfig`）都注入到 `configurers` 集合属性中，
> 2. 并且在 `configure()` 方法中，会将刚才注册到 Spring 容器中的 `ClientDetailsServiceConfigurer` 配置器组件应用到 `AuthorizationServerConfigurer` 的第二个 `configure()` 方法中。
>
> ```java
> @Configuration
> @Order(0)
> @Import({ ClientDetailsServiceConfiguration.class, AuthorizationServerEndpointsConfiguration.class })
> public class AuthorizationServerSecurityConfiguration extends WebSecurityConfigurerAdapter {
>     @Autowired
>     private List AuthorizationServerConfigurer> configurers = Collections.emptyList();
> 
>     @Autowired
>     public void configure(ClientDetailsServiceConfigurer clientDetails) throws Exception {
>       for (AuthorizationServerConfigurer configurer : configurers) {
>           configurer.configure(clientDetails);
>       }
>     }
> }
> ```
>
> 因此咱们只需要在第二个方法中配置 `ClientDetailsService` 即可，配置好后框架会自动向 Spring 容器中注册一个 `ClientDetailsService` 组件（其实该组件实例与咱们配置在第二个方法中配置的 `ClientDetailsService` 实例对象相等）。

修改后的代码如下所示：

```java
@Configuration
@EnableAuthorizationServer
public class JdbcAuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
    private final AuthenticationManager authenticationManager;
    private final DataSource dataSource;
    private final ClientDetailsService clientDetailsService;

    public JdbcAuthorizationServerConfig(
            final AuthenticationManager authenticationManager,
            final DataSource dataSource,
            final ClientDetailsService clientDetailsService) {
        this.authenticationManager = authenticationManager;
        this.dataSource = dataSource;
        this.clientDetailsService = clientDetailsService;
    }

    @Override
    public void configure(final AuthorizationServerSecurityConfigurer security) throws Exception {
        security.checkTokenAccess("permitAll()")
                .allowFormAuthenticationForClients();
    }

    @Override
    public void configure(final ClientDetailsServiceConfigurer clients) throws Exception {
        clients.withClientDetails(new JdbcClientDetailsService(dataSource));
    }

    @Override
    public void configure(final AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints
                // 密码模式需要使用，否则会报错
                .authenticationManager(authenticationManager)
                // 授权码存储方式
                .authorizationCodeServices(authorizationCodeServices())
                // 令牌存储方式
                .tokenServices(tokenServices());
    }

    @Bean
    public AuthorizationCodeServices authorizationCodeServices() {
        return new JdbcAuthorizationCodeServices(dataSource);
    }

    @Bean
    public TokenStore tokenStore() {
        return new JdbcTokenStore(dataSource);
    }

    @Bean
    public AuthorizationServerTokenServices tokenServices() {
        final DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
        defaultTokenServices.setClientDetailsService(clientDetailsService);
        defaultTokenServices.setTokenStore(tokenStore());
        defaultTokenServices.setSupportRefreshToken(true);
        defaultTokenServices.setAccessTokenValiditySeconds(60 * 60 * 24 * 2);
        defaultTokenServices.setRefreshTokenValiditySeconds(60 * 60 * 24 * 7);
        return defaultTokenServices;
    }
}
```

再次启动项目，使用授权码模式进行测试：

1. 当成功获取到授权码之后，会在 `oauth_code` 表中新增一条记录，如下所示：<br /> ![](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202311291514570.png)
2. 使用该授权码去申请访问令牌，响应结果如下所示：

   ```json
   {
     "access_token": "389f8319-ebe5-45d2-a0b3-06f2a1317db7",
     "token_type": "bearer",
     "refresh_token": "8116a7e2-ceeb-4515-a222-271a111b3046",
     "expires_in": 172799,
     "scope": "scope1"
   }
   ```

   当成功获取到访问令牌之后，会将当前授权码记录从 `oauth_token` 表中删除（表示一个授权码只能使用一次），然后分别在 `oauth_access_token` 和 `oauth_refresh_token` 中新增一条记录，如下所示：<br />![image-20231129152055761](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202311291520809.png)![image-20231129152121713](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202311291521752.png)

3. 使用刷新令牌获取新的访问令牌，响应结果如下所示：

   ```json
   {
     "access_token": "14b9d6da-5c55-4247-9028-3ae359a2d0bd",
     "token_type": "bearer",
     "refresh_token": "8116a7e2-ceeb-4515-a222-271a111b3046",
     "expires_in": 172799,
     "scope": "scope1"
   }
   ```

   当成功获取到新的访问令牌之后，会先从 `oauth_access_token` 表中删除原来的记录，然后再新增一条记录，如下所示：<br />![image-20231129153002038](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202311291530077.png)

其他几种授权模式的情况可以自行测试，此处不再赘叙！

### 资源服务器

创建 `resource-server` 资源服务器模块，添加 `spring-boot-starter-web` 、`spring-cloud-starter-oauth2 `、`mysql-connector-java`、`spring-boot-starter-jdbc` 依赖，其 ` pom.xml ` 配置如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xmlns="http://maven.apache.org/POM/4.0.0"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
    <parent>  
        <groupId>fun.xiaorang</groupId>  
        <artifactId>spring-security-oauth2-study</artifactId>  
        <version>1.0-SNAPSHOT</version>  
    </parent>  
  
    <artifactId>resource-server</artifactId>  
  
    <dependencies>  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-web</artifactId>  
        </dependency>  
        <dependency>  
            <groupId>org.springframework.cloud</groupId>  
            <artifactId>spring-cloud-starter-oauth2</artifactId>  
        </dependency>  
        <dependency>  
            <groupId>mysql</groupId>  
            <artifactId>mysql-connector-java</artifactId>  
        </dependency>  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-jdbc</artifactId>  
        </dependency>  
    </dependencies>  
</project>
```

`application.yml` 配置文件如下所示：

```yaml
server:  
  port: 8081  
  
spring:  
  datasource:  
    username: root  
    password: 123456  
    url: jdbc:mysql://localhost:3306/oauth?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true  
    driver-class-name: com.mysql.cj.jdbc.Driver
```

创建资源：其中 `hello` 请求只需用户认证之后就能访问，而 `admin` 请求不仅需要用户认证还需要具有 `ADMIN` 角色。

```java
@RestController  
public class HelloController {  
    @GetMapping("/hello")  
    public String hello() {  
        return "hello";  
    }  
  
    @GetMapping("/admin")  
    @PreAuthorize("hasRole('ROLE_ADMIN')")  
    public String admin() {  
        return "admin";  
    }  
}
```

为了进行测试，修改原有的客户端记录，使当前客户端能同时访问 `res1` 和 `res2` 资源服务器上的资源，并且同时拥有 `scope1` 和 `scope2` 的权限（默认情况下，OAuth 2.0 的权限是从 Client 的 `scope` 中获取，）。如下所示：<br />![](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202311301227581.png)

编写资源服务器配置类：由于咱们基于数据库存储访问令牌信息，因此只需配置 `JdbcTokenStore` 组件让其知道去数据库中取即可。

```java
@Configuration
@EnableResourceServer
@EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true)
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
    private final DataSource dataSource;

    public ResourceServerConfig(final DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Override
    public void configure(final ResourceServerSecurityConfigurer resources) throws Exception {
        resources
                // 配置当前资源服务器的资源 id，默认为 oauth2-resource，
                // 只有客户端所拥有的资源包含当前资源服务器的资源 id 时，才能访问当前资源服务器
                .resourceId("res1")
                .tokenStore(tokenStore())
                .stateless(true);
    }

    @Override
    public void configure(final HttpSecurity http) throws Exception {
        http.authorizeRequests()
                // 配置当前资源服务器必须满足 scope1 的权限才能访问
                .antMatchers("/**").access("#oauth2.hasScope('scope1')")
                // 配置当前资源服务器任何请求都必须认证后才能访问
                .anyRequest()
                .authenticated();
    }

    @Bean
    public JdbcTokenStore tokenStore() {
        return new JdbcTokenStore(dataSource);
    }
}
```

启动资源服务器，然后针对 user 用户和 admin 用户分别使用密码模式获取访问令牌（确保授权服务器已经启动），最后各自拿着获取到的访问令牌去请求 `/hello` 和 `/admin` 资源，观察不登陆的时候是否能访问到 `/hello` 资源以及当使用 `user` 用户登录时，是否具备权限去访问 `/admin` 资源？ 测试的具体流程如下所示：

1. 首先直接去访问 `/hello` 和 `/admin` 请求，发现被拦截，抛出 `Full authentication is required to access this resource` 错误！证明咱们的配置生效，认证不成功的话是无法访问任何资源的！
2. 然后使用 `user` 用户通过密码模式获取访问令牌，发送 POST 请求： http://localhost:8080/oauth/token?client_id=client1&client_secret=123456&grant_type=password&username=user&password=123456 ，获取到访问令牌之后，发送 GET 请求： http://localhost:8081/hello ，并且在请求头上增加 `Authorization: bearer token`（其中的 token 需要替换成真实获取到的访问令牌），发现可以成功访问到资源并返回 `hello`；当以同样的方式去访问 http://localhost:8081/admin 资源时，发现抛出 `access_denied` 权限不足，不允许访问的错误。<br /> ![](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202311301352312.png)
3. 最后使用 `admin` 用户通过密码模式获取访问令牌，发送 POST 请求： http://localhost:8080/oauth/token?client_id=client1&client_secret=123456&grant_type=password&username=admin&password=123456 ，获取到访问令牌之后，不管是去访问 `/hello` 资源还是 `/admin` 资源都是没有任何问题的。

从上面的测试情况可以证明咱们的配置完全没有任何问题，达到预期效果：任何资源都需要认证才能访问，并且对于 `/admin` 资源还必须具备 `ADMIN` 角色才能够访问。

### 使用 JWT

#### 授权服务器颁发 JWT 令牌

其他内容不变，注释掉原来的授权服务配置类 `JdbcAuthorizationServerConfig`，在其基础上编写一个 `JwtAuthorizationServerConfig` 授权服务配置类，使当前授权服务器颁发 JWT 令牌，而不再是原来 UUID 类型的令牌，代码实现如下所示：

```java
@Configuration
@EnableAuthorizationServer
public class JwtAuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
    private final AuthenticationManager authenticationManager;
    private final DataSource dataSource;
    private final ClientDetailsService clientDetailsService;

    public JwtAuthorizationServerConfig(
            final AuthenticationManager authenticationManager,
            final DataSource dataSource,
            final ClientDetailsService clientDetailsService) {
        this.authenticationManager = authenticationManager;
        this.dataSource = dataSource;
        this.clientDetailsService = clientDetailsService;
    }

    @Override
    public void configure(final AuthorizationServerSecurityConfigurer security) throws Exception {
        security.checkTokenAccess("permitAll()")
                .allowFormAuthenticationForClients();
    }

    @Override
    public void configure(final ClientDetailsServiceConfigurer clients) throws Exception {
        clients.withClientDetails(new JdbcClientDetailsService(dataSource));
    }

    @Override
    public void configure(final AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints
                // 密码模式需要使用，否则会报错
                .authenticationManager(authenticationManager)
                // 授权码存储方式
                .authorizationCodeServices(authorizationCodeServices())
                // 令牌存储方式
                .tokenServices(tokenServices())
        ;
    }

    @Bean
    public AuthorizationCodeServices authorizationCodeServices() {
        return new JdbcAuthorizationCodeServices(dataSource);
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        // 配置 JWT 令牌增强器，用于生成与解析 JWT 令牌
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        // 密钥，用于对 JWT 进行签名，在资源服务器中需要配置相同的密钥，才能解密 JWT
        converter.setSigningKey("xiaorang");
        return converter;
    }

    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    public AuthorizationServerTokenServices tokenServices() {
        final DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
        defaultTokenServices.setClientDetailsService(clientDetailsService);
        defaultTokenServices.setTokenStore(tokenStore());
        defaultTokenServices.setSupportRefreshToken(true);
        defaultTokenServices.setAccessTokenValiditySeconds(60 * 60 * 24 * 2);
        defaultTokenServices.setRefreshTokenValiditySeconds(60 * 60 * 24 * 7);
        final TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
        tokenEnhancerChain.setTokenEnhancers(Collections.singletonList(jwtAccessTokenConverter()));
        defaultTokenServices.setTokenEnhancer(tokenEnhancerChain);
        return defaultTokenServices;
    }
}
```

重新启动授权服务器，使用密码模式去申请访问令牌，响应结果如下所示：

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOlsicmVzMSIsInJlczIiXSwidXNlcl9uYW1lIjoiYWRtaW4iLCJzY29wZSI6WyJzY29wZTEiLCJzY29wZTIiXSwiZXhwIjoxNzAxNDk0NzA1LCJhdXRob3JpdGllcyI6WyJST0xFX0FETUlOIl0sImp0aSI6ImNlODA4ODI4LThlODEtNDJjMS1iYWJmLTg5ZjVkNDc3OWM4ZSIsImNsaWVudF9pZCI6ImNsaWVudDEifQ.bAnlYz75_5CuusAI0IJ_d7ivzVfHu_3WzlitCekaY7U",
  "token_type": "bearer",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOlsicmVzMSIsInJlczIiXSwidXNlcl9uYW1lIjoiYWRtaW4iLCJzY29wZSI6WyJzY29wZTEiLCJzY29wZTIiXSwiYXRpIjoiY2U4MDg4MjgtOGU4MS00MmMxLWJhYmYtODlmNWQ0Nzc5YzhlIiwiZXhwIjoxNzAxOTI2NzA1LCJhdXRob3JpdGllcyI6WyJST0xFX0FETUlOIl0sImp0aSI6IjlhOWNlMTllLThmZGItNDczNC04MzU1LTY4NGE3ODRjOGYwYyIsImNsaWVudF9pZCI6ImNsaWVudDEifQ.0vJNGcr972GUn4Yqbf90vU_LjC1nX5GdaBER77Io0J8",
  "expires_in": 172799,
  "scope": "scope1 scope2",
  "jti": "ce808828-8e81-42c1-babf-89f5d4779c8e"
}
```

将当前 access_token 拷贝到 [JSON Web Tokens - jwt.io](https://jwt.io/) 去查看 JWT 负荷部分解析后的数据，其中的 jti 为原始的 access token，如下所示： <br /> ![image-20231130133028139](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202311301330219.png)

#### 使用 JWT 令牌访问资源服务器

在原来的资源服务配置类 `ResourceServerConfig` 基础上，将 `JdbcTokenStore` 修改为 `JwtTokenStore`，并且配置 `JwtAccessTokenConverter` 用于将用户信息从 JWT 中解析出来。代码如下所示：

```java
@Configuration
@EnableResourceServer
@EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true)
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
    @Override
    public void configure(final ResourceServerSecurityConfigurer resources) throws Exception {
        resources
                // 配置当前资源服务器的资源 id，默认为 oauth2-resource，
                // 只有客户端所拥有的资源包含当前资源服务器的资源 id 时，才能访问当前资源服务器
                .resourceId("res1")
                .tokenStore(tokenStore())
                .stateless(true);
    }

    @Override
    public void configure(final HttpSecurity http) throws Exception {
        http.
                csrf()
                .disable()
                .authorizeRequests()
                // 配置当前资源服务器必须满足 scope1 的权限才能访问
                .antMatchers("/**").access("#oauth2.hasScope('scope1')")
                // 配置当前资源服务器任何请求都必须认证后才能访问
                .anyRequest()
                .authenticated()
                .and()
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS);
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        // 配置 JWT 令牌增强器，用于生成与解析 JWT 令牌
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        // 密钥，用于对 JWT 进行解密，必须与授权服务器的密钥一致
        converter.setSigningKey("xiaorang");
        return converter;
    }
    
    @Bean
    public JwtTokenStore tokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }
}
```

重新启动资源服务器，拿着授权服务器颁发的 JWT 令牌（同样是在请求头上增加 `Authorization: bearer token`，其中的 token 替换为 JWT 令牌即可）访问 `/hello` 资源，发现可以正常访问，返回 "/hello" 字符串。

#### 扩展

##### 为 JWT 添加附加信息

JWT 默认生成的用户信息主要是用户角色、用户名等，如果咱们希望在生成的 JWT 上面添加额外的信息，可以按照如下方式添加：

```java
@Bean
public TokenEnhancer customAdditionalInformation() {
    return (accessToken, authentication) -> {
        final Map<String, Object> additionalInformation = accessToken.getAdditionalInformation();
        Map<String, String> info = new LinkedHashMap<>();
        info.put("author", "xiaorang");
        info.put("blog", "https://blog.xiaorang.fun");
        info.put("github", "https://github.com/xihuanxiaorang");
        additionalInformation.put("info", info);
        ((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(additionalInformation);
        return accessToken;
    };
}
```

```java
@Bean
public AuthorizationServerTokenServices tokenServices() {
    final DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
    defaultTokenServices.setClientDetailsService(clientDetailsService);
    defaultTokenServices.setTokenStore(tokenStore());
    defaultTokenServices.setSupportRefreshToken(true);
    defaultTokenServices.setAccessTokenValiditySeconds(60 * 60 * 24 * 2);
    defaultTokenServices.setRefreshTokenValiditySeconds(60 * 60 * 24 * 7);
    final TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
    // TOKEN 增强链的顺序需要保证 JwtAccessTokenConverter 位于附加信息增强器之前
    tokenEnhancerChain.setTokenEnhancers(Arrays.asList(jwtAccessTokenConverter(), customAdditionalInformation()));
    defaultTokenServices.setTokenEnhancer(tokenEnhancerChain);
    return defaultTokenServices;
}
```

Token 增强器处理的顺序是按照集合中保存的顺序，就是先在 `JwtAccessTokenConverter` 中处理，然后在 `CustomAdditionalInformation ` 中处理，顺序不能乱！

重新启动授权服务器，使用密码模式申请访问令牌，响应如下所示：

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOlsicmVzMSIsInJlczIiXSwidXNlcl9uYW1lIjoiYWRtaW4iLCJzY29wZSI6WyJzY29wZTEiLCJzY29wZTIiXSwiZXhwIjoxNzAxNTA2OTY1LCJhdXRob3JpdGllcyI6WyJST0xFX0FETUlOIl0sImp0aSI6IjdmZTdiZWY3LWJiNjYtNDk4Mi1iZDJiLTZmNTczYThkZTM0ZCIsImNsaWVudF9pZCI6ImNsaWVudDEifQ.a8aLiQitPJ8lITPS9z5ss3q5xmVgjaBpA6MPwW1Uf1A",
  "token_type": "bearer",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOlsicmVzMSIsInJlczIiXSwidXNlcl9uYW1lIjoiYWRtaW4iLCJzY29wZSI6WyJzY29wZTEiLCJzY29wZTIiXSwiYXRpIjoiN2ZlN2JlZjctYmI2Ni00OTgyLWJkMmItNmY1NzNhOGRlMzRkIiwiZXhwIjoxNzAxOTM4OTY1LCJhdXRob3JpdGllcyI6WyJST0xFX0FETUlOIl0sImp0aSI6IjgxZGNkYjk3LTdhNWQtNDYwZi1hNjU3LWU0ZDg1N2UwZTE4YSIsImNsaWVudF9pZCI6ImNsaWVudDEifQ.m5APuu7eXy-oJE76s50_3ykMWZi266o38N2D-HjilJY",
  "expires_in": 172799,
  "scope": "scope1 scope2",
  "jti": "7fe7bef7-bb66-4982-bd2b-6f573a8de34d",
  "info": {
    "author": "xiaorang",
    "blog": "https://blog.xiaorang.fun",
    "github": "https://github.com/xihuanxiaorang"
  }
}
```

##### 非对称加解密

在前面的例子中，JWT 令牌转换器使用的是最简单的**对称加密（授权服务器与资源服务器使用相同的密钥，加密和解密过程使用的是同一份密钥）**的方式来加密 JWT 内容：

```java
@Bean
public JwtAccessTokenConverter jwtAccessTokenConverter() {
    // 配置 JWT 令牌增强器，用于生成 JWT 令牌
    JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
    // 密钥，用于对 JWT 进行签名，在资源服务器中需要配置相同的密钥，才能解密 JWT
    converter.setSigningKey("xiaorang");
    return converter;
}
```

在生产环境中，使用的是更加安全的**非对称加密**的方式来加密 JWT。非对称加密使用的是一对秘钥（**非对称密钥对**）：一个称为**私钥**，另一个称为**公钥**。授权服务器使用它的私钥签署令牌，而资源服务器则使用公钥验证签名。

> [!info]- OpenSSL 的下载安装（在后续生成非对称密钥对的过程中需要使用）
>
> 没有 OpenSSL 的小伙伴可以到 [官网](https://www.openssl.org/source/) 下载或者直接使用其他人做的 [便捷安装包](https://slproweb.com/products/Win32OpenSSL.html)（Windows 用户推荐使用这种方式，安装完成之后配置一下环境变量即可），如下所示：![image-20231130230829715](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202311302308775.png)

>[!faq]+ 那么该如何生成一个非对称密钥对呢？
>
>可以使用 JDK 的命令行工具 keytool 和 OpenSSL 来生成，首先来看一下这个命令下有哪些参数：![image-20231130215729201](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202311302157500.png)
>
>参数说明很清晰，咱们需要使用 `-genkeypair` 参数生成密钥对。再来看一下 `keytool -genkeypair` 之下还有哪些参数：![image-20231130220049712](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202311302200997.png)
>
>执行 `keytool -genkeypair -alias oauth2 -keyalg RSA -keypass 123456 -keystore oauth2.jks -storepass 123456` 命令生成 JKS（Java KeyStore）文件，从参数说明可知，设置的别名为 `oauth2 `，签名算法为 `RSA`，密钥口令为 `123456`，密钥库（文件）名称为 `oauth2.jks`，密钥库口令为 `123456`。![image-20231130221348767](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202311302213023.png)
>
>将生成好的密钥库文件 `oauth2.jks` 拷贝到**授权服务器**的 `resources` 资源目录下即可。
>
>执行 `  keytool -list -rfc --keystore oauth2.jks | openssl x509 -inform pem -pubkey` 命令生成公钥，如下所示：![image-20231130230447125](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202311302304453.png)
>
>将公钥保存到 `public.txt` 文件中，然后拷贝到**资源服务器**的 `resources` 目录下即可。

修改授权服务配置类中的 `JwtAccessTokenConverter`，如下所示：

```java
@Bean
public JwtAccessTokenConverter jwtAccessTokenConverter() {
    // 配置 JWT 令牌增强器，用于生成 JWT 令牌
    JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
    final KeyStoreKeyFactory keyStoreKeyFactory = new KeyStoreKeyFactory(new ClassPathResource("oauth2.jks"), "123456".toCharArray());
    converter.setKeyPair(keyStoreKeyFactory.getKeyPair("oauth2"));
    return converter;
}
```

重新启动授权服务器，再次使用密码模式申请访问令牌，响应结果同以前一样，只不过此次生成的访问令牌必须使用与私钥相对应的公钥才能解密，否则的话，拿着该令牌去资源服务器中访问资源时会抛出如下错误：

```json
{
  "error": "invalid_token",
  "error_description": "Cannot convert access token to JSON"
}
```

因此，咱们现在需要去资源服务器中配置对应的公钥才能正确解密。修改资源服务配置类中的 `JwtAccessTokenConverter`，如下所示：

```java
@Bean  
public JwtAccessTokenConverter jwtAccessTokenConverter() {  
    JwtAccessTokenConverter converter = new JwtAccessTokenConverter();  
    // 通过读取本地文件获取非对称加密公钥，用于对 JWT 进行解密  
    converter.setVerifierKey(getPublicKey());  
    return converter;  
}  
  
private String getPublicKey() {  
    final Resource resource = new ClassPathResource("public.txt");  
    String publicKey = null;  
    try {  
        try (BufferedReader br = new BufferedReader(new InputStreamReader(resource.getInputStream()))) {  
            publicKey = br.lines().collect(Collectors.joining("\n"));  
        }  
    } catch (IOException e) {  
        e.printStackTrace();  
    }  
    return publicKey;  
}
```

重新启动资源服务器，咱们再次拿着令牌去访问资源，发现没有任何问题，可以正常访问！

> [!tip]
>
> 可以通过发送 GET 请求：[localhost:8080/oauth/token_key](http://localhost:8080/oauth/token_key) 获取公钥，不过以现在的配置会抛出 `Unauthorized` 错误，需要修改授权服务配置类中的第一个 `configure()` 方法，开放获取公钥端点，如下所示：
>
> ```java
> @Override
> public void configure(final AuthorizationServerSecurityConfigurer security) throws Exception {
>     security
>             // 开放检查 token 端点
>             .checkTokenAccess("permitAll()")
>             // 开放获取公钥端点
>             .tokenKeyAccess("permitAll()")
>             .allowFormAuthenticationForClients();
> }
> ```
>
> 再次访问 [localhost:8080/oauth/token_key](http://localhost:8080/oauth/token_key) 获取公钥，响应结果如下所示：
>
> ```json
> {
>   "alg": "SHA256withRSA",
>   "value": "-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAr61mTQ3B3eFejJ1YY/L7CvAWbTTXkeUqp1eEEYg/NWzY92Ryp27jmTCnlvkbUfVUeEb7JxTxPPQ9RmF4JocV0QGPihyAiLgejINUkkBPLo5KBbyRt2c5OfLZQ3/HAXQCCbOMd7mPLaySdbz5isDmT2T3Aw06E9/DzywahB3IJPlANu7q3fxOTwp599Vl2f+bIfVTOUenISV2cbuwqbKZGUvEdvluosmLM6GX0IjzFkq4Qfshi2LcbYslVRy7nskvm4NozN+e+qonAVw+PLzgQ4/+uK+Io4o+SKB/KbMGK0UylhiHAG2txbPDhayyAWhfjr5leNjnfEmMvE+C9ZVXzwIDAQAB\n-----END PUBLIC KEY-----"
> }
> ```
>
> 其实这种方式与咱们通过命令行方式获取到的公钥一模一样！

## 参考资料🎁

- 文档
	- Oauth2.0 协议 [RFC 6749 - The OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/html/rfc6749)
	- [OAuth 2.0 开放网络授权协议权威指南 - Coding10](https://www.coding10.com/book/book-oauth2-explain)
	- [OAuth2系列 | 江南一点雨](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI1NDY0MTkzNQ==&action=getalbum&album_id=1319833457266163712&subscene=159&subscene=189&scenenote=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzI1NDY0MTkzNQ%3D%3D%26mid%3D2247488214%26idx%3D1%26sn%3D5601775213285217913c92768d415eca%26chksm%3De9c340b6deb4c9a01bc383b2c0ab124358663adf22a58ba385f792224ba532079a028ba92a3d%26cur_album_id%3D1319833457266163712%26scene%3D189%23wechat_redirect&nolastread=1#wechat_redirect)
	- [Spring Security + OAuth 2.0 + JWT 开发随笔 | Clay 的技术空间](https://www.techgrow.cn/posts/894ad1eb.html)
	- [Re：从零开始的 Spring Security OAuth2（一） - 徐靖峰|个人博客](https://www.cnkirito.moe/Spring-Security-OAuth2-1/)
	- [Spring Cloud实战 | 第六篇：Spring Cloud Gateway + Spring Security OAuth2 + JWT实现微服务统一认证授权鉴权 - 有来技术 - 博客园](https://www.cnblogs.com/haoxianrui/p/13719356.html)
- 视频
	- 【【编程不良人】SpringSecurity 最新实战教程，知识点完结！】 https://www.bilibili.com/video/BV1z44y1j7WZ/?share_source=copy_web&vd_source=84272a2d7f72158b38778819be5bc6ad
	- 【最简单的使用 SpringOAuth2 进行分布式权限管理】 https://www.bilibili.com/video/BV1p5411N7Po/?share_source=copy_web&vd_source=84272a2d7f72158b38778819be5bc6ad
	- 【【读源码学架构】spring-security-oauth2.0 】 https://www.bilibili.com/video/BV1pz4y1E72k/?share_source=copy_web&vd_source=84272a2d7f72158b38778819be5bc6ad
