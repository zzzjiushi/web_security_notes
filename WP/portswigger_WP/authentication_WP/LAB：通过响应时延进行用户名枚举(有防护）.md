# LAB：通过响应时延进行用户名枚举（有防护）

本题还是一个登录表单，发送到repeater中，多send几次，就会发现，IP被封禁了：

![image-20260223011319841](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260223011319841.png)

所以：

遇到IP被封禁的情况的解决方式就是：

**伪造请求头**

```
X-Forworded-For:1.1.1.1
```

我们就可以发现，又可以尝试密码了，但是一个IP不够，所以我们通过burp intruder设置两个payload

> ##### 请求头被封禁时：
>
> 尝试：
>
> ```
> X-Real-IP
> Client-IP
> True-Client-IP
> Forwarded
> ```

用Pitchfork模式：

给请求头设置一个payload，选择number列表，给用户名设置一个payload

开始攻击，因为我们要看的是这个response所需要的时间，以此来判断用户名的对错，所以我们点击对话框的三个点，勾选上 response completed和response received

其中真的用户名的response会比其他要长。

> 可以在repeater中尝试一下，输入错误的用户名，他的response时间为200多
>
> 但是输入正确的wiener用户名，就是1500多

![image-20260223015338405](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260223015338405.png)

通过爆破，发现puppet这个用户名的response明显比较长。

下一步就是 **爆破密码**

把另一个payload设置成密码，方式同上：

![image-20260223015525038](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260223015525038.png)

我们发现superman的code为302，成功找到密码。