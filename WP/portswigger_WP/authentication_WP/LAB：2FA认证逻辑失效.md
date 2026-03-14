# LAB：2FA认证逻辑失效

来源：portswigger

漏洞描述：双因素认证的逻辑漏洞在于，第一次验证和第二次验证时的用户可能不是同一个，如果网站不验证执行第二步认证的用户是否与初始用户一致，就会存在逻辑漏洞。

漏洞利用：利用这个漏洞我们就可以登录任一用户的账号。

题目描述：

该LAB的双因素认证逻辑缺陷存在安全漏洞，要解决此实验，请访问carlos的账户页面。

your credentials:wiener:peter

victim's username:carlos



解决方案：

首先输入自己的账号和密码，进入login2的页面，用burp抓包请求：

由于我们需要先生成carlos的OTP（一次性密码），所以抓包login2的GET包，发送到repeater中send,生成OTP。

然后再退出我们本身的账号重新登录（需要login2的验证POST请求包）

![image-20260313131427651](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260313131427651.png)

得到之后发送到intruder中对验证码进行爆破，找到302响应的包，获得验证码。

再把此302的包发送：show response in browser.

这样就完成该实验了。





