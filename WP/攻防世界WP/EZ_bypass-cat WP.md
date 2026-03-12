# EZ_bypass-cat WP

打开环境之后发现是一个登录框，需要用户名和密码

用burp抓包，查看请求：

发现！我们随便试的密码！被加密了

所以说，应该就不是SQL注入。

然后学习到新技能：

在burp的query中有目录，有很多不知道的路径。

但是也有命令行的办法：（更好一点）

```
feroxbuster -u http://61.147.171.105:51390/login.html -k -n
```

根据所列出的目录，我们就来找一下可疑的路径

其中有一个路径是 `syslogin.js` ，我们打开该文件

<img src="C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251216191817530.png" alt="image-20251216191817530" style="zoom:67%;" />

该文件的开头注释，提示说是

```
copyright(c) 华夏erp
```

所以咱们上网搜索华夏erp,就可以找到类似的解：

![image-20251216192052149](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251216192052149.png)

根据这个payload，修改burp中请求，在repeater中发送，就可以获得用户名和密码：

![image-20251216173849346](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251216173849346.png)



根据用户名和密码来登录，刷新页面，再次用burp抓包，复制cookie

在dirsearch中遍历：

```
dirsearch -u http://61.147.171.105:51390/ --cookie="JSESSIONID=307EA14D4BFBC1BCAEF328C9D06632E8"
```

<img src="C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251216174642838.png" alt="image-20251216174642838" style="zoom: 67%;" />

看到可疑文件：`flag.html`

就可以获得flag了。