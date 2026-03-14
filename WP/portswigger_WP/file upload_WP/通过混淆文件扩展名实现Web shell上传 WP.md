# 通过混淆文件扩展名实现Web shell上传 WP

来源：portswigger

题目描述：

该LAB有一个存在漏洞的图片上传功能。某些文件扩展名被列入黑名单，但可通过经典的混淆技术绕过此防御机制。

要完成该LAB，请上传一个基础的PHP网页shell,然后利用它窃取文件内容。

账户密码：`wiener:peter`

https://portswigger.net/academy/labs/launch/4ed109e9f6a1c61c6f344591bd7db083cd04f596804f6ba9670cbd6a3fdcafeb?referrer=%2fweb-security%2ffile-upload%2flab-file-upload-web-shell-upload-via-obfuscated-file-extension





### solution:

#### 漏洞原理：

> 即便是最详尽的黑名单，也可能被经典的混淆技术绕过

混淆技术：

- 提供多个扩展名。

  ```
  exploit.php.jpg
  ```

  这样的文件可能被解释为PHP文件或JPG文件

- 添加尾随字符：

  ```
  exploit.php.
  ```

  某些组件会剥离或忽略尾随空格，点等符号

- 尝试对点号，正斜杠和反斜杠使用URL编码（或双重URL编码）

  ```
  exploit%2Ephp
  ```

  若在验证文件扩展名时未解码该值，但后续在服务器端被解码，则可上传本会被拦截的恶意文件

- 在文件扩展名前添加分号或URL编码的空字节。

  ```
  exploit.asp;.jpg
  exploit.asp%00.jpg
  ```

  若验证逻辑采用PHP或JAVA等高级语言编写，而服务器却通过C/C++等低级语言函数处理文件，则可能导致文件名结尾的判定出现差异

- 尝试使用多字节 `Unicode` 字符，这些字符在 `Unicode` 转换或规范化后可能被转换成空字节和点。

- 







##### 第一步：

先上传一个PHP试试：

![image-20251223092330620](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223092330620.png)

发现返回：

```
only JPG & PNG files are allowed
```

##### 第二步：

尝试混淆文件名：

```
scri.php%00.jpg
```

根据返回成功的上传了文件

> 显示上传的文件必须是：scri.php