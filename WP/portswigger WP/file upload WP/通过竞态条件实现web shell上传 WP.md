# 通过竞态条件实现web shell上传 WP

来源：portswigger

题目描述：

该LAB包含一个存在漏洞的图片上传功能。尽管该功能对能上传的文件都执行了严格的验证，但通过利用其处理文件时的竞争条件，完全有可能绕过这些验证机制。

要完成该实验，请上传一个基础的PHP网页shell。

用户密码：`wiener:peter`

https://portswigger.net/academy/labs/launch/e7c14c1093f823154152c9ce425959ef67d62c9bb7c65eb02f6a8a597ab993e8?referrer=%2fweb-security%2ffile-upload%2flab-file-upload-web-shell-upload-via-race-condition





### solution：







![image-20251223142903884](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223142903884.png)

![image-20251223142838481](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223142838481.png)