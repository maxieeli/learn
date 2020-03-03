# HTTP总结

### HTTP头部

#### 1. HTTP头部

首部字段名  | 描述
------------  | -------------
Accept  |  告诉服务器，客户端支持的数据类型
Accept-Charset  |  告诉服务器，客户端采用的编码
Accept-Encoding  |  告诉服务器，客户机支持的数据压缩格式
Accept-Language  |  告诉服务器，客户机的语言环境
Host  |  客户机通过这个信息告诉服务器，相访问的主机名
If-Mondified-Since  |   客户机通过这个信息告诉服务器，资源缓存的时间
Referer  |   客户机通过这个信息告诉服务器，它是从哪个资源来访问服务器
User-Agent  |   客户机通过这个信息告诉服务器，客户机的软件环境
Cookie  |   客户机通过这个信息告诉服务器，可以向服务器带数据
Connection  |   客户机通过这个信息告诉服务器，请求完后是关闭还是保持连接
Date  |   客户机通过这个信息告诉服务器，客户机当前请求的时间



#### 2. Request Header

Request Header  |  描述
------  | ------
`GET/simgle.html HTTP/1.1`  |  请求行
`Host: www.uuid.online/`  |  请求的目标域名和端口号
`Origin: http://localhost:9001/`  |  请求的来源域名和端口号(跨域请求时，浏览器会自动带上这个头信息)
`Referer: https://localhost:9001/link?query=xxx`  |  请求资源的完整url
`User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36`  |  浏览器信息
`Cookie: BAIDUID=FA89F036:FG=1; BD_HOME=1; sugstore=0`  |  当前域名下的Cookie
`Accept: text/html;image/png`  |  代表客户端希望接受的数据类型是html或者png
`Accept-Encoding: gzip, deflate`  |  代表客户端支持gzip和deflate格式的压缩
`Accept-Language`  |  代表客户端可以支持语言zh-CN或者zh
`Connection: keep-alive`  |  告诉服务器，客户端需要的tcp连接是一个长连接



#### 3. Response Header

Response Header  |  描述
------ | ------
`HTTP/1.1 200 OK`  |  响应状态行
`Date: Mon, 1 Mar 2020 02:50:55 GMT`  |  服务端发送资源时的服务器时间
`Expires: Wed, 31 Dec 1969 23:59:59 GMT`  |  比较过时的一种验证缓存的方式，与浏览器的时间比较，超过这个时间就不用缓存(不和服务器进行验证)
`Cache-Control: no-cache`  |  控制缓存的方式，会和服务器进行缓存校验
`etag: "fb8ba2f80b1d324bb997cbe188f28187-ssl-df"`  |  一般是Nginx静态服务器发来的静态文件签名，浏览在没有 'Disabled cache' 情况下，接收到etag后，同一个url第二次请求就会自动带上 'If-None-Match'
`Last-Modified: Fri, 27 Jul 2019 11:04:55 GMT`  |  服务器发来的当前资源最后一次修改的时间，下次请求时，如果服务器上当前资源的修改时间大于这个时间，就返回新的资源内容
`Content-Type: text/html; charset=utf-8`  |  如果返回的是流式的数据，我们就必须告诉浏览器这个头，不然浏览器就会下载这个页面，同时告诉浏览器是utf8解码，否则可能出现乱码
`Content-Encoding: gzip`  |  告诉客户端，应该采用gzip对资源进行解码
`Connection: keep-alive`  |  告诉客户端服务器的tcp连接也是一个长连接



### Cookie 与 Session

#### 1. Cookie

Cookie是存储在用户本地计算机上，用于保存一些用户操作的历史信息吗，当用户再次访问我们的服务器的时候，浏览器通过HTTP协议，将他们本地的Cookie内容也发送到服务器上，从而完成验证。

+ Cookie是存储在浏览器客户端的一小片数据
+ Cookie可以同时被前端与后端操作
+ Cookie可以跨页面读取
+ Cookie是不可以跨服务器访问的
+ Cookie有限制：每个浏览器存储的个数不能超过300，每个服务器不能超过20，数据量不能超过4K
+ Cookie是有生命周期的，默认与浏览器相同，如果进程退出，cookie也会被销毁

#### 2. Session

Session 存储在服务器上，就是在服务器上保存用户的操作信息。

当用户访问网站时，服务器会生成一个 Session ID，然后把 Session ID存储起来，再把这个 Session ID 发给用户，用户再次访问服务器时，使用这个 Session ID就能验证了。

+ session数据存储在服务器端
+ 每一个会话分配一个单独的session_id
+ 前端只读session的id，不能修改
+ 使用Session之前需要先开启会话
+ Session存储方式比较安全，但是如果Session的个数越多，会导致服务器性能下降



### HTTP版本

#### HTTP/1.1 优点

<strong>1. 增加持久性连接</strong>

  也就是多个请求和响应可以利用同一个TCP连接，而不是每一次请求响应都要新建一个TCP连接，减少了建立和关闭连接的消耗和延迟。

<strong>2. 增加管道机制</strong>

  请求可以同时发出，但是响应必须按照请求发出的顺序依次返回，性能在一定程度上得到改善。

<strong>3. 分块传输</strong>

  在HTTP/1.1中，可以不必等到数据完全处理完毕再返回，服务器产生部分数据，那么就发送部分数据，可以节省等待时间。

<strong>4. 增加host字段</strong>

  使得一个服务器能够用来创建多个 Web 站点

<strong>5. 错误提示</strong>

  HTTP/1.1 引入了一个 Warning头域，增加对错误或警告信息的描述。此外，在 HTTP/1.1 中新增了24个状态响应码(100，101, 203，205，301,305)

<strong>6. 带宽优化</strong>

  (1). HTTP/1.1 中请求信息中引入了 range 头域，它允许只请求资源的某个部分。
  (2). 在响应消息中 Content-Range 头域声明了返回的这部分对象的长度。如果服务器相应地返回了对象所请求范围的内容。则响应码为 206，它可以防止 Cache 将响应误以为是完整的一个对象。
  (3). HTTP/1.1加入了一个新的状态码100，客户端事先发送了一个只带头域的请求，如果服务器因为权限拒绝了请求，就回送响应码401，如果服务器接收此请求就回送响应码100，客户端就可以继续发送带实体的完整请求了。

#### HTTP/1.1缺点

<strong>1. 队头阻塞</strong>

  网络延迟问题主要由于队头堵塞导致，虽然通过持久性连接得到改善，但是每一个请求的响应依然需要按照顺序排队，如果前面的响应处理较为耗费时间，那么同样非常耗费性能。

<strong>2. 浪费资源</strong>

  HTTP/1.1 请求会携带大量冗余的头信息，浪费了很多宽带资源。

#### HTTP/2.0新特性

+ <strong>二进制分帧</strong>

  + 在应用层和传输层之间增加一个二进制分帧层，从而突破 HTTP/1.1 的性能限制，改进传输性能，实现低延迟和高吞吐量。

+ <strong>多路复用(连接共享)</strong>

  + 允许同时通过单一的HTTP/2 连接发起多重的请求-响应消息，这个强大的功能则是基于'二进制分帧' 的特性。
  + 每个数据流以消息的形式发送，而消息由一或多个帧组成。这些帧可以乱序发送，然后在根据每个帧头部的流标识符号(stream_id) 重新组装。

+ <strong>首部压缩</strong>

  + HTTP/1.1 不支持 header 数据的压缩。HTTP/2.0 使用 HPACK 算法对 header的数据进行压缩。这样数据体积小了，在网络上传输就会更快，高效的压缩算法可以很大的压缩header。减少发送包的数量从而降低延迟。

+ <strong>服务器推送</strong>

  + 在HTTP/2.0中，服务器可以对客户端的一个请求发送多个响应。即服务器可以额外的向客户端推送资源，而无需客户端明确的请求。



### HTTPS

#### 1. HTTPS简介

  + HTTPS并非是应用层的一种新协议，只是HTTP通信接口部分采用SSL和TLS协议代替而已。
  + 一般 HTTP和TCP通信，当使用SSL时，则演变成先和SSL通信。在由SSL和TCP通信。
  + 在采用SSL后，HTTP就拥有了HTTPS的加密、证书和完整性保护等功能。
  + HTTPS协议的主要功能基本依赖于 TLS/SSL 协议。TLS/SSL 的功能实现主要依赖于三类基本算法：<strong>散列函数、对称加密和非对称加密</strong>。其利用非对称加密实现身份认证和密钥协商。对称加密算法采用协商的密钥对数据加密，基于散列函数验证信息的完整性。

#### 2. HTTPS工作原理

HTTPS 其实是有两部分组成： HTTP + SSL/TLS，也就是在HTTP上又加了一层处理加密信息的模块，服务端和客户端的信息传输都会通过TLS进行加密，所以传输的数据都是加密后的数据。



![https.jpg](https://i.loli.net/2020/03/03/xGulteEbXFU8I1n.jpg)



<strong>1. 客户端发起HTTPS请求</strong>

  + 浏览器里面输入一个HTTPS网址，然后连接到服务端的443端口上，注意这个过程中客户端会发送一个密文族给服务端，密文族是浏览器所支持的加密算法的清单。

<strong>2. 服务端配置</strong>

  + 采用HTTPS协议的服务器必须要有一套数字证书，可以自己制作，也可以向组织申请。区别就是自己颁发的证书需要客户端验证通过才可以继续访问，而使用受信任的公司申请的证书则不会弹出提示页面。
  + 这套证书其实是一对公钥和私钥，公钥就是一把锁头，私钥就是锁头的钥匙，锁可以给某个东西加锁，但是加锁完成后，只有持有这把锁的钥匙才可以解锁查看加锁的内容。

<strong>3. 传送证书</strong>

  + 证书其实是公钥，只是包含了很多信息，如证书的颁发机构，过期时间等。

<strong>4. 客户端解析证书<strong>

  + 这部分工作是由客户端的TLS来完成的，首先会验证公钥是否有效，如颁发机构、过期时间等。如果发现异常则会弹出一个警告框，提示证书存在问题，如果证书没有问题，那么就生成一个随机值，然后用证书对该随机值进行加密。
  + 证书中会包含数字签名，该数字签名是加密过的，是用颁发机构的私钥对本证书的公钥，名称以及其他信息做hash散列加密而生成的，客户端浏览器会首先找到该证书的根证书颁发机构。如果有，则用该根证书的公钥解密服务器下发的证书，如果不能正常解密，则会发现异常，证明该证书是伪造的。

<strong>5. 传送加密信息</strong>

  + 传送用证书加密后的随机值，目的就是让服务器得到随机值，然后客户端和服务端的通信就可以通过该随机值来进行加密和解密。

<strong>6. 服务端解密信息</strong>

  + 服务端用私钥解密后，得到了客户端传过来的随机值，到此一个非对称加密的过程结束，看到TLS利用非对称加密实现了身份认证和密钥协商。然后把内容通过该值进行对称加密。

<strong>7. 传输加密后的信息</strong>

  + 在服务端用随机值加密后的信息，可以在客户端被还原。

<strong>8. 客户端解密信息</strong>

  + 客户端用之前生成的随机值解密服务端传来的信息，获取解密后的内容，到此一个对称加密的过程结束。对称加密是用于对服务器待传送给客户端的数据进行加密用的。

#### 3. HTTP和HTTPS的共同点和区别

  + HTTPS协议需要到CA申请证书。
  + HTTP是超文本传输协议，信息是明文传输，HTTPS则是具有安全性的SSL加密传输协议。
  + HTTP和HTTPS使用不同的连接方式，端口也有区别，前者是80，后者是443。
  + HTTP的连接是无状态的，HTTPS协议是由 SSL + HTTP协议构建的可进行加密传输、身份认证的网络协议。相对HTTP而言较为安全。



### 跨域

跨域，指的是浏览器不能执行其他网站的脚本，它是由浏览器的同源策略造成的，是浏览器对JavaScript施加的安全限制。

#### 同源策略

+ 概述

  + SOP(Same origin policy)是一种约定。由 Netscape公司在1995年引入浏览器，它是浏览器最核心也是最基本的安全功能，如果没有同源策略，浏览器很容易受到 XSS, CSFR等攻击，所谓同源是指 ‘协议+域名+端口’三者相同，即便两个不同的域名指向同一个ip地址，也非同源。

+ 行为

  + Cookie、LocalStorage和IndexDB无法读取
  + DOM和JS对象无法获取
  + Ajax请求发送不出去

+ 场景

  + http://www.a.cn/index.html 调用  http://www.a.cn/server.php  非跨域。
  + http://www.a.cn/index.html 调用   http://www.b.cn/server.php  跨域，主域不同。
  + http://abc.a.cn/index.html 调用   http://def.b.cn/server.php  跨域，子域名不同。
  + http://www.a.cn:8080/index.html 调用   http://www.a.cn/server.php  跨域，端口不同。
  + https://www.a.cn/index.html 调用   http://www.a.cn/server.php  跨域,协议不同。
  + localhost 调用 127.0.0.1 跨域。

#### 解决方案

+ jsonp
+ document.domain + iframe
+ window.name + iframe
+ location.hash + iframe
+ postMessage
+ 跨域资源共享CORS
+ withCredentials属性
+ WebSocket协议
+ node代理
+ nginx代理

参考资料：  [《前端常见跨域解决方案（全）》](https://github.com/mqyqingfeng/Blog/issues/2) 



### HTTP缓存

#### HTTP缓存相关的header

头部  | 优势和特点  |  劣势和问题
----    |   ------------   |   ---------
Expires  |  1. HTTP1.0产物，可以在HTTP1.0，1.1中使用。2. 以时刻标识失效时间  |  1.时间是由服务器发送的(UTC)，如果服务器时间和客户端时间存在不一致，可能会出现问题。2. 到期之前的修改客户端是不可知的。
Cache-Control  |  1. HTTP1.1产物，以时间间隔标识失效时间，解决了Expires服务器和客户端相对时间问题。2. 比Expires多了很多选项设置。  |  1. HTTP1.1 才有的内容，不适用于1.0。2. 到期之前的修改客户端是不可知的。
Last-Modified  |  1. 不存在版本问题，每次请求都会去服务器进行校验。服务器对比最后修改时间如果相同返回304，不同返回200以及对应资源内容。  |  1. 只要资源修改，无论内容是否发生实质性变化，都会将该资源包含的数据实际上一样的。2. 以时刻作为标识，无法识别一秒内进行多次修改的情况。3. 某些服务器不能精确的得到文件的最后修改时间。
ETag  |  1. 可以更加精确的判断资源是否被修改，可以识别一秒内多次修改的情况。2. 不存在版本问题，每次请求都会去服务器进行校验。  |  1. 计算ETag值需要性能消耗。2.分布式服务器存储的情况下，计算ETag的算法不一致的话，会导致浏览器在服务器之间获取的内容发现ETag不匹配的情况。



#### 强缓存和协商缓存

+ 强缓存

  + 强缓存是利用HTTP头部的 Expires 和 Cache-Control 两个字段来控制的，用来表示资源的缓存时间。
  + 普通刷新会忽略它，但是不会清除它，需要强制刷新。浏览器强制刷新，请求会带上 Cache-Control: no-cache 和 Progma: no-cache;
  + 通常强缓存不会向服务器发送请求，直接从缓存中读取资源。一般可以看到该请求返回200的状态码，分为 from disk cache 和 from memory cache.

    + from disk cache: 一般非脚本会存在内存当中，如css，html等。
    + from memory cache: 资源在内存中，一般为脚本，字体，图片等。


+ 协商缓存

  + 由服务器来确定缓存资源是否可用，所以客户端与服务器端要通过某种标识来进行通信，从而让服务器判断请求资源是否可以缓存访问。
  + 普通刷新会启动弱缓存，忽略强缓存。只有在地址栏或收藏夹输入网址、通过链接引用资源等情况下，浏览器才会启动强缓存。
  + 协商缓存的两组 header 字段： ETag 和 If-None-Match、Last-Modified 和 If-Modified-Since


+ 流程



![cache.png](https://i.loli.net/2020/03/03/xTsfbhrQmZ326CY.png)



以上为HTTP/HTTPS的一些基础知识，还有关于版本的特点，以及HTTP缓存的机制和Header信息。

