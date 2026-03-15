# LAB：通过修改密码进行爆破密码

##### 漏洞描述：

和之前的爆破很像，就是忘记给修改密码的页面进行爆破防护了，而且修改密码不需要受害者账号登录，所以只需要知道URL，就可以爆破目标账号的密码了。

##### 题目描述：

该LAB的密码修改功能使其容易受到爆破攻击。

##### 解决方案：

先根据自己凭证登录，修改密码，抓包：

```
POST /my-account/change-password HTTP/2

username=wiener&current-password=peter&new-password-1=1&new-password-2=2
```

我们发现如果把新密码设成不同的值，当原密码正确时会显示 `New passwords do not match`，如果密码不正确时，会显示 `Current password is incorrect`，所以我们根据这一不同来爆破密码。

用 `GREP-MATCH`这个功能就可以标注关键字，最终爆破出密码。

