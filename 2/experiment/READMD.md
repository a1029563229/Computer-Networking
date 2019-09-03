# 抓包实践

## HTTP 请求

### Request headers
- GET / HTTP/1.1
  - 使用 GET 方法
  - 获取 / 路径上的文件，这里应该是有个 HTTP 查找规则的，默认是寻找路径下的 index.html
  - 使用 HTTP 协议，版本是 1.1
- Host
  - 主机名（可能是域名也可能是 ip）
  - RFC 协议规定所有的 HTTP 请求必须携带 Host 头，即使 Host 没有值，也必须带上这个 Host 头附加一个空串，如果不满足，应用服务器应该抛出 400 Bad Request。协议虽然这样规定，不过大部分网关或者服务器都比较仁慈，既然没有指定 Host 字段，那就给你默认加上一个。网关代理可以根据不同的 Host 值转发到不同的 upstream 服务节点，它常用于虚拟主机服务业务。
- Pragma	no-cache
  - 禁止缓存；
  - 在前端开发模式下经常会加上这个头部，当网关收到一个带有这样请求的头部时，即使内部存在该请求资源的缓存并且有效也不可以直接发送给客户端，而必须转发给后面的 upstream 进行处理。不过如果真的所有的网关都遵循这个协议的话，攻击是很容易伪造的，所以它一般仅用于开发模式，防止静态资源修改后前端得不到即时更新。
- Cache-Control	no-cache
  - 通知服务器，不使用缓存资源；
  - 这个头既可以用于请求，也可以用于响应，在请求和响应的取值不一样，分别代表了不同的意思。
  - no-cache：如果 no-cache 没有指定值，那就表示不允许缓存。对于请求来说，服务器不得使用缓存内容直接返回。对于响应来说，客户端不得缓存响应的资源内容。如果 no-cache 指定了值，那就表示对应的头信息不得使用缓存，其他的信息还是可以缓存的。
  - no-store：告知对方不要持久化请求/响应数据到其他地方，这种信息是敏感的，要保持它的易失性。告知对方存在内存（memory）可以，但是不能写在磁盘（disk）中。
  - no-transform：告知对方不要转换数据。比如客户端上传了 raw 图像数据，服务器一般都会选择性压缩图像数据进行存储。no-transform 告知对方保留原始数据信息，不要进行任何转换。
  - only-if-cached：用于请求头，告知服务器只要那些已经缓存的内容，如果没有缓存内容就返回 504 Gateway Timeout 错误。（只要缓存，不要新数据）
  - max-age：用于请求头。限制缓存内容的年龄，如果超过 max-age 年龄的，需要服务器去 reload 内容资源。
  - max-state：用于请求头。客户端允许服务器返回缓存已过期的资源内容，但是限定了最大过期时间。
  - min-fresh：用于请求头。客户端限制服务器不要那些即将过期的资源内容。
  - public：用于响应头。表示允许客户端缓存响应信息，并可以给别人使用。比如代理服务器缓存静态资源供素有代理用户使用。
  - private：用于响应头。表示仅允许客户端缓存响应信息给自己使用，不得分享给别人。这样是为了禁止代理服务器进行缓存，而允许客户端自己缓存资源内容。
- WWW-Authenticate
  - WWW-Authenticate 是 401 Unauthorized 错误码返回时必须携带的头，该头会携带一个问题 Challenge 给客户端，告知客户端需要携带这个问题的答案来请求服务器才可以继续访问目标资源。这种问题 Challenge 可以自定义，比较常见的是 Basic 认证。
  - WWW-Authenticate: Basic realm=xxx
  - Basic 指代 base64 加密算法(不安全)，realm 指代认证范围 /场合 /情景名称。
- Authorization: Basic YWRtaW46YWRtaW4xMjM=
  - value = base64(user_name:password)
  - 对于某些需要特殊权限才能访问的资源需要客户端在请求里提供用户名密码的认证信息。它是对 WWW-Authenticate 的应答。
- Upgrade-Insecure-Requests：1
  - Upgrade-Insecure-Requests 是一个请求首部，用来向服务器端发送信号，表示客户端优先选择加密及带有身份验证的响应，并且它可以成功处理 upgrade-insecure-requests CSP 指令。