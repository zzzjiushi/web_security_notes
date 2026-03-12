# 文件包含 WP

来源：攻防世界

打开目标地址之后就可以看到：

<img src="C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251219173351819.png" alt="image-20251219173351819" style="zoom:67%;" />

通过该网页中php代码就可以看出：

```
if(isset($_GET['filename'])){
        $filename  = $_GET['filename'];
        include($filename);
    }
```

典型的文件包含，输入点 `$filename  = $_GET['filename'];` 没有任何过滤，用户输入完全可控。

` include($filename);` 这是一个典型的文件包含漏洞（LFI/RFI），可以包含任意文件，取决于：check.php做了什么限制，PHP配置等。



我做的第一件事是用dirsearch扫描有没有什么隐藏路径：

![image-20251219165824931](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251219165824931.png)



访问flag.php，没有东西，所以需要去include它。



> 简单介绍一下文件包含：文件包含（File Inclusion）是一种在编程中常见的功能，它允许一个文件（通常是一个脚本或程序）读取并执行另一个文件的内容。在 Web 开发中，文件包含通常用于 PHP、JSP、ASP.NET 等服务器端脚本语言，但也可以用于任何支持文件读取和执行的编程语言。而文件包含漏洞（File Inclusion Vulnerability）允许攻击者通过操纵输入来包含服务器上的任意文件或目录内容，可能导致敏感信息泄露或任意代码执行。这种漏洞通常出现在应用程序使用 `include()` 或 `require()` 等函数加载文件时，如果攻击者能够控制这些函数的参数，那么他们就可以将任意文件的内容包含到应用程序中执行。
>
> php的文件包含漏洞，可以用到的伪协议如下：
>
> file:// 访问本地文件系统
> http:// 访问http(s）网址
> ftp:// 访问ftp
> php:// 访问各个输入/输出流
> zlib:// 压缩流
> data:// 数据
> rar:// RAR压缩包
> ogg:// 音频流
>
> 本题而言，用到的是php://伪协议，用法如下：
>
> php://input,用于执行php代码，（需要post请求提交数据）。
>
> *php://filter,用于读取源码，?filename=php://filter/read=convert.base64/resource=/etc/passwd*

尝试使用字符集，提示有了变化，于是尝试用别的字符集组合。

![img](https://img2024.cnblogs.com/blog/3642332/202505/3642332-20250502151249999-1609423748.png)

从[PHP官网](https://www.php.net/manual/zh/mbstring.supported-encodings.php)得知，PHP 扩展支持的字符编码有以下几种，* 表示该编码也可以在正则表达式中使用，开始暴力破解



[![复制代码](https://assets.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
UCS-4*
UCS-4BE
UCS-4LE*
UCS-2
UCS-2BE
UCS-2LE
UTF-32*
UTF-32BE*
UTF-32LE*
UTF-16*
UTF-16BE*
UTF-16LE*
UTF-7
UTF7-IMAP
UTF-8*
ASCII*
EUC-JP*
SJIS*
eucJP-win*
SJIS-win*
ISO-2022-JP
ISO-2022-JP-MS
CP932
CP51932
SJIS-mac
SJIS-Mobile#DOCOMO
SJIS-Mobile#KDDI
SJIS-Mobile#SOFTBANK
UTF-8-Mobile#DOCOMO
UTF-8-Mobile#KDDI-A
UTF-8-Mobile#KDDI-B
UTF-8-Mobile#SOFTBANK
ISO-2022-JP-MOBILE#KDDI
JIS
JIS-ms
CP50220
CP50220raw
CP50221
CP50222
ISO-8859-1*
ISO-8859-2*
ISO-8859-3*
ISO-8859-4*
ISO-8859-5*
ISO-8859-6*
ISO-8859-7*
ISO-8859-8*
ISO-8859-9*
ISO-8859-10*
ISO-8859-13*
ISO-8859-14*
ISO-8859-15*
ISO-8859-16*
byte2be
byte2le
byte4be
byte4le
BASE64
HTML-ENTITIES
7bit
8bit
EUC-CN*
CP936
GB18030
HZ
EUC-TW*
CP950
BIG-5*
EUC-KR*
UHC
ISO-2022-KR
Windows-1251
Windows-1252
CP866
KOI8-R*
KOI8-U*
ArmSCII-8
```

[![复制代码](https://assets.cnblogs.com/images/copycode.gif)](javascript:void(0);)

使用 Burp Suite 的 Intruder 模块，切换为集束炸弹模式，设置好 payload 位置

![img](https://img2024.cnblogs.com/blog/3642332/202505/3642332-20250502151932884-1842529408.png)

 设置 payload 的内容，点击开始攻击，总共请求近六千次，不过刚开始没多久就爆破出flag了

![img](https://img2024.cnblogs.com/blog/3642332/202505/3642332-20250502152253771-1302075999.png)

![](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251220132419102.png)

最后的payload：

```
?filename=php://filter/convert.iconv.utf-7.UCS-4%2a/resource=flag.php  
```

核心作用：

1. `php:filter` 允许你再**读取文件时做编码转换**

2. `convert.iconv.A.B`:

   A:原编码

   B:目标编码

3. 在CTF中的常见用途：

   - 绕过 `include` / `file_get_contents` 的过滤
   - 利用非法编码——触发报错/截断/内容泄露
   - 搭配**字符集爆破**找服务器允许的编码组合



##### 在intruder中找到正确payload的方式：

关注**响应长度**

直接点开`response` 

