---
layout: post
title: '网络中HTTP/TCP/UDP的应用'
date: 2018-02-19
author: 李大鹏
cover: ''
tags: iOS 网络
---

### 一、 HTTP，超文本传输协议

#### 1. 请求/响应报文

- 请求报文
  ![](http://files.pandaleo.cn/0eca23c66636aed2bcac6ff4008a5aae.png?imageMogr2/thumbnail/!68p)

  - 方法
  - 日常：GET、POST
  - GET 和 POST 区别？普通总结。
  - GET 请求参数以？分割拼接到 URL 后面，POST 请求参数在 Body 里面
  - GET 参数长度限制 2048 个字符，POST 一般没有该限制
  - GET 请求不安全，POST 请求比较安全

              * 从语义的角度总结。语义：协议定义规范，语法：具体实现路径。
                  * GET：获取资源
                      * 安全的
                      * 幂等的
                      * 可缓存的

                  * POST：处理资源
                      * 非安全的
                      * 非幂等的
                      * 不可缓存的

                  * 安全性：不应该引起Server端的任何状态变化。GET、HEAD、OPTIONS。
                  * 幂等性：同一个请求方法执行多次和执行一次的效果完全相同。PUT、DELETE。
                  * 可缓存性：请求是否可以缓存。GET、HEAD。

          * HEAD、PUT、DELETE、OPTIONS

    - URL：地址
    - 协议版本：v1.1
    - CRLF：首部字段名，多个构成首部字段区域
    - 实体主题：Post 有，Get 五

- 响应报文
  ![](http://files.pandaleo.cn/add5b5a10bdfd848bf27a4a1f343c251.png?imageMogr2/thumbnail/!68p)
  - 状态码
  - 1xx
  - 2xx，200 响应成功
  - 3xx，301、302 发生了网络重定向
  - 4xx，401、404，客户端发起的请求本身存在问题
  - 5xx，681、682，Server 端存在异常

#### 2. 连接建立流程

![](http://files.pandaleo.cn/6e1cc04025f93707e9f73d5fbfb12ba3.png?imageMogr2/thumbnail/!68p)

- 通过 TCP 的三次握手建立连接
- 在连接上进行 HTTP 请求和响应的传递
- 通过 TCP 的四次挥手释放连接

#### 3. HTTP 的特点

- 无连接

  - HTTP 的持久连接
    ![](http://files.pandaleo.cn/f07302bca2b8524559deb1a0d0445c33.png?imageMogr2/thumbnail/!68p)
  - HTTP 提供持久连接方案为了进行连接复用，节省 TCP 连接的数量，提高连接效率。
  - 头部字段
  - Connection：keep-alive，客户端期许采用持久连接
  - time：20，20s 以内不会进行四次挥手关闭，20s 以内可以复用 TCP 连接
  - max：10，这条连接最多可以发生的 HTTP 请求/响应对

          * 怎样判断一个请求是否结束？
              * Content-length：1024，响应报文头部字段，响应数据字节数到达Content-length，说明HTTP请求响应接收完毕，HTTP请求结束。
              * chunked，结束时最后会有一个空的chunked。

    - Charles 抓包原理
      - 中间人攻击，利用 HTTP 中间人攻击的漏洞实现的。

  ![](http://files.pandaleo.cn/659fbdc3b8c419d8c6fe8b3055ea5fa7.png?imageMogr2/thumbnail/!68p)  
  当客户端发送 HTTP 请求时，由中间人 Hold 住，假冒客户端的身份，发送同样的请求给 Server 端。Server 端返回同样的结果给中间人，再由中间人返回给客户端，中间人可以篡改客户端和 Server 段请求和返回的数据。

- 无状态
  - 多次发送 HTTP 请求时，Server 端不知道是否为同一个用户
  - Cookie / Session

### 二、HTTP 与网络安全

#### 1. HTTPS 和 HTTP 的区别

- HTTPS = HTTP + SSL/TLS
  ![](http://files.pandaleo.cn/abe5b2ee52d3d9d70b0b36472133dc99.png?imageMogr2/thumbnail/!68p)
  HTTPS 是安全的 HTTP，在传输层 TCP 和应用层 HTTP 之间，通过 SSL/TLS 协议保障

#### 2. HTTPS 连接建立流程

![](http://files.pandaleo.cn/58bd3b65e6c0a83fd226bb0c9bdf4048.png?imageMogr2/thumbnail/!68p)

- 会话密钥 = random S + random C + 预主秘钥（对称加密）
- HTTPS 使用的加密方式

  - 连接建立过程使用非对称加密，非对称加密很耗时！加密和解密方式不一样，在连接建立时使用，后续使用对称加密，减少损耗。
  - 后续通信过程使用对称加密

- 非对称加密
  ![](http://files.pandaleo.cn/b705f78283abc6b37fff6e3f4b59314e.png?imageMogr2/thumbnail/!68p)
  加密用公钥，解密用私钥；加密用私钥，解密用公钥。两把钥匙不一样。
- 对称加密
  ![](http://files.pandaleo.cn/d0c7410f80ed193520f60a1bd0842ca6.png?imageMogr2/thumbnail/!68p)
  用同一把对称密钥解密。不能保证绝对安全，密钥可能被中间人攻击劫持。

### 三、TCP / UDP

#### 1. UDP（用户数据报协议）

- 特点

  - 无连接
  - 尽最大努力交付
  - 面向报文。  
     既不合并，也不拆分  
    ![](http://files.pandaleo.cn/14a5efb67ac22a4ae926f1bf764ad904.png?imageMogr2/thumbnail/!68p)

- 功能
  - 复用、分用
    ![](http://files.pandaleo.cn/9f710263bfe5d4ccbd8a70629844616b.png?imageMogr2/thumbnail/!68p)
  - 差错检测  
    ![](http://files.pandaleo.cn/7d183d36c579a532e3b82a84dcaa02d3.png?imageMogr2/thumbnail/!68p)
  - 检验和位于 8 字节 UDP 首部末尾，当接收方接收到 UDP 数据时，采用同样的办法计算检验和，和传递来的检验和比对。
  - IM 即时通讯软件开发时，消息传递时刻借鉴 UDP 差错检测的策略。

#### 2. TCP（传输控制协议）

- 特点
- 面向连接
- 三次握手。数据传输开始之前，需要建立连接。
  ![](http://files.pandaleo.cn/06b739c454203e81ccf3c7e46928806a.png?imageMogr2/thumbnail/!68p)
  当 SYN 发送超时，如果没有三次握手，会直接建立连接。再次收到超时重传策略的 SYN 时，会建立两次 TCP 连接。

      * 四次挥手。数据传输结束之后，需要释放连接。

  ![](http://files.pandaleo.cn/1cbde76f277489286e81063b29977621.png?imageMogr2/thumbnail/!68p)
  TCP 通道全双工，双方都可以进行发送和回复，所以需要双方关闭。

- 可靠传输
  - 无差错、不丢失、不重复、按序到达
  - 停止等待协议
  - 无差错情况
    ![](http://files.pandaleo.cn/e09c7cad7bdc83a701683cd136ecfa1d.png?imageMogr2/thumbnail/!68p)
  - 超时重传
    ![](http://files.pandaleo.cn/dd1ebf4ac034b254ab01cc6ab0e19b6f.png?imageMogr2/thumbnail/!68p)
    在客户端设置超时定时器，如果在规定时间内没有收到对应分组报文的确认，开启超时重传机制，进行报文的重传。
    差错校验、不丢失
  - 确认丢失
    ![](http://files.pandaleo.cn/9b63cf51f870db210bd29fd5436e9164.png?imageMogr2/thumbnail/!68p)
    客户端确认报文丢失，客户端超时重传，服务端丢弃重传的报文，重传确认。
  - 确认迟到  
    ![](http://files.pandaleo.cn/a6fd3fda07f653d4601e683a830899b3.png?imageMogr2/thumbnail/!68p)

* 面向字节流
  ![](http://files.pandaleo.cn/aa75fc8a7aa102e39073e8ad5398c4e0.png?imageMogr2/thumbnail/!68p)

  - 发送方和接收方都有缓冲区，发送方通过 TCP 连接通道传递给接收方。
  - 发送方发送数据到缓冲区时，具体发送字节由 TCP 自行控制划分。如：H56、H789。

* 流量控制

  - 滑动窗口协议
    ![](http://files.pandaleo.cn/a4e93a9de8e55ad58c62a33ea653ba98.png?imageMogr2/thumbnail/!68p)
  - 描述：发送方为 Server 端，接收方为客户端，当我们继续发送数据时，可能由于接收方的接收缓存不够，如果发送方窗口过大会以很快的速率发送，需要接收窗口通过修改 TCP 首部中的窗口值调整发送方的发送速率。
  - 发送窗口前沿、后沿，发送窗口的最左边和最右边
  - 发送窗口后沿和最后发送的字节之间，是可以发送、尚未发送的字节
  - 发送窗口大小由发送缓存和接收方接收窗口的大小控制，接收方根据自己的处理能力，调整接收方的发送窗口，决定发送方的发送速率和大小。
  - TCP 报文首部中包含发送窗口和接收窗口大小的字段，通过修改值完成控制速率的目的
  - 首先提交按序到达的报文，接收窗口中的数据需要报文完整才会提交
  - 接收窗口大小受限于接收缓存，可以通过报文首部中的接收窗口值反向制约发送方发送窗口大小，控制发送速率。
  - 按序到达可通过字节序号进行控制

* 拥塞控制

  - 慢开始、拥塞避免
    ![](http://files.pandaleo.cn/bd95edbcf09f3ba32dd9089b731c0203.png?imageMogr2/thumbnail/!68p)
  - 横轴：交互次数、轮询次数；纵轴：窗口值大小。
  - 开始先发送 1 个报文，如果不拥塞发送 2 个报文，继续指数型翻倍增长。
  - 当增长到窗口门限值时，采用假发增大的方式，调整为线性增长，进行拥塞避免。 \* 当产生网络拥塞时（连续 3 个报文都没有收到 ACK），采用拥塞避免乘法减小测序，恢复到只发送 1 个报文，减少网络中发送报文数量，减小网络层压力，重启开始慢开始，将拥塞窗口降低一半。

    - 快恢复、快重传

  达到拥塞窗口时，回到某个门限值重新开始新的拥塞避免，不重新开始慢开始和之后的拥塞避免。

### 四、DNS 解析

#### 1. 什么是 DNS 解析？

- 域名到 IP 地址的映射，DNS 解析请求采用 UDP 数据包，切明文

#### 2. DNS 解析查询方式

- 递归查询
  “我去给你问一下”，域名服务器不知道域名对应服务器 IP 的域名时，会作为主问方询问别人，之后层层返回。
  ![](http://files.pandaleo.cn/b3bfc0ec408e21b1c7439415448097da.png?imageMogr2/thumbnail/!68p)
- 迭代查询
  “我告诉你谁可能知道”，域名服务器告诉本地 DNS 谁可能知道
  ![](http://files.pandaleo.cn/fcbcd2721330b171a43a8fc26a2f03a5.png?imageMogr2/thumbnail/!68p)

#### 3. 存在的常见问题

- DNS 劫持
  ![](http://files.pandaleo.cn/10025d3adbbcd8ba88d2bfa3f9d7758e.png?imageMogr2/thumbnail/!68p)
  - 客户端询问本地 DNS 服务器时，由于采用 UDP 明文传输，可能存在被窃听的风险。如果被钓鱼 DNS 劫持，可能会返回错误 IP，访问错误的 Server 上的网站。
  - DNS 劫持和 HTTP 没有关系，建立在连接之前，使用 UDP 数据包，端口号 53。
  - 解决方案
  - httpDNS
    ![](http://files.pandaleo.cn/5e8b9ee38290b8390044be190042835a.png?imageMogr2/thumbnail/!68p)
    ![](http://files.pandaleo.cn/b4d08c3e1e818345fd14705abe50cc8f.png?imageMogr2/thumbnail/!68p)
    GET 请求时，采用 IP 直连的方式请求 HTTP DNS Server 服务器，如：119.29.29.29，中国最大的 DNS 中介服务器，DNSPod。参数包含：dn 需要解析的域名，ip 客户端自身 IP 地址。客户端根据返回的 IP 地址进行后续网络请求访问，规避 DNS 劫持。
  - 长连接
    ![](http://files.pandaleo.cn/275399f39d470f168ce9e4e9ad2f630e.png?imageMogr2/thumbnail/!68p)
    在客户端和 APIServer 之间建立长连 Sever（代理服务器），客户端和长连 Sever 建立 TCP 长连通道，通过内网专线进行 http 请求和响应。

* NDS 解析转发
  ![](http://files.pandaleo.cn/a9643d8ca6ded6eecea4f2257744cf13.png?imageMogr2/thumbnail/!68p)
  - 客户端请求时，运营商 DNS 服务器为了节省资源，请求其他 DNS 服务器，之后请求权威 DNS。
  - 权威 DNS 中根据不同运营商请求，针对不同网络的流量分发，返回代理 DNS 对应的 IP，出现跨网访问场景，造成请求缓慢的效率问题。

### 五、Session / Cookie

#### 1. HTTP 协议无状态特点的补偿

![](http://files.pandaleo.cn/ddba3585526edf085a05213c24597ba7.png?imageMogr2/thumbnail/!68p)

#### 2. Cookie

- 主要用来记录用户状态，区分用户；状态保存在客户端。
  ![](http://files.pandaleo.cn/5b108da0d1233960bb0de975f576dd92.png?imageMogr2/thumbnail/!68p)
- 客户端发送的 cookie 在 http 请求报文的 cookie 首部字段中
- 服务器端设置 http 响应报文的 Set-Cookie 首部字段
- 修改 Cookie 方法

  - 新 cookie 覆盖旧 cookie
  - 覆盖规则：name、path、domain 等需要与原 cookie 一致

- 删除 Cookie

  - 新 cookie 覆盖旧 cookie
  - 覆盖规则和修改一样
  - 设置 cookie 的 expires=过去的一个时间点，或者 maxAge=0

- 保证 Cookie 的安全
  - 对 Cookie 进行加密处理
  - 只在 https 上携带 Cookie
  - 设置 Cookie 为 httpOnly，防止跨站脚本攻击

#### 3. Session

- 也是记录用户状态，区分用户的；状态存放在服务器端。
- 工作流程
  ![](http://files.pandaleo.cn/2f924fa1614f170a40fd3c092362ac9a.png?imageMogr2/thumbnail/!68p)
  - 记录用户状态：如用户名、密码，通过用户状态生成 SessionId。
  - 与 Cookie 的联系和区别
  - 联系：依赖 Set-Cookie 和 Cookie，请求和响应报文字段。
  - 区别：Cookie 存储在客户端，Session 存储在服务端。

#### 六、总结

#### 1. 从语义上总结 http 中 GET 和 POST 方式。

- 安全性、幂等性、缓存性。

#### 2. HTTPS 链接建立流程。

- 客户端发送到服务端支持的加密算法列表，包含 tls 版本号、随机数 c；服务端返回证书，包含商定的加密算法。
- 首先通过非对称加密进行对称加密的密钥传输，后续通过 HTTP 网络请求通过被非对称加密保护的对称加密进行网络王文。

#### 3. TCP 和 UDP 的区别。

- TCP 面向连接，支持可靠连接，面向字节流，提供流量和拥塞控制。
- UDP 无连接，只简单提供了复用、分用，差错检测，基本传输层功能。

#### 4. TCP 的慢开始过程。

- TCP 拥塞控制，慢开始拥塞算法。

#### 5. 客户端避免 DNS 劫持方案。

- httpDNS 和长连接。
