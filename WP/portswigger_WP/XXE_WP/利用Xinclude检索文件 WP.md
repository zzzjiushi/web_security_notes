# 利用Xinclude检索文件 WP

来源：portswigger

题目描述：

该LAB具备库存检查的功能，该功能将用户输入嵌入服务器的XML文档中，随后该文档会被解析。

由于你无法控制整个XML文档，因此无法通过定义DTD来发起经典的XXE攻击

为解决该实验，请插入一个Xinclude语句来获取文件 `/etc/passwd`的内容

##### hint：

默认情况下，`XInclude`会尝试将包含的文档解析为XML。由于`/etc/passwd`不符合XML规范，您需要在指令`XInclude`中添加额外属性来更改此行为。

https://portswigger.net/academy/labs/launch/68bb9b0d753781beeb1bd78ec3be309bed3de2f8b62b6336db6a92593bc5daaa?referrer=%2fweb-security%2fxxe%2flab-xinclude-attack



## solution：

> ### 关于Xinclude：
>
> 某些应用程序接收客户端提交的数据，将其嵌入服务器端的XML文档中，然后解析该文档。例如，当客户端提交的数据被放入后端SOAP请求中，随后由后端SOAP服务进行处理时，就会发生这种情况。
>
> 在此情况下，您无法实施经典的XXE攻击，因为您无法控制整个XML文档，因此无法定义或修改DOCTYPE<element>元素。不过，您或许可以改用`XInclude<subdocument>`XInclude元素。该元素是XML规范的一部分，允许通过子文档构建XML文档。 攻击者可在XML文档的任意数据值中植入攻击XInclude代码，因此即使仅能控制单个数据项（该数据项将被写入服务器端的XML文档），此类攻击仍可实施。
>
> 要执行攻击`XInclude`，您需要引用命名`XInclude`空间并提供要包含的文件路径。例如：
>
> ```
> <foo xmlns:xi="http://www.w3.org/2001/XInclude"> <xi:include parse="text" href="file:///etc/passwd"/></foo>
> ```

![image-20251227184725404](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251227184725404.png)

![image-20251227184737994](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251227184737994.png)