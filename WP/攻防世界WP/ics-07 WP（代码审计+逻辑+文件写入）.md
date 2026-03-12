# ics-07 WP（代码审计+逻辑+文件写入）

题目来源：XCTF

难度：5

题目场景：http://61.147.171.105:64191/



先扫描一下目录，查看是否存在有用信息

![image-20260123001342577](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123001342577.png)

存在flag.php,config.php,js查看一下

直接访问flag.php和config.php是打不开的，js里面就是两个js代码文件



漏洞点应该在index.php?page=flag.php上

还有index.php/login?page=flag.php上

![image-20260123001908795](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123001908795.png)

所以用burp抓包查找项目的请求，在repeater中修改测试：

![image-20260123002100626](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123002100626.png)



> 有一个细节：

页面上有一个按钮：view-source，按了没反应，这就是提示：

直接访问：/view-source.php

获得源码：

```php+HTML
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>cetc7</title>
  </head>
  <body>
    <?php
    session_start();

    if (!isset($_GET[page])) {
      show_source(__FILE__);
      die();
    }

    if (isset($_GET[page]) && $_GET[page] != 'index.php') {
      include('flag.php');
    }else {
      header('Location: ?page=flag.php');
    }

    ?>

    <form action="#" method="get">
      page : <input type="text" name="page" value="">
      id : <input type="text" name="id" value="">
      <input type="submit" name="submit" value="submit">
    </form>
    <br />
    <a href="index.phps">view-source</a>

    <?php
     if ($_SESSION['admin']) {
       $con = $_POST['con'];
       $file = $_POST['file'];
       $filename = "backup/".$file;

       if(preg_match('/.+\.ph(p[3457]?|t|tml)$/i', $filename)){
          die("Bad file extension");
       }else{
            chdir('uploaded');
           $f = fopen($filename, 'w');
           fwrite($f, $con);
           fclose($f);
       }
     }
     ?>

    <?php
      if (isset($_GET[id]) && floatval($_GET[id]) !== '1' && substr($_GET[id], -1) === '9') {
        include 'config.php';
        $id = mysql_real_escape_string($_GET[id]);
        $sql="select * from cetc007.user where id='$id'";
        $result = mysql_query($sql);
        $result = mysql_fetch_object($result);
      } else {
        $result = False;
        die();
      }

      if(!$result)die("<br >something wae wrong ! <br>");
      if($result){
        echo "id: ".$result->id."</br>";
        echo "name:".$result->user."</br>";
        $_SESSION['admin'] = True;
      }
     ?>

  </body>
</html>
```

要进行代码审计：

一共分为三段，第一段：

```php
<?php
    session_start();
//如果没有传入page参数，那么就显示show源码（FILE）表示当前文件路径
    if (!isset($_GET[page])) {
      show_source(__FILE__);
      die();
    }
//如果存在page参数且page参数不等于index.php，则无条件包含flag.php
//else,page等于index.php，服务器就会302重定向，跳转到?page=flag.php
    if (isset($_GET[page]) && $_GET[page] != 'index.php') {
      include('flag.php');
    }else {
      header('Location: ?page=flag.php');
    }

    ?>
```

我们根据审计这段代码得到的信息，去写url的参数：

![image-20260123004015019](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123004015019.png)

可以得到这样的页面



审计第二段源码：

```php
<?php
    //如果当前session中存在admin且为真
     if ($_SESSION['admin']) {
       //获取用户输入
       $con = $_POST['con'];//写入文件的内容
       $file = $_POST['file'];//文件名
       $filename = "backup/".$file;//拼接文件路径
       //uploaded/backup/{$file}路径穿越风险
		//黑名单校验
       if(preg_match('/.+\.ph(p[3457]?|t|tml)$/i', $filename)){
          die("Bad file extension");
       //.php .php3 .php4 .php5 .php7 .pht .phtml
       }else{
           //改变工作目录
            chdir('uploaded');
           //文件写入（致命点）
           //以w写的模式打开文件
           $f = fopen($filename, 'w');
           fwrite($f, $con);
           fclose($f);
       }
     }
     ?>
```

根据第二段代码的审计，可以得到，如果得到admin的权限，就可以写入任意恶意文件来获取敏感信息



第三段代码审计：

```php
<?php
    //如果有参数id且最后一个字符为9
      if (isset($_GET[id]) && floatval($_GET[id]) !== '1' && substr($_GET[id], -1) === '9') {
        include 'config.php';
          //一些转义
        $id = mysql_real_escape_string($_GET[id]);
        $sql="select * from cetc007.user where id='$id'";
        $result = mysql_query($sql);
        $result = mysql_fetch_object($result);
      } else {
        $result = False;
        die();
      }

      if(!$result)die("<br >something wae wrong ! <br>");
      if($result){
        echo "id: ".$result->id."</br>";
        echo "name:".$result->user."</br>";
        $_SESSION['admin'] = True;//权限提升！
      }
     ?>
```

所以，我们根据第三段代码的审计，就可以获得admin的权限：

page=flag.php id=1-9

![image-20260123010233876](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123010233876.png)

获得admin的权限，就可以传入恶意文件了（根据第二段代码审计）



用burp上传一直上传不上去，所以改用kali

![image-20260123122539913](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123122539913.png)

上传成功：

![image-20260123122502279](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123122502279.png)

用kali来访问shell.php利用一句话木马

![image-20260123124306011](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260123124306011.png)



> ### 一些pain point：
>
> 1.上传文件内容，一定要用POST。
>
> 2.有些不支持burp上传，就可以用kali，kali上传的格式：
>
> ```
> curl -i -b "PHPSESSID" -X POST "http网址" -d "上传内容"
> ```
>
> 3.用burp访问读取不可读的文件（利用一句话木马）
>
> ```
> curl -X POST "网址" -d "cmd=..."
> ```
>
> 4.有时候cat读取会被禁用，那么就用echo：
>
> ```
> cmd = echo file_get_contents('/var/www/html/flag.php')
> ```



> ### 如果不知道文件位置：
>
> ##### 访问web根目录：
>
> ```
> pwd
> ```
>
> 一般会返回(这是apache默认的web根目录)
>
> ```
> /var/www/html
> ```
>
> 然后再ls-la
>
> ##### 联系源码相关文件
>
> ##### 没线索时，全局搜索：
>
> ```
> find -name "*flag*" 2>/dev/null
> ```
>
> ##### 数据库：
>
> flag有时候会藏在数据库中
>
> 于是流程：
>
> ```
> cat config.php //找到DB账号密码
> mysql -u xxx -p xxx
> show databases
> show tables
> select * from flag
> ```





