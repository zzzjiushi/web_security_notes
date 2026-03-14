# 通过绕过扩展黑名单实现web shell上传 WP

来源：portswigger

题目描述：

该LAB包含一个存在漏洞的图片上传功能。虽然某些文件扩展名已被列入黑名单，但由于黑名单配置存在根本性缺陷，此防御机制可被绕过。

要完成实验，请上传一个基础PHP网页shell,然后利用它窃取文件内容。通过实验横幅提供的 `home/carlos/secret`按钮提交该密钥

用户密码：`wiener：peter`

https://portswigger.net/academy/labs/launch/23b377e90a4c2eb083ef9f02cb4800e763796995a1449f85714790d15c30b9fa?referrer=%2fweb-security%2ffile-upload%2flab-file-upload-web-shell-upload-via-extension-blacklist-bypass



### solution：

#### 漏洞原理：

##### 核心：

- 黑名单式扩展名校验 + 允许用户上传可影响服务器解析规则的文件(`.htaccess`)

- Apache + mod_php的解析机制被滥用

  关于 `Apache` 是怎么决定一个文件要不要执行？

  > 这个文件的MIME类型是不是 `application/x-httpd-php`

  而这个映射关系来自：

  - 主配置文件
  - `.htaccess`

  例如：

  ```
  AddType application/x-httpd-php .php
  ```

  意思就是：所有 `.php` 当PHP文件执行



##### 本质：

> 服务器只禁止了 `.php` ，却允许你上传 `.htaccess` ,而 `.htaccess`可以修改`Apache`**对哪些扩展名会当PHP执行的规则**

##### 本题的环境：

- Web Server:Apache

- PHP运行方式：mod_php

- 上传目录：`/files/avatars`

- 上传限制：

  黑名单禁止 `.php`

  没禁止：`.htaccess`





##### 思路：

1. 先上传一个 `.htaccess` ，告诉 `Apache` ，`.133t`也是PHP
2. 再上传一个 `exploit.133t` 的PHP web shell
3. 服务器看到 `.133t` 不再黑名单——放行
4. `Apache` 解析的时候按照PHP解析——RCE







##### 第一步：

我们先尝试提交恶意PHP文件：

![image-20251223000730743](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223000730743.png)

我们会发现，会报错，所以应该被黑名单限制了。



##### 第二步：

上传 `.htaccess` 修改 `Apache` 的执行规则：

```
filename=".htaccess"
Content-Type=text/plain    //避免被当成脚本
内容修改：AddType application/x-httpd-php .l33t   //修改规则
```

![image-20251223090618865](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223090618865.png)

![](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223090631030.png)



发现返回200，成功上传和修改。

##### 第三步：

上传真正的PHP文件。

但是把后缀名改成 `.133t`

![image-20251223090706417](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223090706417.png)

返回200，成功上传。



##### 第四步：

查看上传的文件，抓包一个GET包，修改其路径，指到我们上传的文件

```
GET /files/avatars/exploit.l33t
```

查看响应，获得密码

![image-20251223001500920](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223001500920.png)



#### 总结：

这是一个文件上传 + 服务器解析规则被挟持的漏洞。

黑名单禁止 `.php` 毫无意义，只要攻击者能够上传 `.htaccess` ，就能重新定义什么文件会被当成PHP文件执行，最终实现任意代码执行。

> 黑名单挡文件，白名单挡攻击，能传 `.htaccess` ，迟早能RCE

