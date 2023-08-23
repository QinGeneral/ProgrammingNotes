# JWT 认证机制

## 传统认证流程 Cookie + Session

互联网服务提供认证机制的传统流程如下：

1. 用户向服务器发送用户名和密码；
2. 服务器验证通过后，在当前对话（Session）里面保存相关数据，比如用户角色、登录时间等等；
3. 服务器向用户返回一个 session_id，写入用户的 Cookie；
4. 用户随后的每一次请求，都会通过 Cookie，将 session_id 传回服务器；
5. 服务器收到 session_id，找到前期保存的数据，由此得知用户的身份。

但是这种认证方式有以下缺点：

- Session：每个用户经过认证之后，都要在服务端做一次记录，以方便用户下次请求的鉴别，通常而言 Session 都是保存在内存中，而随着认证用户的增多，服务端的开销会明显增大；
- 扩展性：用户认证之后，服务端做认证记录，如果认证的记录被保存在内存中的话，这意味着用户下次请求还必须要请求在这台服务器上,这样才能拿到授权的资源，这样在分布式的应用上，相应的限制了负载均衡器的能力。这也意味着限制了应用的扩展能力；
- CSRF：因为是基于 Cookie 来进行用户识别的, Cookie 如果被截获，用户就会很容易受到跨站请求伪造的攻击。

如果要克服上述缺点，其中一种解决方案是 Session 数据持久化，写入数据库或别的持久层。各种服务收到请求后，都向持久层请求数据。这种方案的优点是架构清晰，缺点是工程量比较大。另外，持久层万一挂了，就会单点失败。

另一种方案是服务器索性不保存 Session 数据了，所有数据都保存在客户端，每次请求都发回服务器。JWT 就是这种方案的一个代表。

## JWT（Json Web Token） 认证流程

![](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1692787491.8047009SmartPic.png)

JWT 的认证流程如下：

1. 客户端向自定义授权服务发起认证请求，请求中一般会携带终端用户的用户名和密码；
2. 自定义授权服务读取请求中的验证信息（例如用户名、密码等）进行验证，验证通过之后使用私钥生成标准的 token；
3. 自定义授权服务将携带 token 的应答返回给客户端，客户端需要将这个 token 缓存到本地；
4. 客户端向服务器发送业务请求，请求的 header 的 Authorization 中携带 token，格式为 `Authorization: Bearer <token>`。服务器使用用户设置的公钥对请求中的 token 进行验证。

## JWT 格式

一个 JWT 数据的格式如下：

```shell
header.payload.signature

# 示例
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.
eyJ0b2tlbkV4cCI6MTY5MjAwNDg1NSwibW4iOjAsInJvbGUiOjAsImV4cCI6MTY5MjAwNDg1NSwiaWF0IjoxNjkyMDAzMDU1fQ.
5k0rJgpg18PhllCepYWxYBJNiX2wrYfmaLO5ZJy3HLc
```

### Header

Header 部分是一个 JSON 对象，描述 JWT 的元数据，通常是下面的样子。

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

上面代码中，alg 属性表示签名的算法（algorithm），默认是 HMAC SHA256（写成 HS256）；typ 属性表示这个令牌（token）的类型（type），JWT 令牌统一写为 JWT。

最后，将上面的 JSON 对象使用 Base64URL 算法（详见后文）转成字符串。

### Payload

Payload 部分也是一个 JSON 对象，用来存放实际需要传递的数据。JWT 规定了 7 个官方字段，供选用。

- iss (issuer)：签发人
- exp (expiration time)：过期时间
- sub (subject)：主题
- aud (audience)：受众
- nbf (Not Before)：生效时间
- iat (Issued At)：签发时间
- jti (JWT ID)：编号

除了官方字段，你还可以在这个部分定义私有字段，如下：

```
{
  "name": "John Snow",
}
```

JWT 默认是不加密的，任何人都可以读到，所以不要把密码这些秘密信息放在这个部分。这个 JSON 对象也会使用 Base64URL 算法转成字符串。

### Signature

Signature 部分是对前两部分的签名，防止数据篡改，作为服务器的信息校验部分。

首先，需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名。

```java
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用"点"（.）分隔，返回给用户。

## 注意事项

1. JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次；
2. JWT 不加密的情况下，不能将秘密数据写入 JWT；
3. JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数；
4. JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑；
5. JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证；
6. 为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输。

> 你可以在 [jwt.io](https://jwt.io/#debugger-io) mkjwk.org 网站上模拟各种算法生成的 JWT。
