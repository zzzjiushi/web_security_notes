# 基于白名单的input filter的SSRF WP

来源：portswigger

题目描述：

这个LAB有库存检查功能，可以从内部系统获取数据。

要解决该问题，可以更改库存检查URL以访问管理员界面：

```
http://localhost/admin
```

并且delete用户carlos。开发者部署了反SSRF防御机制，你需要绕过该机制。

https://portswigger.net/academy/labs/launch/7e946e5cc74a1c4df272ba01df0415b53548e0dfdb326d35f1d58e9b38dcbda4?referrer=%2fweb-security%2fssrf%2flab-ssrf-with-whitelist-filter

### solution：

#### 漏洞原理：

靶场的库存检查功能通过`stockApi`参数接收 URL 并发起请求，但开发者部署了基于主机名的白名单过滤（仅允许`stock.weliketoshop.net`等可信域名）。

SSRF 漏洞的关键在于：不同系统对 URL 的解析规则可能存在差异，可通过构造特殊 URL，让前端过滤器认为是合法域名，而后端实际请求的是内部地址（如localhost）。



##### **第一步：**

还是抓包查询库存的请求：

![image-20251222230138607](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251222230138607.png)

##### 第二步：

修改 `stockApi` 的参数，改成主机域名，查看是否拦截：
![image-20251222230345607](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251222230345607.png)（白名单拦截内部 IP），说明过滤器会提取并验证 URL 中的主机名。也就是存在漏洞。

并且返回：

```
"External stock check host must be stock.weliketoshop.net"
```

给了提示。

##### 第三步：

**利用 URL 嵌入式凭据绕过**URL 格式支持`http://[用户名]@[主机名]`的语法。

```
http://user@stock.weliketoshop.net
```

![image-20251222230742003](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251222230742003.png)

测试发现该格式被白名单接受（因主机名是`stock.weliketoshop.net`），但后端可能优先解析`用户名`部分作为实际请求的主机。

##### 第四步：

**通过双重编码#符号截断主机名**

1. `#`在 URL 中是锚点符号，会截断其后的内容，但直接使用`#`会被过滤器识别为非法字符。
2. 对`#`进行双重 URL 编码：`#` → 第一次编码`%23` → 第二次编码`%2523。`
3. 此时 URL 中的`%2523`在后端解码后会变为`#`，截断后续的`@stock.weliketoshop.net`，使实际请求的主机变为`localhost`。

```
http://localhost:80%2523@stock.weliketoshop.net
```

![image-20251222231436792](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251222231436792.png)

通过了，访问成功

##### 第五步：

构建最终payload

```
http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos
```

![image-20251222231545329](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251222231545329.png)

solved。

## **总结**

通过利用 URL 的`用户名@主机名`语法和双重编码`#`符号，成功绕过白名单对主机名的验证，将请求重定向到内部`localhost`的管理界面，实现 SSRF 攻击。核心是利用前后端 URL 解析规则的差异，构造 “前端合法、后端实际攻击” 的特殊 URL。