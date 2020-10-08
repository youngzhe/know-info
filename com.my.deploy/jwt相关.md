#jwt定义
    JWT作为一个开放的标准（RFC 7519），定义了一种简洁的，自包含的方法用于通信双方之间以Json对象的形式安全的传递信息。
        简洁(Compact): 可以通过URL，POST参数或者在HTTP header发送，因为数据量小，传输速度也很快
        自包含(Self-contained)：负载中包含了所有用户所需要的信息，避免了多次查询数据库
        
##JWT的主要应用场景
    身份认证
        在这种场景下，一旦用户完成了登陆，在接下来的每个请求中包含JWT，可以用来验证用户身份以及对路由，服务和资源的访问权限进行验证。由于它的开销非常小，可以轻松的在不同域名的系统中传递，所有目前在单点登录（SSO）中比较广泛的使用了该技术。
    信息交换
        在通信的双方之间使用JWT对数据进行编码是一种非常安全的方式，由于它的信息是经过签名的，可以确保发送者发送的信息是没有经过伪造的。
        
##JWT的结构
    JWT包含了使用.分隔的三部分：
        Header 头部
        Payload 负载
        Signature 签名
###Header
       在header中通常包含了两部分：token类型和采用的加密算法。
       
       {
         "alg": "HS256",
         "typ": "JWT"
       }  
    对这部分内容使用 Base64Url 编码组成了JWT结构的第一部分
###Payload
    Token的第二部分是负载，它包含了claim， Claim是一些实体（通常指的用户）的状态和额外的元数据，有三种类型的claim： reserved, public 和 private.
        Reserved claims: 这些claim是JWT预先定义的，在JWT中并不会强制使用它们，而是推荐使用，常用的有 iss（签发者）, exp（过期时间戳）, sub（面向的用户）, aud（接收方）, iat（签发时间）。
        Public claims：根据需要定义自己的字段，注意应该避免冲突
        Private claims：这些是自定义的字段，可以用来在双方之间交换信息
    负载使用的例子：
    {
      "sub": "1234567890",
      "name": "John Doe",
      "admin": true
    }
    上述的负载需要经过Base64Url编码后作为JWT结构的第二部分。  
###Signature
    创建签名需要使用编码后的header和payload以及一个秘钥，使用header中指定签名算法进行签名。例如如果希望使用HMAC SHA256算法，那么签名应该使用下列方式创建：
    HMACSHA256(
      base64UrlEncode(header) + "." +
      base64UrlEncode(payload),
      secret)  
    签名用于验证消息的发送者以及消息是没有经过篡改的。    

##为什么要使用JWT
    相比XML格式，JSON更加简洁，编码之后更小，这使得JWT比SAML更加简洁，更加适合在HTML和HTTP环境中传递。
    在安全性方面，SWT只能够使用HMAC算法和共享的对称秘钥进行签名，而JWT和SAML token则可以使用X.509认证的公私秘钥对进行签名。与简单的JSON相比，XML和XML数字签名会引入复杂的安全漏洞。
    因为JSON可以直接映射为对象，在大多数编程语言中都提供了JSON解析器，而XML则没有这么自然的文档-对象映射关系，这就使得使用JWT比SAML更方便。