# 利用XXE漏洞执行SSRF攻击  WP

来源：portswigger

题目描述：

该LAB具备库存检查功能，可解析XML输入并响应中出现的任何异常值。

LAB服务器正在（模拟的）默认网址上 `http://169.254.169.254/` 运行一个EC2元数据端点。该端点可用于检索实例相关数据，其中部分数据可能涉及敏感信息。

为完成实验，需利用XXE漏洞执行SSRF攻击，从EC2元数据端点获取服务器的IAM秘密访问密钥

https://portswigger.net/academy/labs/launch/b856c63f6e47c5473568908b886232e76c6f1d02e9ebae4fd2ffe9d26adb1a56?referrer=%2fweb-security%2fxxe%2flab-exploiting-xxe-to-perform-ssrf



## solution：

#### 考点：

这是一个XXE漏洞，被用来实现SSRF，最终攻击的是AWS EC2 Metadata服务，从而拿到 IAM的 secretaccesskey

也就是说：

> 你不是直接攻击AWS,而是诱导“目标服务器”去访问它本地才能访问的敏感接口。

#####  `http://169.254.169.254/` ：

这是AWS EC2 Metadata service的固定地址：

**特点：**

- 只能在EC2实例内部访问
- 外部网络访问不到
- 用来给云主机提供：
  - 实例信息
  - IAM临时凭证

#### IAM：

**IAM Role** 是 AWS 给 EC2 分配权限的方式：

- 不需要把密钥写在代码里
- EC2 通过 Metadata Service **临时获取密钥**

这些密钥包括：

- `AccessKeyId`
- `SecretAccessKey`
- `Token`

本 lab 要你拿的是：
 👉 **SecretAccessKey**

> #### 利用XXE漏洞实施SSRF攻击：
>
> 这个潜在的漏洞可以诱使服务器端应用程序像服务器可访问的任意URL发起HTTP请求。
>
> 要利用XXE执行SSRF攻击，需使用目标URL定义外部XML实体，并在数据值中使用该实体。若能在应用程序响应中返回数据值内使用该实体，即可通过响应内容查看URL的响应结果，从而实现与后端系统的双向交互。否则仅能执行盲SSRF攻击。
>
> 在下面的XXE示例中，外部实体将导致服务器向组织基础设施内的内部系统发起后端HTTP请求：
>
> ```
> <!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://internal.vulnerable-website.com/"> ]>
> ```

当我们按照这个payload来构造XML的时候：

```
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://169.254.169.254/"> ]>
```

我们能看到的响应是：

```
Invalid product ID: latest
```

##### 这代表什么？

- EC2 Metadata API是一个目录型API
- 访问路径返回的是“下一层路径名称”

所以我们把这个路径名称加到url中，就可以逐层进入：



![image-20251227175521295](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251227175521295.png)

像图上的是最后一层：

![image-20251227175459189](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251227175459189.png)

就可以获得secret了。