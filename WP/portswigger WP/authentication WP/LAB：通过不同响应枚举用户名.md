# LAB：通过不同响应枚举用户名

有一个登录表单，我们通过intruder工具，先对username进行爆破，看一下有没有响应的差别，发现：

![image-20260221004959775](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260221004959775.png)

只有ak的响应是 incorrect password，所以推测这是正确的存在的用户。

![image-20260221005131354](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260221005131354.png)

再对密码进行爆破，同样的方式，发现该密码的状态码不同是302，重定向，也就是说已经登录成功，切换表单了。

至此得出密码。