# LAB：通过中间件实现的密码重置中毒攻击

##### 漏洞描述：

`Host`是代理服务器域名，`X-Forwarded-Host`是用户真实访问域名，`X-Forwarded-Host`优先级更高，当有中间件(Proxy/Middleware)处理Host时，Host很有可能是不能改的，否则请求直接被拒绝，所以很多`Middleware`的逻辑是验证host,但生成的URL用`X-Forwarded-Host`,所以host修改，请求拒绝，就尝试`X-Forwarded-Host`。

##### 题目描述：



##### 解决方案：