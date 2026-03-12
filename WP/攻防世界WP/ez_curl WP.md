# ez_curl WP

拿掉题目获得一个js文件和环境：

![image-20251216141149136](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251216141149136.png)

主体部分就是：

```js
app.get('/flag',(req,res) => {
	if(!req.query.admin.includes('false') && req.headers.admin.includes('true')){
        res.send(flag);
    }else{
        res.send('try hard');
    }
});
```

主要的if语句，翻译过来就是：

> 想拿到flag，就必须同时满足两个条件：
>
> - URL参数中的admin,不能包含字符串 “false”
> - HTTP请求头中的admin必须包含字符串“true”

环境中所给的php代码：

<img src="C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251216141526767.png" alt="image-20251216141526767" style="zoom: 80%;" />

> 分析这段代码

#### 入口

```php
$input = file_get_contents('php://input');
```

`php://input` = **原始HTTP请求体**

凡是直接读取“原始请求体”的，100%是用户完全可控的

> | 语言  | 等价             |
> | ----- | ---------------- |
> | PHP   | php://input      |
> | Node  | req.body         |
> | Flask | request.data     |
> | Java  | getInputStream() |

从这句中可以看出：

**用户可以完全控制JSON请求体**

**那么请求体后面发生了什么？**

```php
json_decode($input)
```

这就是解析JSON

说明用户发送的是JSON：

```
Source：HTTP JSON body（完全可控）
```



#### 安全检查

```php
$headers = (array)json_decode($input)->headers;

for (...) {
    if(
        stripos($key, 'admin') > -1 &&
        stripos($value, 'true') > -1
    ){
        die('try hard');
    }
}

```

这是PHP对header做的**安全检查**：

意思就是：

如果你传的header中包含：

- admin
- true

就会**直接拦截**



#### URL拼接

```php
$params = (array)json_decode($input)->params;
$url .= http_build_query($params);
$url .= '&admin=false';
```

重点：

- 不管你传什么，PHP都会强行追加：`&admin=false` 
- `.=` 是拼接符



#### PHP发送请求

```php
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $url);
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
```

**为什么这段代码就是发送请求呢？**

关键字：

| 关键词 | 含义       |
| ------ | ---------- |
| curl   | HTTP客户端 |
| URL    | 请求目标   |
| exec   | 发请求     |

你可以控制的：

- URL参数的一部分
- 所有的HTTP header



##### 通用经验：

> 只要在后端代码中：
>
> - 构造URL
> - 使用HTTP客户端
> - URL含用户输入
>
> =SSRF候选点

##### 这是一个SSRF到Node服务



#### 本题的最终漏洞：

##### 第一个：include ≠ =

所以说只要不是“字符串false”，就可能绕过。

##### 第二个：参数类型在PHP--Node过程中发生变化

##### 用字符串的方式处理非字符串=逻辑漏洞

当在php中传入JSON：

```json
{
  "params": {
    "admin": ["1", "2"]
  }
}
```

在PHP中会成为：

```php
$params = (array)json_decode($input)->params;
```

此时 `$params['admin']` 是什么？

是一个数组，JSON里的数组，在后端也是数组

**重要函数**：

```php
http_build_query($params);
```

作用：把数组结构转换成URL参数形式：`admin[0]=1&admin[1]=2` 



##### 第三个漏洞：

PHP没有挡住header

```php
$headers = (array)json_decode($input)->headers;
```

且PHP的检查逻辑是：

```
$key = substr($headers[$i], 0, $offset);
$value = substr($headers[$i], $offset + 1);
```

PHP把每一行都当作一个独立的header

```php
stripos($key, 'admin') > -1
stripos($value, 'true') > -1
```

它只会检查：

- 单个header值
- 字符串层面的包含

那么node面对多个同名的header会怎么处理？

- 会把同名header合并

所以这就是漏洞！！！



##### 总结：

由于node的权限条件是：

| 位置          | 条件          |
| ------------- | ------------- |
| query.admin   | 不能包含false |
| headers.admin | 必须包含true  |

但是我们不能控制query，因为PHP它会在后面强制加 `admin=false`

那么headers呢？

```php
$headers = (array)json_decode($input)->headers;
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
```

是可以的，因为这里有漏洞。

那么这些headers是谁提供的？用户在JSON中提供的。

**所以我们唯一可以走的路就是绕过headers。**

#### 绕过headers的判断

对于php文件中的绕过，有两种方法。

##### 第一种

```swift
{"headers": ["xx:xx\nadmin: true"]}
```

我们可以看到`admin`和`true`字符串都在第一个冒号后面，因此可以绕过PHP代码的检测，而在NodeJS解析时，会解析得到`admin`的字段为true.

##### 第二种

```json
{"headers": ["admin: x", " true: y"]}
```

由于`admin`和`ture`出现在数组的两个元素中，因此可以绕过PHP文件的判断。在正常解析过程中，在键名中是不允许存在空格的，但NodeJS在遇到这类情况时是宽容的，会将其解析成

```json
{"admin": "x true y"}
```

即NodeJS会将分隔符直接去掉。



##### 构造body

python代码如下

```java
import json

datas = {"headers": ["xx:xx\nadmin: true"],        
    "params": {"admin": "true"}}

for i in range(1020):
    datas["params"]["x" + str(i)] = i

json1 = json.dumps(datas)
print(json1)
```

增加1000多个参数是为了，绕过header的检查。





> node是js的后：端，就是给出的文件的代码

##### 重要习惯：

> 谁返回flag，谁就是最终裁判



