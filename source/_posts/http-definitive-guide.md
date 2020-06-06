---
title: 「HTTP权威指南」学习笔记
catalog: true
date: 2020-05-31 19:22:48
subtitle:
header-img:
tags:
- net
categories:
- 工程
---
> 书籍豆瓣链接：[《HTTP权威指南》](https://book.douban.com/subject/10746113/)
> 
> 相关资料笔记：[《HTTP权威指南》概念手册](https://github.com/SoaringhawkCheng/http)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

# 前言

《HTTP权威指南》中的主要概念和内容都在[《HTTP权威指南》概念手册](https://github.com/SoaringhawkCheng/http)中，本笔记对部分知识点进行了提炼和补充

# 第一部分 HTTP基础

## 第1章 HTTP概述

### URI

有两个子集URL和URN，URL受路径变化影响，URN不会

### 事务

一个HTTP事务由一条请求命令和一条响应结果组成

### web结构组件

隧道 连接通道，http信道上发送非http协议资源

用户Agent代理 代表用户发起HTTP请求的客户端程序，所有发布Web请求的应用程序都是HTTP Agent代理，比如Web浏览器

## 第2章 URL与资源

### url编码

为了避开安全字符集表示法带来的限制，使用编码机制，对不安全字符进行转义

## 第3章 HTTP报文

### 方法

HEAD 获取文档首部

TRACE 对可能经过代理服务器传送到服务器上的报文进行追踪

OPTIONS 请求服务器告知其支持什么功能

### 状态码
[HTTP状态码详解与选用](https://segmentfault.com/a/1190000006058316)
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/http-definitive-guide/code.png?raw=true)
#### 2xx 成功状态码
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/http-definitive-guide/2xx.png?raw=true)
#### 3xx 重定向状态码
状态码|原因短语|中文描述|含义|
---|---|---|---|
300|Multiple Choices|多中选择|客户端请求一个实际指向多个资源的URL|
301|Moved Permanently|永久重定向||
302|Found|临时重定向(HTTP1.0)|临时使用Location首部给出的URL来临时定位资源|
303|See Other|临时重定向(HTTP1.1)|POST方法的重定向在未询问用户的情况变成GET|
304|Not Modified|未被修改|响应条件GET请求|
307|Temporary Redireat|临时重定向(HTTP1.1)|相当于HTTP1.0的302，不会把POST转为GET|
308|Permanent Redireat|临时重定向(HTTP1.1)||

#### 4xx 客户端错误
状态码|原因短语|中文描述|含义|
---|---|---|---|
400|Bad Request|错误的请求|
401|Unauthorized|未授权||
403|Forbidden|拒绝访问|
404|Not Found|未找到||
405|Method Not Allowd|方法禁用|
407|Proxy Authentication Required|需要代理授权|见第十二章|
408|Request Timeout|超时提醒||

#### 5xx服务端错误
状态码|原因短语|中文描述|含义|
---|---|---|---|
500|Internal Server Error|服务器内部错误|所请求的服务器遇到意外的情况并阻止其执行请求|
501|Not Implemented|请求未实现|只有GET和HEAD是要求服务器支持的，它们必定不会返回此错误代码|
502|Bad Gateway|网关错误|表示作为网关或代理角色的服务器，从上游服务器中接收到的响应是无效的|
503|Service Unavailable|服务器不可用|通常造成这种情况的原因是由于服务器停机维护或者已超载|
504|Gateway Timeout|网关超时|当服务器作为网关，不能及时得到响应时返回此错误代码|
505|HTTP Version Not Supported|HTTP版本不支持|服务器不支持请求中所使用的HTTP协议版本|

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/http-definitive-guide/5xx.png?raw=true)

## 第4章 连接管理

### Connection首部

两个相邻的HTTP应用程序会为他们共享的连接应用一组选项，Connection首部字段中有一个逗号分隔的连接标签列表，为此连接指定了一些不会传播到其他连接的选项

### 串行连接

HTTP1.0的基本特点是：

1. 每来一个请求就开一个TCP连接，请求完了关闭连接
2. 服务器无法在客户端没有请求的情况下，向客户端推送消息

### 并行连接

同时开多个tcp连接，并行处理多个http事务。tcp连接会竞争带宽，大量连接会消耗很多内存资源

浏览器确实会使用并行连接，但是会将并行连接数限制为一个较小的值(通常是4个)

### 持久连接

* HTTP1.0

HTTP1.0设置了keepalive机制，来实现tcp连接的复用

Connection首部添加Keep-Alive，同一个四元组的tcp连接可以串行处理多个http事务

在HTTP响应报文中，返回一个Content-Length首部，告诉客户端单次请求的响应长度，与其他响应区分

使用Connection:Close在处理完请求后关闭持久连接

* HTTP1.1

HTTP1.1默认连接复用，除非显示加上Connection:Close属性

Content-Length首部的问题是，服务端需要计算响应包长度

HTTP1.1引入了chunk机制，将相应的body分块，块之间有分隔标记，使用Transfer-Encoding:chunked首部告诉客户端相应包含几个块

### 管道化连接

HTTP1.1在持久连接上引入了pipeline机制，发送完第一个请求，不必等响应到达就可以发送下一个

服务端必须按请求报文的顺序返回响应报文，即**队头阻塞**问题

* 二进制分帧

HTTP2为了解决**队头阻塞**，将TCP报文转换成二进制，并分成多个帧

每对请求和响应组成一个逻辑流，每个流的不同帧都带有流ID和帧ID

# 第二部分 HTTP结构

## 第5章 Web服务器

### 重定向

永久搬离的资源 301

临时搬离的资源 303 307

URL增强 重写URL，嵌入上下文，是事务间维持状态的方式 303 307

负载均衡 303 307

服务器关联 服务器可能会有用户的本地信息，重定向到包含客户端信息的服务器上去 303 307

## 第6章 代理

### web的中间实体

代理和网关的对比：代理的两端使用相同的协议，网关的两端使用不同的应用层协议，网关负责转换

### 为什么使用代理

主要使用代理作以下功能使用：

1. 儿童过滤器：如服务器响应的成人内容进行过滤
2. 文档访问控制：验证客户端访问某个的文件需要的证书
3. 安全防火墙：提供一个防火墙保护客户端或服务器
4. web缓存（缓存资源的副本）：对客户端响应资源的副本，节省带宽、减少网络拥堵
5. 反向代理（原始服务的替代物，能访问其他服务器，作服务器加速器使用）：反向代理伪装成原始服务器，不过与服务器不同的是反向代理还可以向其他服务器发送请求，以便实现按需定位所请求的内容！
6. 内容路由器：比如网络中实现为了一些付费用户提供更好、更快的网络速度，让请求发往缓存服务器，而没有付费的用户请求则发往更远或原始服务器！
7. 转码器（比如改变图片格式，以便更轻巧利于传输）
8. 匿名者：保护客户端隐私

## 第7章 缓存

### 缓存的优点

减少了冗余的数据传输

缓解了网络的带宽瓶颈

降低了原服务器的压力

降低了距离时延

### 缓存分类

私有缓存 web浏览器中有内建的私有缓存

公有缓存 公有缓存是特殊的共享代理服务器，被称为缓存代理服务器

### 副本新鲜度

HTTP使用文档过期和服务器再验证机制，保持已缓存数据与原服务器数据之间充分一致

已缓存对象包含服务器响应主体和原始服务器响应首部，以及其他元数据，用来记录对象停留时长，访问次数

#### 文档过期

通过特殊的HTTP Cache-Control首部和Expires首部，HTTP让原始服务器附加一个“过期日期”

#### 服务器再验证

已缓存对象过期，需要进行再验证

* 验证方法

	If-Modified-Since首部和Last-Modified响应首部，判断原始服务器文档截止期之后是否改动

	If-None-Match，根绝实体标签Etag进行版本比较
	
* 更新规则

	如果验证显示内容变化，缓存会获取一份新的文档副本
	
	如果没有，缓存只需要获取新的首部，包括新的过期时间

### 缓存和广告

因为缓存的存在，导致某些靠用户点击次数收费的广告的请求未达到原始服务，从而服务器很难对用户进行了多少次点击进行计数！一种解决方案就是配置缓存，每次访问时都与原始服务器进行再验证。这样，每次访问时都会将命中推向原始服务器，但通常不会传送任何主体数据。当然，这样会降低事务处理的速度。

## 第8章 集成电路：网关、隧道及中继

### 网关

web网关在一侧使用HTTP协议，另一侧使用另一种协议

使用<客户端协议>/<服务端协议>格式对网关进行描述

服务器端网关 使用其他协议和服务端通信，使用HTTP协议和客户端通信

客户端网关 使用其他协议和客户端通信，使用HTTP协议和服务端通信

### 协议网关

HTTP/* 服务端Web网关 将客户端HTTP请求转换为其他协议

HTTP/HTTPS 服务端安全网关

HTTPS/HTTP 客户端安全网关 谨慎使用，确保网关和原始服务器之间的网络是安全的

### 资源网关

应用程序服务器，将目标服务器和网关结合在一个服务器实现

客户端通过HTTP与应用程序服务器通信，服务器将请求通过网关应用编程接口(Application Programming, API)发送给运行在服务器上的应用程序

#### CGI
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/http-definitive-guide/cgi.gif?raw=true)

第一个流行的应用程序网关API是通用网关接口(Common Gateway Interface, CGI)。CGI是一个标准接口集，Web服务器可以用它来装载程序，响应HTTP请求，并收集输出数据，构造HTTP响应会送。

CGI工作原理：每当客户请求CGI的时候，WEB服务器就请求操作系统生成一个新的CGI解释器进程(如php-cgi.exe)，CGI 的一个进程则处理完一个请求后退出，下一个请求来时再创建新进程。当然，这样在访问量很少没有并发的情况也行。可是当访问量增大，并发存在，这种方式就不 适合了。于是就有了fastcgi。

## 第9章 Web机器人

## 第10章 HTTP-NG

# 第三部分 识别、认证与安全

## 第11章 客户端识别与cookie机制

http最初是一个匿名、无状态的请求/响应协议。服务器处理来自客户端的请求，然后向客户端回送一条响应。web服务器几乎没有什么信息可以用来判定是哪个用户发送的请求，也无法记录来访用户的请求序列。

### 解决方案

#### 客户端IP地址

存在以下缺点：

1. IP很可能不是客户端，而是代理
2. 多个用户公用一个IP
3. IP地址很容易伪造

#### 用户登陆

服务器返回报文并添加WWW-Authorization首部，客户端下次访问带上Authorization首部，验证用户身份信息

#### 胖URL

把用户信息添加到url，缺点：

1. 暴露用户信息
2. 破坏缓存
3. 跳出跳回丢失用户信息
4. 会话非持久

#### Cookie

是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上，使基于无状态的HTTP协议记录稳定的状态信息成为了可能。

Cookie规范看RFC 6265，[中文翻译](https://www.cnblogs.com/jj1106/p/11221239.html)

### Cookie用途

Cookie主要用于以下三个方面：

1. 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
2. 个性化设置（如用户自定义设置、主题等）
3. 浏览器行为跟踪（如跟踪分析用户行为等）

Cookie一度用户存储客户端数据，现在逐渐被Web Storage API和IndexedDB取代

#### Web Storage API

**sessionStorage** 为每一个给定的源（given origin）维持一个独立的存储区域，该存储区域在页面会话期间可用（即只要浏览器处于打开状态，包括页面重新加载和恢复）

**localStorage** 同样的功能，但是在浏览器关闭，然后重新打开后数据仍然存在

### Cookie分类

cookie两类会话：会话cookie和持久cookie

会话cookie退出浏览器就被删除了

### Cookie属性

Domain: cookie的域

Allh: 那些主机可以使用此cookie

Path: 那些路径能使用cookie

Secure: 是否在发送https报文的时候使用cookie

Expires: 过期时间

Name: cookie的名字

Value: cookie的值

HttpOnly: 通过js脚本无法读到cookie信息

### Cookie作用域

#### 相关字段

Domain和Path标识定义了Cookie的作用域：即Cookie应该发送给哪些URL

Domain 标识指定了哪些主机可以接受Cookie

Path 标识指定了主机下的哪些路径可以接受Cookie

#### Cookie设置

Domain属性只能设置当前域名以及他的父域名，不能设置子域名

浏览器会将domain和path都相同的cookie保存在一个文件里，cookie间用*隔开

#### Cookie访问

Cookie的作用域是Domain本身以及Domain下的所有子域名

子目录页面能访问到父目录Cookie

### Cookie首部

Cookie 客户端向服务器传送的cookie

Set-Cookie 服务器传送cookie给客户端

## 第12章 基本认证机制

### 认证流程

基本认证描述的就是用户第一次访问服务器的时候，服务器返回401状态码和WWW-Authenticate响应首部，并在首部中描述了密码编码算法和对应要使用密码的安全域

然后客户端弹框，在输入用户名和密码提交后。客户端发送含有Authorization首部的get请求申请验证身份，其中首部中携带了用户名和密码糅合在一起的编码后的验证码给服务器端，

如果验证通过，那么服务器返回200的响应，并以Authentication-Info首部携带相关信息，以后用户就可以不用密码访问相关文件了

### 安全缺陷

明文发送，即使是密文发送，也容易被解码

唯一安全使用方式是与SSL配合使用

### 代理认证

代理认证：就是代替服务器向客户端发起质询/认证，与服务器端发起的质询/认证主要有几点不同：

质询的时候返回的是407状态码，服务器端质询返回的首部是Proxy-Authenticate

客户端认证时发送的是Proxy-Authorizatio首部，认证成功之后返回的是Proxy-Authentication-Info首部。

## 第13章 摘要认证

### 单向摘要

使用常见的单向摘要函数，比如MD5对密码加密，服务端匹配客户端摘要与服务端摘要

### 随机数防止重放攻击

## 第14章 安全HTTP

### 加密技术

#### 对称密钥加密技术

编码和解码使用的密钥是同一个

#### 公开密钥加密技术

使用RSA生成两个非对称密钥Pub和Pri

Pub用来对主机报文进行编码，Pri用来给主机报文解码

Pub是公钥，Pri是只有主机才知道的私钥

### 数字签名

除了加解密报文之外，还可以对报文进行签名，说明是谁编写的报文，同时证明报文未被篡改过。这种技术被称为**数字签名**，是附加在报文上的特殊加密校验码
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/http-definitive-guide/digital-signature.png?raw=true)

### 对称加密

使用对称密钥加密技术

问题：密钥如何传输？密钥A的传输需要另一个密钥，循环

### 非对称加密
	
存在着签名和验签与加密和解密两个过程：

1. 签名和验签 私钥签名，公钥验签，目的是防篡改
2. 加密和解密 公钥加密，私钥解密，防止信息被拦截偷听

### 中间人攻击
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/http-definitive-guide/mitm.jpeg?raw=true)

中间人攻击(Man-in-the-MiddleAttack, MITM)，客户端和服务器交换公钥，各自把自己的公钥发给对方，但被中间人劫持了

中间人C用PubC替换PubA，发给服务器，用PubC替换PubB，发给客户端

客户端和服务器之间的消息，使用PubC加密，中间人可以使用PriC解密

### 数字证书

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/http-definitive-guide/ca.png?raw=true)
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/http-definitive-guide/mail.png?raw=true)

公钥的传输过程是不安全的，比如中间人攻击，需要验证收到的公钥是对方发出的，这就需要**数字证书**

#### 服务端证书获取

服务器A将自己的公钥PubA+个人信息发给CA，CA用CA私钥为服务器生成一个数字证书

#### 客户端证书检查

客户端预先信任一些CA机构，并保存好这些CA的公钥

客户端B收到后，用CA的公钥解密数字证书，拿到A的公钥，然后验证数字签名

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/http-definitive-guide/ca-process.png?raw=true)

#### 根证书与CA信任链

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/http-definitive-guide/ca-chain.jpg?raw=true)

数字证书使用CA私钥进行加密，同样需要证明CA的公钥不是伪造的

解决方法是给CA颁发证书，将CA的公钥发送给上级的证书中心，颁发证书给当前的CA

这样就形成了一条CA信任链，一直到根证书Root CA

根证书是自验证证书，Root CA机构是公认的第三方权威机构

### HTTPS

HTTPS是应用层协议，默认运行在443端口，是一种安全的传输协议

通过在HTTP层与运输层的TCP直接加入一个加密/身份验证层来保证安全传输

HTTPS实际上就是HTTP协议和SSL/TSL协议的组合

HTTP是文本协议，HTTPS是二进制协议

### SSL/TLS协议
安全套接字协议(Secure Sockets Layer, SSL)，及其继任者传输层安全（Transport Layer Security, TLS）是为网络通信提供安全及数据完整性的一种安全协议

TLS与SSL在传输层与应用层之间对网络连接进行加密。

SSL协议位于TCP/IP协议与各种应用层协议之间，为数据通讯提供安全支持

#### SSL/TLS记录协议

建立在可靠的传输协议（如TCP）之上，为高层协议提供数据封装、压缩、加密等基本功能的支持

#### SSL/TLS记录协议

用于在实际的数据传输开始前，通讯双方进行身份认证、协商加密算法、交换加密密钥等。

#### 四次握手[TO RECITE]
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/http-definitive-guide/ssl-handshaking-0.png?raw=true)

1. 一次握手：

	步骤 1：客户端通过发送 Client Hello 报文开始 SSL通信。 报文中包含客户端支持的 SSL的指定版本、 加密组件（Cipher Suite） 列表（所使用的加密算法及密钥长度等） 。

2. 二次握手：

	步骤 2：服务器可进行 SSL通信时， 会以 Server Hello 报文作为应答。 和客户端一样， 在报文中包含 SSL版本以及加密组件。 服务器的加密组件内容是从接收到的客户端加密组件内筛选出来的。

	步骤 3：之后服务器发送 Certificate 报文。 报文中包含公开密钥证书。

	步骤 4：最后服务器发送 Server Hello Done 报文通知客户端， 最初阶段的 SSL握手协商部分结束。

3. 三次握手：

	步骤 5：客户端以 Client Key Exchange 报文作为回应。 报文中包含通信加密中使用的一种被称为 Pre-mastersecret 的随机密码串。 该报文已用步骤 3 中的公开密钥进行加密。

	步骤 6：接着客户端继续发送 Change Cipher Spec 报文。 该报文会提示服务器， 在此报文之后的通信会采用Pre-master secret 密钥加密。

	步骤 7：客户端发送 Finished 报文。 该报文包含连接至今全部报文的整体校验值。 这次握手协商是否能够成功， 要以服务器是否能够正确解密该报文作为判定标准。

4. 四次握手：

	步骤 8：服务器同样发送 Change Cipher Spec 报文。

	步骤 9：服务器同样发送 Finished 报文。

	总的来说，SSL会话的建立完成了以下几个工作：

	交换协议版本号，选择一个通信双方都支持的加密方式；对两端实现身份验证；密钥交换。

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/http-definitive-guide/ssl-handshaking.png?raw=true)

# 第四部分 实体、编码和国际化

## 第15章 实体和编码

服务器通过HTTP协议的Content-Type首部中的charset参数和Content-Language首部告知客户端文档的字母表和语言。

### 实体摘要

发送方可以在生成初始的主体时，生成一个数据的校验和。遮掩接收方就可以通过检查这个校验和来捕获所有意外的实体修改了

Content-MD5首部发送对实体主体运行MD5摘要算法的结果

### 媒体类型和字符集

Content-Type 说明的是原始实体主体的MIME类型

参数charset表示字符集，说明把实体中的比特转换为文本文件中的字符方法

## 第16章 国际化

Content-Language 客户端文档的语言

Accept-Language 客户端能优先接收的语言

## 第17章 内容协商与转码

# 第五部分 内容发布与分发

## 第18章 Web主机托管

## 第19章 发布系统

## 第20章 重定向与负载均衡

## 第21章 日志记录与使用情况跟踪

# 第六部分 附录