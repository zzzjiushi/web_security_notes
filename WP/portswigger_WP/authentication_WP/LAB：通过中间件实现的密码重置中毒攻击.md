# LAB：通过中间件实现的密码重置中毒攻击

##### 漏洞描述：

`Host`是代理服务器域名，`X-Forwarded-Host`是用户真实访问域名，`X-Forwarded-Host`优先级更高，当有中间件(Proxy/Middleware)处理Host时，Host很有可能是不能改的，否则请求直接被拒绝，所以很多`Middleware`的逻辑是验证host,但生成的URL用`X-Forwarded-Host`,所以host修改，请求拒绝，就尝试`X-Forwarded-Host`。

##### 题目描述：

该LAB存在密码重置漏洞。

##### 解决方案：

先利用自己的凭据了解重置密码的流程，记住重置密码的URL：

```
https://0a03004803baed7e8071081a006200ff.web-security-academy.net/forgot-password?temp-forgot-password-token=。。。。
```

由于本题的host是不能更改的，尝试更改之后发现请求直接被拒绝，所以很多中间件的逻辑就是验证host,但是生成的URL用 `X-Forwarded-Host`,所以我们增加一个标头，并且填上我们自己的域名，发送请求。

查看域名的日志，发现唯一的token就被传过来了：

```
forgot-password?temp-forgot-password-token=85sgaeidtxgxt6i9xt4jid406xuczaz7
```

根据唯一的token构造URL，复制到浏览器打开，重置密码。

