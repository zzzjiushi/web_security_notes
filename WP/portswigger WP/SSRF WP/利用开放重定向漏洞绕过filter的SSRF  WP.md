# 利用开放重定向漏洞绕过filter的SSRF  WP

来源：portswigger

题目描述：

这个LAB有库存检查功能，可以从内部系统获取数据。

要解决实验室问题，可以更改库存检查URL以访问管理员界面：

```
http://192.168.0.12:8080/admin
```

并删除用户carlos。

库存检查程序已**被限制仅访问本地应用程序**，因此你需要找到影响该应用程序的开放重定向漏洞。

https://portswigger.net/academy/labs/launch/3bd839053db0960daf7046201a4b58936ba21cbb4a15840db8237690eb2c4be7?referrer=%2fweb-security%2fssrf%2flab-ssrf-filter-bypass-via-open-redirection



### Solution:

根据题目描述，可以知道：
`stockApi` 限制只能访问本地应用程序，也给出了解决方案：重定向

> #### 关于重定向(redirect)：
>
> 服务器告诉浏览器，不要来我这个地址，去另一个地址访问。
>
> #### 关于开放重定向(open redirect)：
>
> 用户提供的参数决定——要重定向去哪——并且服务器没有严格检查，就会出现安全问题：
>
> ```
> https://safe-site.com/redirect?url=https://evil.com
> ```
>
> 此类SSRF漏洞利用原理在于：
>
> 应用程序首先验证提供的stockApi网址是否属于允许的域名（该网址确实属于允许范围）。随后应用程序请求该网址，从而触发开放重定向。它会遵循重定向路径，像攻击者指定的内部网址发起请求。
>
> 

还是抓包库存查询的请求：

![image-20251221203747912](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251221203747912.png)

尝试修改stockApi为内网管理地址：

```
http://192.168.0.12:8080/admin
```

![image-20251221205111269](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251221205111269.png)

因为stockApi已经被限制了，所以寻找其他的地方（开放重定向漏洞）

我们发现页面右下角有一个 `next product` 按钮，点击抓取请求：

![image-20251221205327538](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251221205327538.png)观察其请求URL，path参数指定挑战目标。

修改path参数为任意地址（`http://example.com`），发送后发现响应头中出现：

```
Location:http://example.com
```

证明存在开放重定向（path参数的值被直接作为重定向目标，未作限制）

所以我们尝试把我们的内网管理地址放进去：

```
GET /product/nextProduct?currentProductId=1&path=http://192.168.0.12:8080/admin 
```

发现：

![image-20251221210832545](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251221210832545.png)

所以成功了。

那么我就可以返回到stockApi中，利用可以漏洞来访问目标地址：

![image-20251221210918368](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251221210918368.png)

该地址的网址是本网站内部的网址（nextProduct），所以可以通过验证，然后当其访问的时候，就会被重定向至新的目标网址中，成功攻击！