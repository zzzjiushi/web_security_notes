# Intranet  WP

来源：TryHackme

题目：

> 网页应用开发公司SecureSolaCoders创建了自己的内联网页面。开发者们还很年轻且经验不足，但他们向老板（Magnus）保证了网页应用的安全措施。开发者说：“别担心，马格努斯。我们已经从过去的错误中吸取了教训。不会再发生了。”然而，Magnus并不相信，因为他们之前在客户的应用中引入了许多奇怪的漏洞。
>
> Magnus雇佣你作为第三方来对他们的网页应用进行渗透测试。你能成功利用该应用实现root权限吗？



#### 扫描目标开放的端口和服务版本

因为不知道目标网址，所以我得通过工具来扫描目标开放的端口和服务版本：

```
nmap -sC -sV -p- 10.64.164.119
```

得到结果：

```
Nmap scan report for 10.64.164.119
Host is up (0.00022s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE
7/tcp    open  echo
21/tcp   open  ftp
22/tcp   open  ssh
23/tcp   open  telnet
80/tcp   open  http
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 0.32 seconds
root@ip-10-64-123-230:~# nmap -sC -sV -p- 10.64.164.119
Starting Nmap 7.80 ( https://nmap.org ) at 2025-12-19 06:42 GMT
mass_dns: warning: Unable to open /etc/resolv.conf. Try using --system-dns or specify valid servers with --dns-servers
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Stats: 0:00:29 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 83.33% done; ETC: 06:42 (0:00:05 remaining)
Nmap scan report for 10.64.164.119
Host is up (0.00022s latency).
Not shown: 65529 closed ports
PORT     STATE SERVICE    VERSION
7/tcp    open  echo
21/tcp   open  ftp        vsftpd 3.0.5
22/tcp   open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
23/tcp   open  tcpwrapped
80/tcp   open  http       Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
8080/tcp open  http-proxy Werkzeug/2.2.2 Python/3.8.10
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 NOT FOUND
|     Server: Werkzeug/2.2.2 Python/3.8.10
|     Date: Fri, 19 Dec 2025 06:42:15 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 207
|     Connection: close
|     <!doctype html>
|     <html lang=en>
|     <title>404 Not Found</title>
|     <h1>Not Found</h1>
|     <p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>
|   GetRequest: 
|     HTTP/1.1 302 FOUND
|     Server: Werkzeug/2.2.2 Python/3.8.10
|     Date: Fri, 19 Dec 2025 06:42:15 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 199
|     Location: /login
|     Connection: close
|     <!doctype html>
|     <html lang=en>
|     <title>Redirecting...</title>
|     <h1>Redirecting...</h1>
|     <p>You should be redirected automatically to the target URL: <a href="/login">/login</a>. If not, click the link.
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.2.2 Python/3.8.10
|     Date: Fri, 19 Dec 2025 06:42:15 GMT
|     Content-Type: text/html; charset=utf-8
|     Allow: OPTIONS, HEAD, GET
|     Content-Length: 0
|     Connection: close
|   RTSPRequest: 
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
|_http-server-header: Werkzeug/2.2.2 Python/3.8.10
| http-title: Site doesn't have a title (text/html; charset=utf-8).
|_Requested resource was /login
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8080-TCP:V=7.80%I=7%D=12/19%Time=6944F3C8%P=x86_64-pc-linux-gnu%r(G
SF:etRequest,18A,"HTTP/1\.1\x20302\x20FOUND\r\nServer:\x20Werkzeug/2\.2\.2
SF:\x20Python/3\.8\.10\r\nDate:\x20Fri,\x2019\x20Dec\x202025\x2006:42:15\x
SF:20GMT\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length
SF::\x20199\r\nLocation:\x20/login\r\nConnection:\x20close\r\n\r\n<!doctyp
SF:e\x20html>\n<html\x20lang=en>\n<title>Redirecting\.\.\.</title>\n<h1>Re
SF:directing\.\.\.</h1>\n<p>You\x20should\x20be\x20redirected\x20automatic
SF:ally\x20to\x20the\x20target\x20URL:\x20<a\x20href=\"/login\">/login</a>
SF:\.\x20If\x20not,\x20click\x20the\x20link\.\n")%r(HTTPOptions,C7,"HTTP/1
SF:\.1\x20200\x20OK\r\nServer:\x20Werkzeug/2\.2\.2\x20Python/3\.8\.10\r\nD
SF:ate:\x20Fri,\x2019\x20Dec\x202025\x2006:42:15\x20GMT\r\nContent-Type:\x
SF:20text/html;\x20charset=utf-8\r\nAllow:\x20OPTIONS,\x20HEAD,\x20GET\r\n
SF:Content-Length:\x200\r\nConnection:\x20close\r\n\r\n")%r(RTSPRequest,1F
SF:4,"<!DOCTYPE\x20HTML\x20PUBLIC\x20\"-//W3C//DTD\x20HTML\x204\.01//EN\"\
SF:n\x20\x20\x20\x20\x20\x20\x20\x20\"http://www\.w3\.org/TR/html4/strict\
SF:.dtd\">\n<html>\n\x20\x20\x20\x20<head>\n\x20\x20\x20\x20\x20\x20\x20\x
SF:20<meta\x20http-equiv=\"Content-Type\"\x20content=\"text/html;charset=u
SF:tf-8\">\n\x20\x20\x20\x20\x20\x20\x20\x20<title>Error\x20response</titl
SF:e>\n\x20\x20\x20\x20</head>\n\x20\x20\x20\x20<body>\n\x20\x20\x20\x20\x
SF:20\x20\x20\x20<h1>Error\x20response</h1>\n\x20\x20\x20\x20\x20\x20\x20\
SF:x20<p>Error\x20code:\x20400</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Mes
SF:sage:\x20Bad\x20request\x20version\x20\('RTSP/1\.0'\)\.</p>\n\x20\x20\x
SF:20\x20\x20\x20\x20\x20<p>Error\x20code\x20explanation:\x20HTTPStatus\.B
SF:AD_REQUEST\x20-\x20Bad\x20request\x20syntax\x20or\x20unsupported\x20met
SF:hod\.</p>\n\x20\x20\x20\x20</body>\n</html>\n")%r(FourOhFourRequest,184
SF:,"HTTP/1\.1\x20404\x20NOT\x20FOUND\r\nServer:\x20Werkzeug/2\.2\.2\x20Py
SF:thon/3\.8\.10\r\nDate:\x20Fri,\x2019\x20Dec\x202025\x2006:42:15\x20GMT\
SF:r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x202
SF:07\r\nConnection:\x20close\r\n\r\n<!doctype\x20html>\n<html\x20lang=en>
SF:\n<title>404\x20Not\x20Found</title>\n<h1>Not\x20Found</h1>\n<p>The\x20
SF:requested\x20URL\x20was\x20not\x20found\x20on\x20the\x20server\.\x20If\
SF:x20you\x20entered\x20the\x20URL\x20manually\x20please\x20check\x20your\
SF:x20spelling\x20and\x20try\x20again\.</p>\n");
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```

筛选一下有用信息：

```
PORT     STATE SERVICE
7/tcp    open  echo
21/tcp   open  ftp
22/tcp   open  ssh
23/tcp   open  telnet
80/tcp   open  http
8080/tcp open  http-proxy


href="/login">/login</a>. If not, click the link.
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.2.2 Python/3.8.10
|     Date: Fri, 19 Dec 2025 06:42:15 GMT
|     Content-Type: text/html; charset=utf-8
|     Allow: OPTIONS, HEAD, GET
|     Content-Length: 0
|     Connection: close
```

这表明一共有6个端口，但是只有8080的port是有用的，并且只有/login是200是通的。



#### 访问 `http://10.64.164.119:8080/login`

得到一个登录页面。

```
<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
body {
  font-family: Arial, Helvetica, sans-serif;
  background-color: black;
}

* {
  box-sizing: border-box;
}

/* Add padding to containers */
.container {
  padding: 16px;
  background-color: white;
}

/* Full-width input fields */
input[type=text], input[type=password] {
  width: 100%;
  padding: 15px;
  margin: 5px 0 22px 0;
  display: inline-block;
  border: none;
  background: #f1f1f1;
}

input[type=text]:focus, input[type=password]:focus {
  background-color: #ddd;
  outline: none;
}

/* Overwrite default styles of hr */
hr {
  border: 1px solid #f1f1f1;
  margin-bottom: 25px;
}

/* Set a style for the submit button */
.login_button {
  background-color: #4571d0;
  color: white;
  padding: 16px 20px;
  margin: 8px 0;
  border: none;
  cursor: pointer;
  width: 100%;
  opacity: 0.9;
}

.registerbtn:hover {
  opacity: 1;
}

/* Add a blue text color to links */
a {
  color: dodgerblue;
}

/* Set a grey background color and center the text of the "sign in" section */
.signin {
  background-color: #f1f1f1;
  text-align: center;
}

.footer {
   position: fixed;
   left: 0;
   bottom: 0;
   width: 100%;
   background-color: black;
   color: white;
   text-align: center;
}

</style>
</head>
<body>

<div class="container">
<form action="" method="POST">
    <h1>Intranet login</h1>
    <hr>
    <label for="usr"><b>E-mail</b></label>
    <input type="text" placeholder="firstname@securesolacoders.no" name="username" id="username" value="" required>

    <label for="psw"><b>Password</b></label>
    <input type="password" placeholder="Password" name="password" id="password" value="" required>
    <hr>

    <button type="submit" class="login_button"><b>Log in</b></button>
  
</form>
    
      </div>

<div class="footer">
  <p>Proudly hosted, maintained, and developed by <b>SecureSolaCoders.no ©</b></p>
</div>
</body>
</html>
<!--- Any bugs? Please report them to our developer team. We have an open bug bounty program!
  For any inquiries, contact devops@securesolacoders.no.
  Sincerely, anders (Senior Developer
```

