# HTTP 报文

## 请求报文

- 请求行 `GET / HTTP/1.1`
  - 请求方法 `GET`
  - 请求目标 URI `/`
  - HTTP 协议版本号 `HTTP/1.1`
- header
- entity

## 响应报文

- 状态行 `HTTP/1.1 200 OK`
  - HTTP 协议版本号 `HTTP/1.1`
  - 状态码 `200`
  - 原因 `OK`
- header
- entity

## header

特性：
- key: value 格式；
- 不区分大小写；
- 不允许空格；
- 可以使用 -，不可以使用 _；

类型：
- 通用字段
  - Date：表示 HTTP 报文创建的时间，通常出现在响应头；
- 请求字段
  - Host：告诉服务器这个请求应该由哪个主机来处理；
  - User-Agent：使用一个字符串来描述发起 HTTP 请求的客户端，服务器可以依据它来返回最合适此浏览器显示的页面；
  - Accept：客户端可理解的 MIME type；
  - Accept-Encoding：客户端支持的压缩格式；
  - Accept-Language：客户端支持的自然语言；
  - Range: bytes=0-31：请求部分资源；
  - Connection: keep-alive：保持连接，HTTP/1.1 默认启用长连接；
  - Cookie：服务器返回的客户端标识；
- 响应字段
  - Server：告诉客户端当前正在提供 Web 服务的软件名称和版本号；
  - Content-Type：实体数据的类型；
  - Content-Encoding：数据的压缩格式；
  - Content-Language：实际的语言类型；
  - Transfer-Encoding: chunked：分块传输编码策略；
  - Connection: keep-alive：启用了长连接；
  - Connection: close：链接即将关闭；
  - Set-Cookie：服务器返回的客户端标识；
  - Cache-Control：资源缓存的控制策略；
  - 
- 实体字段
  - Content-Length：它表示报文里 body 的长度；

## HTTP 请求方法

- GET：获取资源，可以理解为读取或者下载数据；
- HEAD：获取资源的元信息；
- POST：向资源提交数据，相当于写入或上传数据；
- PUT：类似 POST；
- DELETE：删除资源；
- CONNECT：建立特殊的连接隧道；
- OPTIONS：列出可对资源实行的方法；
- TRACE：追踪请求 - 响应的传输路径。