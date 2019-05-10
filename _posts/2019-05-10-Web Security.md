---
layout:     post
title:      Web Security
subtitle:   Web安全
date:       2019-05-10
author:     Sun8min
header-img: 
catalog: true
tags:
    - Web Security
    - Web安全
    - CSRF
    - DDOS
    - SQL注入
    - XSS
    - 中间人攻击
---

# Web Security

---
## CSRF
###### [wiki链接](https://zh.wikipedia.org/wiki/跨站请求伪造)
- 跨站请求伪造（英语：Cross-site request forgery），
- 也被称为one-click attack 或者 session riding，
- 通常缩写为 CSRF 或者XSRF， 是一种挟制用户在当前已登录
的Web应用程序上执行非本意的操作的攻击方法。
- 跟跨网站脚本（XSS）相比：

  ```text
  1. XSS利用的是用户对指定网站的信任，
  2. CSRF利用的是网站对用户网页浏览器的信任。
  3. XSS是源于代码注入方式，js引擎把输入作为脚本执行。
  4. CSRF是源于欺骗用户浏览器，让其以用户的名义运行操作。
  ```

###### example:
假如一家银行用以运行转账操作的URL地址如下： 

`http://www.examplebank.com/withdraw?account=AccoutName&amount=1000&for=PayeeName`

那么，一个恶意攻击者可以在另一个网站上放置如下代码： 

`<img src="http://www.examplebank.com/withdraw?account=Alice&amount=1000&for=Badman">`

如果有账户名为Alice的用户访问了恶意站点，而她之前刚访问过银行不久，
登录信息尚未过期，那么她就会损失1000资金。

这种恶意的网址可以有很多种形式，藏身于网页中的许多地方。
此外，攻击者也不需要控制放置恶意网址的网站。
例如他可以将这种地址藏在论坛，博客等任何用户生成内容的网站中。
这意味着如果服务端没有合适的防御措施的话，
用户即使访问熟悉的可信网站也有受攻击的危险。

在上面示例里，能够看出，
攻击者并不能通过CSRF攻击来直接获取用户的账户控制权，也不能直接窃取用户的任何信息。
他们能做到的，是欺骗用户浏览器，让其以用户的名义运行操作。

#### 主要危害：
1. 可以盗用受害者的身份，完成受害者在web浏览器对目标站点有权限进行的任何操作。

#### 主要解决办法：
###### 概要
```text
如果选择不支持低版本浏览器（ie6），检查referer字段就足够，
如果需要支持，那就添加校验token，特殊地方使用特殊token。
```

1. 检查Referer字段

HTTP头中有一个Referer字段，这个字段用以标明请求来源于哪个地址。
在处理敏感数据请求时，通常来说，Referer字段应和请求的地址位于同一域名下。
以上文银行操作为例，Referer字段地址通常应该是转账按钮所在的网页地址，
应该也位于www.examplebank.com之下。而如果是CSRF攻击传来的请求，
Referer字段会是包含恶意网址的地址，不会位于www.examplebank.com之下，
这时候服务器就能识别出恶意的访问。

这种办法简单易行，工作量低，仅需要在关键访问处增加一步校验。
但这种办法也有其局限性，因其完全依赖浏览器发送正确的Referer字段。
虽然http协议对此字段的内容有明确的规定，但并无法保证来访的浏览器的具体实现，
亦无法保证浏览器没有安全漏洞影响到此字段。并且也存在攻击者攻击某些浏览器，
篡改其Referer字段的可能。

2. 添加校验token

由于CSRF的本质在于攻击者欺骗用户去访问自己设置的地址，
所以如果要求在访问敏感数据请求时，要求用户浏览器提供不保存在cookie中，
并且攻击者无法伪造的数据作为校验，那么攻击者就无法再运行CSRF攻击。
这种数据通常是窗体中的一个数据项。服务器将其生成并附加在窗体中，
其内容是一个伪随机数。当客户端通过窗体提交请求时，
这个伪随机数也一并提交上去以供校验。正常的访问时，
客户端浏览器能够正确得到并传回这个伪随机数，而通过CSRF传来的欺骗性攻击中，
攻击者无从事先得知这个伪随机数的值，服务端就会因为校验token的值为空或者错误，
拒绝这个可疑请求。

3. 在 HTTP 头中自定义属性并验证
这种方法也是使用 token 并进行验证，和上一种方法不同的是，
这里并不是把 token 以参数的形式置于 HTTP 请求之中，
而是把它放到 HTTP 头中自定义的属性里。通过 XMLHttpRequest 这个类，
可以一次性给所有该类请求加上 csrftoken 这个 HTTP 头属性，
并把 token 值放入其中。这样解决了上种方法在请求中加入 token 的不便，
同时，通过 XMLHttpRequest 请求的地址不会被记录到浏览器的地址栏，
也不用担心 token 会透过 Referer 泄露到其他网站中去。

---
## DDOS
###### [wiki链接](https://zh.wikipedia.org/wiki/阻斷服務攻擊)
- 拒绝服务攻击（英语：denial-of-service attack，简称DoS攻击）亦称洪水攻击，
- 是一种网络攻击手法，其目的在于使目标计算机的网络或系统资源耗尽，
  使服务暂时中断或停止，导致其正常用户无法访问。
- 当黑客使用网络上两个或以上被攻陷的计算机作为“僵尸”向特定的目标发动“拒绝服务”式攻击时，
  称为分布式拒绝服务攻击（distributed denial-of-service attack，简称DDoS攻击）。
  
###### example:
1. 带宽消耗型攻击:
- UDP洪水攻击（User Datagram Protocol floods），
  UDP（用户数据报协议）是一种无连接协议，当数据包通过UDP发送时，
  所有的数据包在发送和接收时不需要进行握手验证。当大量UDP数据包
  发送给受害系统时，可能会导致带宽饱和从而使得合法服务无法请求访
  问受害系统。遭受DDoS UDP洪泛攻击时，UDP数据包的目的端口可能是
  随机或指定的端口，受害系统将尝试处理接收到的数据包以确定本地运行
  的服务。如果没有应用程序在目标端口运行，受害系统将对源IP发出ICMP
  数据包，表明“目标端口不可达”。
- ICMP洪水攻击（ICMP floods）
  ICMP（互联网控制消息协议）洪水攻击是通过向未良好设置的路由器
  发送广播信息占用系统资源的做法。攻击者向安全薄弱网络所广播的地址
  发送伪造的 ICMP 响应数据包。那些网络上的所有系统都会向受害计算机
  系统发送 ICMP 响应的答复信息，占用了目标系统的可用带宽并导致合法
  通信的服务拒绝（DoS）。
- 死亡之Ping（ping of death）
  死亡之Ping是产生超过IP协议能容忍的数据包数，系统不能够将这些数据包
  重新组合起来，若系统没有检查机制，就会死机。
2. 资源消耗型攻击
- SYN洪水攻击（Syn Flood），Ack洪水攻击（Ack Flood），
  以Syn Flood攻击为例，它利用了TCP协议的三次握手机制，
  当服务端接收到一个Syn请求时，服务端必须使用一个监听队列
  将该连接保存一定时间。因此，通过向服务端不停发送Syn请求，
  但不响应Syn+Ack报文，从而消耗服务端的资源。当等待队列被占满时，
  服务端将无法响应正常用户的请求，达到拒绝服务攻击的目的。
- CC攻击（Distributed HTTP flood，分布式HTTP洪水攻击），
  使用代理服务器向受害服务器发送大量貌似合法的请求（通常使用HTTP GET)。
  CC（Challenge Collapsar，挑战黑洞）根据其工具命名，
  攻击者创造性地使用代理机制，利用众多广泛可用的免费代理服务器发动
  DDoS攻击。许多免费代理服务器支持匿名模式，这使追踪变得非常困难。

#### 主要危害：
1. 服务暂时中断或停止，导致其正常用户无法访问。

#### 主要解决办法：
拒绝服务攻击的防御方式通常为入侵检测，流量过滤和多重验证，
旨在堵塞网络带宽的流量将被过滤，而正常的流量可正常通过。

1. CDN服务，保证静态网页的访问加速，例如[Cloudflare](https://www.cloudflare.com/) 是一个免费 CDN 服务，并提供防火墙
2. 流量清洗，当流量被送到DDoS防护清洗中心时，通过采用抗DDoS软件处理，
   将正常流量和恶意流量区分开。正常的流量则回注回客户网站。
   这样一来可站点能够保持正常的运作，处理真实用户访问网站带来的合法流量。
   
---
## SQL注入
###### [wiki链接](https://zh.wikipedia.org/wiki/SQL注入)
- SQL注入（英语：SQL injection），也称SQL注入或SQL注码，
- 是发生于应用程序与数据库层的安全漏洞。

###### example:
以一个网页有两个字段让用户输入用户名与密码为例：
```sql
SELECT UserList.Username
FROM UserList
WHERE UserList.Username = 'Username'
AND UserList.Password = 'Password'
```
如果恶意在密码栏注入某些合法代码 ("password' OR '1'='1")，查询结果便如下所示：
```sql
SELECT UserList.Username
FROM UserList
WHERE UserList.Username = 'Username'
AND UserList.Password = 'password' OR '1'='1'
```
在上面示例里，"'1'='1'"逻辑式将永远为真。

#### 主要危害：
1. 数据表中的数据外泄，例如企业及个人机密数据，账户数据，密码等。
2. 破坏硬盘数据，瘫痪全系统（例如xp_cmdshell "FORMAT C:"）。


#### 主要解决办法：
1. 在设计应用程序时，完全使用参数化查询（Parameterized Query）来设计数据访问功能。
2. 在组合SQL字符串时，先针对所传入的参数加入其他字符（将单引号字符前加上转义字符）。
```text
所谓参数化是指预编译，编译是分析sql语句，找出关键字，完成语法分析，生成执行计划。
因为完成了语法分析，所以后面传入的任何字符都不会作为sql关键字，只能作为字面量参数。
```

---
## XSS
###### [wiki链接](https://zh.wikipedia.org/wiki/跨網站指令碼)
- 跨站脚本（英语：Cross-site scripting，通常简称为：XSS）
- 是一种网站应用程序的安全漏洞攻击，是代码注入的一种。
- 它允许恶意用户将代码注入到网页上，其他用户在观看网页时就会受到影响。
- 这类攻击通常包含了HTML以及用户端脚本语言。

ps: 本应为CSS，但因为与网页设计CSS样式（Cascading Style Sheets）冲突，
所以将Cross（意为“交叉”）改以交叉形的X做为缩写。

###### example:
```html
<html>
    <head>
       <title>XSS测试</title>
    </head>
<body>
<div>
    <!-- 假设这里是其他用户输入的内容 -->
    <script>alert("XSS!")</script>
    <script>alert(document.cookie)</script>
</div>
</body>
</html>
```

#### 主要危害：
1. 盗用cookie，获取敏感信息。
3. 利用iframe、frame、XMLHttpRequest或上述Flash等方式，以
  （被攻击）用户的身份执行一些管理动作，或执行一些一般的如发微博、
   加好友、发私信等操作。
4. 利用可被攻击的域受到其他域信任的特点，以受信任来源的身份请求一
   些平时不允许的操作，如进行不当的投票活动。
5. 在访问量极大的一些页面上的XSS可以攻击一些小型网站，实现DDoS
   攻击的效果。

#### 主要解决办法：
1. 将用户所提供的内容进行过滤，例如Java的xssprotect (Open Source Library)。
2. 针对cookie：可以设置cookie的HttpOnly属性，指示浏览器只能在http/https
   请求下使用cookie，js无法获取操作cookie
   
---
## 中间人攻击
###### [wiki链接](https://zh.wikipedia.org/wiki/中间人攻击)
- 中间人攻击（英语：Man-in-the-middle attack，缩写：MITM）
- 在密码学和计算机安全领域中，是指攻击者与通讯的两端分别创建独立的联系，
并交换其所收到的数据，使通讯的两端认为他们正在通过一个私密的连接与对方直接对话，
但事实上整个会话都被攻击者完全控制。
- 在中间人攻击中，攻击者可以拦截通讯双方的通话并插入新的内容。

一个中间人攻击能成功的前提条件是攻击者能将自己伪装成每一个参与会话的终端，
并且不被其他终端识破。中间人攻击是一个（缺乏）相互认证的攻击。
大多数的加密协议都专门加入了一些特殊的认证方法以阻止中间人攻击。
例如，SSL协议可以验证参与通讯的一方或双方使用的证书是否是由权威的受信任的数字证书认证机构颁发，
并且能执行双向身份认证。

###### example:
假设爱丽丝（Alice）希望与鲍伯（Bob）通信。
同时，马洛里（Mallory）希望拦截窃会话以进行窃听并可能在某些时候传送给鲍伯一个虚假的消息。

首先，爱丽丝会向鲍勃索取他的公钥。
如果Bob将他的公钥发送给Alice，并且此时马洛里能够拦截到这个公钥，就可以实施中间人攻击。
马洛里发送给爱丽丝一个伪造的消息，声称自己是鲍伯，并且附上了马洛里自己的公钥（而不是鲍伯的）。

爱丽丝收到公钥后相信这个公钥是鲍伯的，于是爱丽丝将她的消息用马洛里的公钥（爱丽丝以为是鲍伯的）加密，
并将加密后的消息回给鲍伯。马洛里再次截获爱丽丝回给鲍伯的消息，并使用马洛里自己的私钥对消息进行解密，
如果马洛里愿意，她也可以对消息进行修改，然后马洛里使用鲍伯原先发给爱丽丝的公钥对消息再次加密。
当鲍伯收到新加密后的消息时，他会相信这是从爱丽丝那里发来的消息。

#### 主要危害：
1. 与服务器的通信被第三方解密、查看、修改。

#### 主要解决办法：
1. 针对web：使用https，用受信任的证书颁发机构（CA）颁发的证书