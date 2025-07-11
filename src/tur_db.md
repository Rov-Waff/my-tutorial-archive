# 从零教你用数据库

我的猫站洛阳铲这个项目里使用了**SQLite3**数据库来存储帖子数据，这篇文章我将会以SQLite3为例来教各位朋友如何玩一下**关系型数据库（RDB）** **本篇教程根据CC-BY 4.0协议开源**


    目 录 
    
    1 关系型数据库
    2 SQL语言
    3 安装SQLite
    4 建立你的数据库
    5 基本SQL
      5.1 增
      5.2 删
      5.3 改
      5.4 查
    6.用Python接入数据库
## 1.关系型数据库

关系型数据库组织数据的方式类似Excel表，每个数据库中包含很多张二维表，数据就存储在这些表上，具体包含关系如下

数据库管理软件(SQLite3)--**管理**-->数据库--**管理**-->二维数据表--**存储**--->数据

注意SQLite只是数据库管理软件而非数据库，常见的数据库管理软件有MySQL,PostgreSQL,MariaDB,Oracle，但这些都是大型的数据库管理软件，而SQLite是轻量级数据库管理软件，这些软件能干的SQLite都能干，只是性能不如这些大型软件

数据库的类型有很多，最常见的就是RDB，因为别的数据库能干的它都能干，原因？稍微想想什么东西是一张Excel表存不下的

> [!NOTE]
>
> 微信就是用SQLite3存你的聊天信息的

## 2.SQL语言

**SQL语言**即结构化查询语言，这是我们操作数据库使用的语言，它的语法和自然语言很像，以下是一个SQL脚本的例子

```sql
CREATE DATABASE test;
USE test;
CREATE TABLE test(id INT PRIMARY KEY AUTO_INCREMENT,flag VARCHAR(255));
INSERT INTO test VALUE(DEFAULT,"114514");
INSERT INTO test VALUE(DEFAULT,"1919810");
DELETE FROM test WHERE id=1;
UPDATE test SET flag="114514" WHERE id=2;
SELECT * FROM test;
```

每条SQL语句使用分号来分隔，通常把SQL语言关键字大写，但这不是硬性要求

在以上的SQL脚本中，体现了SQL语言的四种基本操作：**增删改查(CRUD)**

## 3.安装SQLite

对于Windows用户，请访问[SQLite官网](https://www.sqlite.org/download.html),下载[sqlite-tools-win-x64-3500200.zip](https://www.sqlite.org/2025/sqlite-tools-win-x64-3500200.zip)，别下载源代码，你肯定不想自己编译，对吧

对于Linux用户，请直接使用你的包管理器下载

```shell
#Debian系
$ sudo apt install sqlite3
#RHEL系
$ sudo dnf install sqlite3
```

## 4.建立你的数据库

Windows用户请解压并打开`sqlite3.exe`，对于Linux用户，打开你的终端，然后输入`sqlite3`，进入SQLite的命令行工具

在这个命令行工具中，你可以输入SQL语句，也可以输入以`.`开头的SQLite3指令。你可以输入`.open <数据库名>`来打开一个SQLite数据库，通常以.db作为数据库的后缀名，如果没有该数据库，那么就会创建这个数据库。示例:`.open data.db `随后您输入的所有SQL语句都会作用在该数据库中

## 5.基本SQL

### 5.1增

您可以使用`CREATE TABLE <表名> (<列名称> <列数据类型>,...);`来创建一张表，并定义这张表的每个列，示例:

```SQL
CREATE TABLE test (id INT,foo TEXT);
```

#### 5.1.1数据类型

SQL里有很多数据类型，常见的有以下这些

##### 数

| 数据类型 | 解释         |取值范围|
| -------- | ------------ |---|
| INT      | 普通整数类型 |-约21亿到正21亿|
|TINYINT|小整数|-128～127|
|SMALLINT|较小整数|-32768~32767|
|BIGINT|大整数|-9.2*10^18~9.2*10^18|
|FLOAT|单精度浮点数|7位数|
|DOUBLE|双精度浮点数|15位数|

##### 字符

| 数据类型            | 解释             |
| ------------------- | ---------------- |
| CHAR(<长度>)        | 不可变长度字符串 |
| VARCHAR(<最大长度>) | 可变长度字符串   |
| TEXT                | 超大字符串       |

##### 特殊

| 类型    | 解释               |
| ------- | ------------------ |
| BOOLEAN | 布尔值，True/False |
| NULL    | 空值               |
| BLOB    | 二进制数据         |

|      |
| ---- |

> [!IMPORTANT]
>
> SQLite3里所有的整数类型都是INT，所有浮点类型都是REAL，所有字符类型都是TEXT，BOOLEAN也是整数，SQLite里只有NULL,INT,REAL,TEXT,BLOB五种类型，以上列举的是标准SQL里的几种数据类型，但SQLite具有类型亲和性，你在SQLite里写这些数据类型也是不会报错的，但是这些类型会被强制转换为NULL,INT,REAL,TEXT,BLOB这五种类型中对应的类型

#### 5.1.2列属性

在列的数据类型后面，还可以跟上列属性，常见的列属性有这几种

| 属性          | 解释                             |
| ------------- | -------------------------------- |
| NOT NULL      | 非空                             |
| PRIMARY KEY   | 主键，具有唯一性和非空性         |
| AUTOINCREMENT | 自增，只能作用于整数[SQLite特有] |
| UNIQUE        | 唯一值                           |
| COMMENT       | 列解释                           |

示例：

```sql
CREATE TABLE foo(id INT PRIMARY KEY AUTOINCREMENT,
 name TEXT NOT NULL,
 email TEXT UNIQUE COMMENT "E-Mail");
```

#### 5.1.3增加行

你可以使用`INSERT`语句来增加一行数据，语法为`INSERT INTO <表名称> (<列名称>,...) VALUES (<数据1>，<数据2>)`示例

```sql
INSERT INTO test (id,flag) VALUES(3,"10086");
```

列名和VALUES后面的数据严格对照，如果你的**VALUES**后面的数据和这张表的列严格对照，那么列名可以省略。如果你插入的列具有自增属性，你可以用**DEFAULT**来自动生成这一列的数据，你也可以用括号包裹多行数据，数据之间用逗号隔开，示例:

```sql
INSERT INTO test (id,flag) VALUES(3,"10086"),(DEFAULT,"10010");
```

### 5.2删

#### 5.2.1删库跑路

使用`DROP DATABASE <数据库名>;`来删除数据库，**此操作不可逆**，删完别问我怎么跑路！示例:

```sql
DROP DATABASE 生死簿;
```

<img src="./assets/drop database.jpeg" style="zoom:33%;" /><------删完之后的阎王

> [!TIP]
>
> SQLite没有这个语句，这是标准SQL中的，要删库请先`.quit`退出SQLite命令行工具，然后把你的数据库文件给删掉，同样不教你怎么跑路，因为我也不会

#### 5.2.2删表

使用`DROP TABLE <表名>;`来删表，此操作不可逆，示例：

```sql
DROP TABLE 小区停车名单;
```

小区保安：阿米诺斯！

#### 5.2.3删行

使用`DELETE FROM <表名> WHERE <条件>；`来删除表格里的数据，不写条件就是删完，示例

```sql
DELETE FROM test WHERE id=1 AND flag="114514";
```

> [!CAUTION]
>
> SQL语句中赋值和比较都是等号

#### 5.2.4条件

##### 比较运算符

| 运算符       | 描述     | 示例                         |
| :----------- | :------- | :--------------------------- |
| `=`          | 等于     | `WHERE age = 25`             |
| `<>` 或 `!=` | 不等于   | `WHERE status <> 'inactive'` |
| `>`          | 大于     | `WHERE price > 100`          |
| `<`          | 小于     | `WHERE quantity < 10`        |
| `>=`         | 大于等于 | `WHERE score >= 60`          |
| `<=`         | 小于等于 | `WHERE date <= '2023-12-31'` |

##### 逻辑运算符

| 运算符 | 描述   | 示例                                    |
| :----- | :----- | :-------------------------------------- |
| `AND`  | 逻辑与 | `WHERE age > 18 AND country = 'US'`     |
| `OR`   | 逻辑或 | `WHERE status = 'active' OR is_vip = 1` |
| `NOT`  | 逻辑非 | `WHERE NOT deleted`                     |

> [!CAUTION]
>
> 不要直接拼接SQL语句，因为用户输入有可能会是 OR 1=1 --
>
> 这样SQL语句就是`DELETE FROM test WHERE id=1 OR 1=1 --  AND password="123456";`
>
> 你 表 没 了

### 5.3改

#### 5.3.1改一行的数据

使用`UPDATE <表名> SET <列名>=<新的数据> WHERE 条件;`

>[!TIP]
>
>先写SET再写条件，因为英语的语序就是这样的，不写条件就是改整张表

示例：

```sql
UPDATE test SET flag="95110" WHERE id=3;
```

#### 5.3.2改表结构

使用`ALTER <表名>`语句改表结构，这个语句比较复杂

##### 加列

用`ALTER TABLE <表名> ADD COLUMM <列名> <数据类型> <属性> 来朝表里加一个列`

示例:

```sql
ALTER TABLE test ADD COLUMN man TEXT UNIQUE;
```

##### 删列

用`ALTER TABLE <表名> DROP COLUMN <列名>`删一个列

> [!CAUTION]
>
> SQLite3不支持这个语句，这是标准SQL的语法

##### 改列

用`ALTER TABLE <表名> RENAME COLUMN <先前名> TO <修改名>;` 重命名列

### 5.4查

分为查数据库，查表，查行

#### 查数据库

使用`SHOW DATABASES;`显示该数据库管理系统所管理的所有数据库

> [!CAUTION]
>
> SQLite3没这个语句，你懂

#### 查表

使用`SHOW TABLES;`显示这个数据库里所有的表

> [!CAUTION]
>
> 也没有

#### 查行

使用`SELECT`语句查行，语法为`SELECT <列1>,<列2>... FROM <表名> WHERE <条件>`

示例

```sql
SELECT title,content FROM posts WHERE id>100;
```

使用*****代表所有列，例如查整张表:

```sql
SELECT * FROM posts;
```

使用`ORDER BY <列名> <排序方式>` 规定输出的顺序,`DESC`表示降序排序，`ASC`表示升序排序

## 6.让你的Python程序接入你的数据库

你可以用Python自带的`sqlite3`模块来连接并操作你的数据库

### 导入模块 & 连接数据库

```python
import sqlite3

# 连接数据库（不存在则自动创建）
conn = sqlite3.connect('mydatabase.db')  # 数据库文件名为 mydatabase.db
cursor = conn.cursor()  # 创建游标对象
```

### 执行SQL语句

使用`cursor.execute(<SQL语句>)`来执行一条SQL语句,示例：

```python
cursor.execute("SELECT * FROM flags;")
```

对于**增、删、改**的语句，需要使用`conn.commit()`来提交你对表的变更，示例：

```python
cursor.execute('INSERT INTO flag VALUES(4,"114514");')
conn.commit()
```

对于查询操作的语句，使用`cursor.fetchall()`获取查询到的东西，示例：

```python
cursor.execute("SELECT id,flag FROM flags;")
print(cursor.fetchall())
```

输出结果：

```
[(1,"114514"),(2,"1919810")]
```

该方法返回了一个列表，列表内是一个个元组，每个元组是你查询到的一行内容，元组内数据的顺序和你查表时使用的SQL语句中规定的顺序严格对照

### 拼接SQL语句

`?`符号是sqlite3模块的SQL语句占位符号，`cursor.execute()`方法接受两个参数，第一个为SQL语句，第二个是一个元组用来填充第一个参数中的占位符，示例:

```python
cursor.execute("SELECT id,flag FROM flag WHERE id=?;",(1,))
print(cursor.fetchall())

```

输出结果:

```
[(1,"114514")]
```

**永远不要直接使用拼接字符串的方式来拼接SQL语句！这非常危险**，SQLite3会自动对填充占位符的字符串中的符号用转义字符表达，来防止SQL注入，而你自己拼接的字符串不会这样处理。例如:

```python
sql="1 OR 1=1 --"
cursor.execute('SELECT * FROM flag WHERE id=? AND flag="114514"',sql) #处理后的SQL：SELECT * FROM flag WHERE id=1 OR 1 \= 1 \- \- 这些符号就被转义字符转义了，从而防止了SQL注入的发生
cursor.fetchall()
cursor.execute('SELECT * FROM flag WHERE id={} AND flag="114514"'.format(sql)) #拼接后的SQL:SELECT * FROM flag WHERE id=1 OR 1=1 --flag="114514" 后面的flag条件被注释掉了，而前面的条件永远为True，你的整张表就被查出来了
cursor.fetchall()
```

## 结语

SQL能干的事情还有很多，这只是SQL语言的一角，但对于我们平时写的程序来说足够用了





<a href="whanxiao855@gmail.com">从零教你用DB</a> © 2025 by <a href="shequ.codemao.cn">xiaole233</a> is licensed under <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a><img src="https://mirrors.creativecommons.org/presskit/icons/cc.svg" alt="" style="max-width: 1em;max-height:1em;margin-left: .2em;"><img src="https://mirrors.creativecommons.org/presskit/icons/by.svg" alt="" style="max-width: 1em;max-height:1em;margin-left: .2em;">

