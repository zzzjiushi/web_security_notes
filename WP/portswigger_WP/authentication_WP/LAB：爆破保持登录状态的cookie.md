# LAB：爆破保持登录状态的cookie

来源：portswigger

漏洞描述：许多网站都有一个功能就是允许用户在关闭浏览器之后仍保持登录状态，并且该功能会生成某种令牌，存储在cookie中，只是对cookie简单加密，密码或者用户名等就被藏在其中，那么我们就可通过解密来猜出用户信息。如果网站并未把登录框的防爆破措施在cookie上实施的，那么就很容易构造cookie。



题目描述：该LAB允许用户在关闭浏览器之后仍保持登录状态，实现此功能所使用的cookie易受暴力破解攻击。

your credentials:wiener:peter

victim's username:carlos



解决方案：

首先通过自己的凭证登录，抓包保持登录状态的请求包，查看cookie值，分析其组成部分：

![image-20260313152237015](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260313152237015.png)

```
stay-logged-in=d2llbmVyOjUxZGMzMGRkYzQ3M2Q0M2E2MDExZTllYmJhNmNhNzcw
```

看起来像base64编码，给其解码获得：

```
wiener : 51dc30ddc473d43a6011e9ebba6ca770
```

该结构为：`wiener:hash`

一共35长度，这是 MD5 hash的典型长度。所以整体的结构：`username:md5(password)`

我们获得该信息之后，首先想到的是，该位置没有爆破防护，所以我们可以爆破密码获得cookie。

**由于cookie中的密码形成过程是，先进行md5编码，在加前缀carlos,在进行base64编码，所以我们通过intruder提供的payload processing来爆破。**

![image-20260313160045416](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260313160045416.png)

再设置一个 Grep-Match的值:update-email,因为只有在登录页面才会有一共button，所以增加一个值方便查看。

设置好intruder的设置之后进行爆破，获得密码。完成lab。



