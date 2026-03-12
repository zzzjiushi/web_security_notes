# LAB：通过细微不同的响应实现用户名枚举

该LAB是有一个登录表单，先枚举爆破用户名，看看有没有什么不一样，length，code都没有什么可疑的点。

这里学到一个点：

我们要对比的是响应，所以设置一个 grep-extract

![image-20260223002105793](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260223002105793.png)

把预期的响应，add进去，本题中就是：

`invalid username or password.`

设置好之后再进行爆破，会发现多出一行warning

点击warning就会自动排序，第一个就是可疑的响应：

![image-20260221011828077](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260221011828077.png)

![image-20260221011841169](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260221011841169.png)

本题中这个微小的变化就是，句号消失了。

所以确定用户名为：apollo

![image-20260221012053520](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260221012053520.png)

然后再枚举爆破密码，找到code响应不同的，得到密码：121212

