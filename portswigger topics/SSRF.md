# Server-side request forgery(SSRF)

服务器端请求伪造

## What is SSRF?

SSRF是一种网络安全漏洞，允许攻击者使服务器端应用程序向非预期位置发出请求。

**简单来说：**让服务器去帮我访问它本不该访问的地址。

当一个网站提供了“由服务器去访问某个链接”的功能（比如，下载照片，获取网页内容，验证URL是否可用等），攻击者就会利用这个功能，让服务器去访问他们指定的任意地址。

如果这个网站没有对这个“地址”做严格限制，就会发生SSRF。



在典型的SSRF攻击中，攻击者可能会使服务器连接到组织基础设施内仅内部的服务。在其他情况下，他们可能能够强制服务器连接到任意的外部系统。这可能会泄露敏感数据，比如授权证书等。

##### **SSRF的危险**：

服务器通常有：

- 内网访问权限
- 本机权限
- 高安全级别的资源访问权限

攻击者就可能让服务器去访问一些本应该被保护的资源：

- 内网服务（如数据库，管理端口，内部API）
- 服务器本机敏感接口（例如：http://127.0.0.1/admin)



## What is the impact of SSRF at

## tacks?

成功的SSRF攻击通常会导致组织内部的未授权作或数据访问。

这可以是在易受攻击的应用程序中，也可以是在应用程序能够通信的其他后端系统中，在某些情况下，SSRF漏洞可能允许攻击者执行任意命令。

SSRF漏洞利用外部第三方系统连接，可能导致恶意的后续攻击。这些问题可能看起来来自托管该易受攻击应用的组织。



## Common SSRF attacks

### 基础概念：

<u>SSRF攻击常利用信任关系，将攻击升级到受攻击应用的层级化，并执行未经授权的操作。这些信任关系可能存在于服务器之间，也可能存在于同一组织内的其他后端系统之间。</u>

##### 信任关系：

在网络/系统里，A信任B表示A接受来自B的请求，身份或流量，而不做严格的二次验证。信任可以是基于网络位置，证书，IP白名单，服务账户或内部网络可达性等。

常见形式：

- 防火墙/访问控制 只放行内网IP（把“来自内网”的请求当成可信）
- 服务之间用内部的API key,相同VPC，或mutual TLS，相互认为来自内部的连接是可信任的。
- 元数据/管理接口只在本机(localhost)可访问，默认认为只有本机进程会请求它。

##### 什么是层级化？

系统通常分层(边界少——多，权限从低到高)，比如：

1. 外部用户(互联网)

2. 前端应用(WEB服务)

3. 后端微服务（内部API）

4. 数据库/管理界面/云控制台

   这些层级之间有不同的信任和访问权限

层级化升级：SSRF能把从外层推进到更高，更敏感的层级(逐层提升影响范围与权限)——不是一次性直接控制核心，而是通过逐步访问内部，后端，管理接口来“爬梯式”提升危害。

举例：

- 开始：攻击者把一个URL发给网站的“预览外部图片“功能
- 过程：服务器被诱导，访问内部API（后端层），得到接口再返回。
- 高级：通过内部API的信息或访问权限，再一步访问数据库或管理面板（最敏感层）



### SSRF attacks against the server

SSRF对服务器的攻击

在针对服务器的SSRF攻击中，攻击者使应用程序通过环回网络接口向托管该应用的服务器发送HTTP请求。这通常涉及提供带有主机名的URL，比如 `127.0.0.1` (指向环回适配器的预留IP地址)或 `localhost`(同一适配器常用的名称)。

##### 环回网络接口：

是主机内部的一条”虚拟网线/地址簇“，用来让本机与本机自己通信，数据包不会离开这条机器。

常见名称是 `lo` ，对应的地址包括IPv4的 `127.0.0.1` （以及整个 `127.0.0.0/8`段）和IPv6的 `::1`。

- 当程序向环回地址(如 `127.0.0.1`或`::1`)发送网络请求的时候，这些请求不会发到网线或者交换机，而是直接在存储系统内回环(loop back)到本机的网络栈，再由本机上监听响应端口的进程接收。

- 形象图：

  外部客户端——公网接口——应用

  本机进程A`127.0.0.1:xxxx`——环回接口——本机进程B 监听 `127.0.0.1:8080`

##### 预留IP地址：

指一组为特定用途保留，不用于公网路由的地址（如换回地址，私有网段，链路本地）。`127.0.0.1` 就是为环回保留的地址，`10.0.0.0/8` ,`192.168.0.0/16` 是为私有网络保留的地址段。



##### 实例

例如，想象一个购物应用，允许用户查看某商品再某家门店是否有库存。为了提供库存信息，应用程序必须查询各种后端 REST API。它通过前端HTTP请求将URL传递给相关的后端API端点来实现这一点。当用户查看某项商品的库存状态时，浏览器会发出以下请求：

```css
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
```

这将导致服务器向指定URL发送请求，获取库存状态，并将其返回给用户。

在这个示例中，攻击者可以修改以指定服务器本地的URL：

```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://localhost/admin
```

改变stockapi参数。

服务器获取`/admin` URL内容并将其返回给用户。

攻击者可以访问该 `/admin` URL，但管理功能通常仅对已认证的用户开放。这意味着攻击者看不到任何有趣的东西。然而对 `/admin` URL的请求来自本地机器，通常的访问控制会被绕过。应用程序授予管理功能的完全访问权限，意味请求似乎来自可信的位置。

------

#### LAB：Basic SSRF against local server

#### Question:

这个实验室有库存检查功能，可以从内部系统获取数据。

要解决实验室问题，可以更改库存检查URL以访问管理员界面，`http://localhost/admin`并删除用户`carlos`。

#### Pain POint:

- 为什么 `http://localhost/admin` 不行？而 `http://localhost/admin/delete?username=carlos` 就可以？
  **关键在于**SSRF请求由服务器发出，而不是浏览器发出，浏览器不是管理员，所以在浏览器上点击delete当然就没用了。

  所以要把delete放在URL中。

  **逻辑链路：**

  SSRF（stockApi发起）——服务器本机——localhost  所以内部可信，可以执行删除。

- payload：

  ```
  http://localhost/admin/delete?username=carlos
  ```

  服务器执行这个URL：

  - 来源变成服务器本机访问localhost
  - 地址是localhost/admin/delete
  - 正好是管理员接口允许访问的来源



#### Solved:

1. 打开burp suite，拦截查看stock的请求

2. 查看stockApi,修改其参数。

3. 让其服务器访问本地localhost,地址为localhost/admin/delete

4. ```
   http://localhost/admin/delete?username=carlos
   ```

   

------

为什么程序会这样表现，并且隐式信任来自本地机器的请求？这可能由多种原因引起：

- 访问控制检查可能在应用服务器前方的其他组件中实现。当连接回服务器的时候，检查被绕过。

  **简单来说：**
  访问控制再服务器外部实现——回环请求绕过检查

  **详细说明：**

  很多架构会把入口认证/授权放在边缘组件（例如API网关，反向代理，WAF，负载均衡器）：

  - 边缘组件负责：会话验证，IP白名单，速率限制，单点登录（SSO）等、
  - 主应用只关注业务逻辑，信任来自边缘的流量已经被”筛过“

  问题出现在于：当有用本身向内部地址（如127.0.0.1)发起请求，这个请求不会经过外部边缘（因为是在本机内部回环），于是边缘的认证/授权逻辑被绕过。也就是说，边缘负责的“谁能操作X”不再生效。

  **为什么会被信任：**

  开发运维往往认为：只有系统管理员或运维脚本会在服务器上发送这些请求，所以直接信任本机来源可以简化实现。

  

- 处于灾难恢复的目的，应用程序可能允许任何来自本地机器的用户无需登录即可访问管理员权限。这为管理员在丢失凭证时恢复系统提供了一种方式。这假设只有完全受信任的用户会直接从服务器访问。

  **简单来说：**

  为了灾难恢复而放开本机访问。

  **详细说明：**

  某些系统为了应急/运维方便，会在本机提供无需登录或较低门槛的管理接口（例如只要在服务器上访问就认为可信）。目的通常是：

  - 当远程管理凭证丢失或者SSO出问题时，运维可在控制台/SSH上直接修复。
  - 在脚本话运维时，本机调用更方便（避免复杂凭证分发）

  这个机制假设：只有运维人员可以登录主机。因此把来自主机的请求当成管理员。

  

- 管理界面可能在主应用程序的不同端口号上监听，用户可能无法直接访问。

  **简单来说：**

  管理界面监听在不同端口/不同接口上（端口/接口分离）

  为了安全，开发者通常把管理或调试界面绑定在：

  - 只监听 `127.0.0.1:XYZ` (本机可达)
  - 或监听某个内部网段接口(10.x.x.x"XYZ)
  - 或监听非标准端口（例如8081.9000），以免被普通用户访问。



此类信任关系中，源自本地机器的请求与普通请求处理方式不同，往往使SSRF成为关键漏洞





### SSRF attacks against other back-end systems

针对其他后端系统的SSRF攻击

在某些情况下，应用服务器能够与用户无法直接访问的后端系统交互。

这些系统通常采用不可路由的私有IP地址。由于后端系统通常受到网路拓扑结构的保护，其安全防护措施往往较为薄弱。许多内部后端系统包含敏感功能，任何能够与这些系统交互的人员均未经过身份验证直接访问。

在前面的示例中，假设后端URL `https://192.168.0.68/admin` 存在一个管理界面。攻击者可以提交以下请求利用SSRF漏洞，从而访问该管理界面：

```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://192.168.0.68/admin
```

------



#### LAB：Basic SSRF against another back-end system

#### Question:

这个实验室有库存检查功能，可以从内部系统获取数据。

要完成该实验，请使用库存检查功能扫描内部`192.168.0.X` 范围，查找端口`8080` 上的管理界面，然后通过该界面删除用户`carlos` 。

#### Pain POint:

- burp intruder

  根据题目：扫描 `192.168.0.X`的范围

  所以必须要意识到遍历，需要用到intruder。

  新用法：如果要遍历的payload的数字是按照顺序且数量比较多，那么可以把payload type换成numbers



#### Solved:

1. 用burp拦截查看库存的请求，发送到intruder中

2. 修改stockApi的值：

   ```
   http://192.168.0.1:8080/admin
   ```

3. 在X=1的部分 Add $,配置attack的payload，修改payload为number type

   FROM 1 TO 255 STEP 1

4. 开始attack，查看status为200的数字

5. 补齐stockApi:

   ```
   http://192.168.0.108:8080/admin/delete?username=carlos
   ```





## Circumventing common SSRF defenses

规避常见的SSRF防御

常见的情况是：应用程序中既存在SSRF行为，又包含旨在防止恶意利用的防御措施。然而，这些防御措施往往可以被绕过。



### SSRF with blacklist-based input filters

基于黑名单的输入过滤器的SSRF

某些应用程序会组织包含主机名的输入，例如 `127.0.0.1` 和 `localhost` ，或敏感网址 如 `/admin` 。在这种情况下，通常可以通过以下技巧绕过过滤器：

- 使用 `127.0.0.1` 的替代IP的表示形式：

  `2130706433` `017700000001` 或 `127.1`。

- 注册你自己的域名，使其解析至 `127.0.0.1` 。

  你可以使用 `spoofed.burpcollaborator.net` 来实现此目的。

- 使用URL编码或大小写变换来混淆被屏蔽的字符串。

- 提供一个你控制的URL,该URL会重定向至目标URL。尝试使用不同的重定向代码，以及为目标URL采用不同的协议。

  例如，在重定过程中将 `http:` 切换为 `https:` 的URL，已被证实可以绕过某些反SSRF过滤器。



------

#### LAB：SSRF with blacklist-based input filter

#### Question:

这个实验室有库存检查功能，可以从内部系统获取数据。

要解决实验室问题，可以更改库存检查URL以访问管理员界面，`http://localhost/admin`并删除用户`carlos`。

开发者部署了两个薄弱的SSRF防御措施，您需要绕过这些防御。

#### Pain POint:

- 尝试不同的可能性，猜测出是什么被过滤了？
  首先输入正常的URL：

  ```
  http://127.0.0.1
  ```

  发现不通，所以关于这个localhost的解决方式：可以简写成：http://127.1

  尝试该方式，发现可以通，再次尝试：

  ```
  http://127.1/admin
  ```

  发现又不通，所以该给admin编码！

  把a写成 %2561





------

### SSRF with whitelist-based input filters

基于白名单的输入过滤器的SSRF



有些应用只允许匹配的输入，即只允许值得白名单。

过滤器可能在输入开始得时候查找匹配。你可能可以利用URL解析中得不一致来绕过这个过滤器。

##### 先理解URL的标准结构：

根据RFC，常见的URL结构大致是：

```
scheme://[userinfo@]host[:port][?query][#fragment]
```

- `userinfo` （用户名/密码，写在@前面 ）
- `host` 域名或IP，真正决定请求去哪
- `path`,`query` 请求资源
- `fragment` 这是客户端/浏览器端的片段标识，通常不会被送到HTTP服务器。

很多绕过技术利用的，正是校验器（验证输入）与实际发出请求的HTTP库/客户端对上面这些部分的解析和处理不一致。



- URL规范包含若干功能，若使用该方法实现临时解析和验证得时候，这些功能可能被忽略：
  你可以在主机名之前得URL里嵌入凭证，使用角色。

  例如：

  ```
  https://expected-host:fakepassword@evil-host
  ```

- 你可以用该 `#` 字符来表示URL片段。例如:

  ```
  https://evil-host#expected-host
  ```

- 你可以利用DNS命名层级，将所需输入到你控制的完全限定的DNS名称中。例如：

  ```
  http://expected-host.evil-host
  ```

- 你可以用URL编码字符来混淆URL解析代码。如果实现有过滤器的代码处理URL编码字符的方式与执行后端HTTP请求的代码不同，这一点尤为有用。你也可以尝试双编码字符；有些服务器递归的对接收到的输入进行URL编码，这可能导致进一步的差异。

- 你可以将以上这些技巧组合使用。

 

------

#### LAB：SSRF with whitelist-based input filter

EXPERT级别

#### Question:







------

### Bypassing SSRF filters via open redirection

绕过SSRF过滤器通过公开重定向



有时候可以通过利用公开重定向漏洞来绕过基于过滤器的防御。

在之前的例子中，假设用户提交的URL经过严格的验证，以防止任意利用SSRF行为。然而，允许的URL所在程序存在一个公开的重定向漏洞。只要用于后端HTTP请求的API支持重定向，就可以构造一个满足过滤条件的URL，导致重定向请求到所需的后端目标。

##### 什么是重定向(redirect)？

redirect就是=服务器告诉浏览器：不要来我这个地址了，去另一个地址访问。

例如：用户访问：

```
https://exaple.com/login
```

服务器就会返回一个 302 redirect响应头：

```
location:https://example.com/home 
```

于是浏览器自动访问：

```
https://example.com/home
```

##### 什么是Open Redirect?

当用户提供的参数决定——要重定向去哪——并且服务器没有严格检查，就会出现安全问题：

```
https://safe-site.com/redirect?url=https://evil.com
```

绕过服务器直接执行：

```
Location: https://evil.com
```

那就意味着：攻击者可以利用safe-site.com把用户引到恶意站点。

##### 为什么说：Open redirect可以被用来绕过SSRF过滤？

假设网站有一个SSRF保护机制：
只允许访问https://safe-site.com/开头的URL

则用户就不能写：

```
stockApi=http://localhost/admin      ⛔ 被过滤
stockApi=http://127.0.0.1            ⛔ 被过滤
stockApi=http://evil.com             ⛔ 被过滤
```

但是如果safe-site.com里存在公开重定向功能：

```
https://safe-site.com/redirect?url=http://localhost/admin
```

虽然用户输入的URL通过了白名单检查（因为它确实是safe-site.com）

但访问后服务器内部收到的响应是：

```
302 redirect
location：http://localhost/admin
```



此类SSRF漏洞利用原理在于：

应用程序首先验证提供的 `stockAPI` 网址是否属于允许的域名（该网址确实属于允许范围）。随后应用程序请求该网址，从而触发开放重定向。它会遵循重定向路径，向攻击者指定的内部网址发起请求。



------

#### LAB：SSRF with filter bypass via open redirection vulnerability

#### Question:

这个实验室有库存检查功能，可以从内部系统获取数据。

要解决实验室问题，可以更改库存检查URL以访问管理员界面，`http://192.168.0.12:8080/admin`并删除用户`carlos`。

库存检查程序已被限制仅访问本地应用程序，因此您需要先找到影响该应用程序的开放重定向漏洞。



#### Pain POint:

- 关于stockApi参数：

  ```
  stockApi=/product/stock/check?productId=2&storeId=1
  ```

  这会告诉你：stock checker只请求本地应用的/product/路径——不能直接请求内网IP。

- 如何确定这里有重定向端点：

  点击 “next product”,同时在burp proxy中观察相应的请求/响应。

  可以看到响应头中包含：

  ```
  HTTP/1/1 302 Found
  Location:/product/stock/check?productId=3
  ```

  这就意味着path参数被原封不动的放进location——这就是典型的开发重定向。



#### Solved:

1. 拦截请求，发送给burp repeater。

2. 尝试修改`stockApi` 的参数，会发现无法让服务器将请求直接发送到其他主机。

3. 点击 `next product` ,观察到`path` 被原封不动的放如location的表头中，导致出现open redirect。

4. 创建一个利用open redirect 漏洞的URL，将其重定向至管理界面：

5. ```
   /product/nextProduct?path=http://192.168.0.12:8080/admin
   ```

6. 注意，成功之后，就可以修改路径以删除目标用户：

   ```
   /product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos
   ```



------



## Blind SSRF vulnerabilities

若能诱使应用程序向指定URL发起后端HTTP请求，但该后端请求的响应未包含在应用程序前端响应中，则会形成盲SSRF漏洞。

盲SSRF攻击更难利用，但有时候会导致在服务器或其他后端组件上实现完整的远程代码运行。



### What is blind SSRF?

盲SSRF漏洞发生于应用程序被诱导向指定URL发出后端HTTP请求时，但该后端请求的响应内容并未包含在应用程序的前端响应中。

##### 与普通SSRF的区别：

- 普通SSRF：应用程序将后端请求的响应体或状态直接包含/返回前端（或在错误信息中泄露），攻击者可以直接看到内部资源返回的内容（比如内部服务的HTML,JSON,敏感信息），从而进行信息泄露，数据获取或进一步利用。
- 盲SSRF：应用程序不把后端响应给攻击者，攻击者“看不见”直接响应。攻击者需要通过监听外部服务（例如，攻击者控制的服务器，DNS日志或者其他可观测渠道）或通过观察应用的其他行为（比如延时，资源使用，日志）来判断是否成功或达成攻击目的。

##### 盲SSRF常见的可观测信道：

- 网络连接痕迹：攻击者在其可控主机上观察来自目标服务器的TCP/HTTP请求（例如，某个域名对IP的连接）
- DNS请求日志：当目标解析攻击者控制的域名（或刻意使用多个子域来编码信息）时，攻击者可通过DNS服务器日志获知解析事件。
- 时延/带宽副作用：后端请求导致内部服务器延迟或者资源占用，攻击者通过观察应用表现变化间断推断。
- 内部行为触发：比如请求导致某个内部认为执行，数据库记录生成，邮件发送等，而这些行为在外部可以观察（如收到邮件，日志触发的回调等）
- 端口扫描结果：盲SSRF有时候可以被用来让后端去连接内部主机的不同端口，攻击者可通过是否有连接到其可控服务来推测内部端口是否开放（这是高级滥用情形，应在合法授权下测试）。



### What is the impact of blind SSRF vulnerabilities?

由于盲SSRF漏洞具有单向特性，其危害程度低于完全可利用的SSRF漏洞。

虽然在某些情况下能实现完整的远程代码执行，但这类漏洞无法轻易被利用来从后端系统窃取敏感数据。



### How to find and exploit blind SSRF vulnerabilities?

检测盲SSRF漏洞最可靠的方法就是采用带外安全分析技术（OAST）。该方法涉及向受外部系统触发HTTP请求，并监控与该系统的网络交互。

使用OAST最简单有效的方式是采用burp collaborator。你可以通过burp collaborator生成唯一的域名，将这些域名封装在payload发送至应用程序，并监控其与这些域名的交互情况。若观察到应用程序发出了入站HTTP请求，则表明该应用存在SSRF漏洞。



在测试SSRF漏洞的时候，常见现象是观察到对指定协作方域名的DNS解析操作，但后续未出现HTTP请求。

这种情况通常源于应用程序尝试向该域名发起HTTP请求，触发了初始DNS解析，但实际HTTP请求被网络层过滤机制拦截。基础设施普遍允许外向DNS流量（因其具有多重必要性），却会阻断指向未来目标的HTTP连接，这种配置模式较为常见。





------

#### LAB：Blind SSRF with out-of-band detection

#### Question:

本网站使用分析软件，当产品页面加载时，该软件会获取Referer标头中指定的URL。

要完成该实验，请使用此功能向公共Burp协作服务器发起HTTP请求。

#### Pain POint:

- 没有特别大的pain point
- 了解burp collaborator的用法

#### Solved:

1. 拦截请求，找到referer标头
2. 修改referer参数
3. insert  collaborator payload
4. send request
5. 在burp collaborator中就可以看到触发的DNS和HTTP的交互。



------

仅识别出可触发带外HTTP请求的盲SSRF漏洞，本身并不能提供可利用路径。由于无法查看后端请求的响应，该行为无法用于探索应用服务器可访问系统的内容。但仍可利用其探测服务器本身或其他后端系统的其他漏洞。可对内部IP地址空间进行盲扫，发送专门用于检测已知漏洞的payload。若这些payload同时采用OAST，则可能在未打补丁的内部服务器上发现关键漏洞。



利用盲SSRF漏洞的另一种途径是诱使应用程序连接到攻击者控制的系统，并向其发起连接的HTTP客户端返回恶意响应。若能利用服务器HTTP实现中的严重客户端漏洞，攻击者可能在应用程序基础设施内实现远程代码执行。



------

#### LAB：Blind SSRF with Shellshock exploitation

EXPERT 级别





------



## Finding hidden attack surface for SSRF vulnerabilities

发现用于SSRF漏洞的隐藏攻击面



### Partial URLs in requests(请求中的不完整URL)

有时候，应用程序仅将主机名或URL路径的一部分放入请求参数中。提交的值随后会在服务器端被整合为完整的URL进行请求。如果该值能被轻易识别为主机名或URL路径，潜在的攻击面可能显而易见。然而，由于无法控制整个请求URL，其作为完整SSRF漏洞的可利用性可能收到限制。



### URLs within data formats(数据格式中的URL)

某些应用程序采用的格式规范允许包含URL,这些URL可能被该格式的数据解析器请求。XML数据格式便是典型案例，该格式在Web应用中被广泛用于客户端向服务器传输结构化数据。当应用程序接收XML格式数据并进行解析时，可能存在XXE注入漏洞，同时亦可能通过XXE漏洞遭受SSRF攻击。

（详情请见XXE漏洞咯）



### SSRF via the Referer header(通过Referer头部实现SSRF)

一些应用使用服务器端分析软件追踪访客。该软件通常会在请求中记录referer头，以便追踪进入的链接。分析软件通常会访问出现在Referer头部的任何第三方URL。这通常用于分析引用网站的内容，包括用于入站链接的锚文本。因此，Referer头部常常时SSRF漏洞的有效攻击面。



##### TIPS：

URL验证绕过速查表：
https://portswigger.net/web-security/ssrf/url-validation-bypass-cheat-sheet

