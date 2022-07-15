# mysql学习笔记

<!-- TOC -->

- [mysql学习笔记](#mysql学习笔记)
  - [1. mysql基础操作](#1-mysql基础操作)
    - [1.1. 数据库相关](#11-数据库相关)
      - [1.1.1. 连接与断开数据库](#111-连接与断开数据库)
      - [1.1.2. 查看有哪些数据库](#112-查看有哪些数据库)
      - [1.1.3. 创建数据库](#113-创建数据库)
      - [1.1.4. 选择数据库](#114-选择数据库)
    - [1.2. 表相关](#12-表相关)
      - [1.2.1. 数据表的字段类型](#121-数据表的字段类型)
        - [1.2.1.1. 数值类型](#1211-数值类型)
        - [1.2.1.2. 日期和时间类型](#1212-日期和时间类型)
        - [1.2.1.3. 字符串类型](#1213-字符串类型)
      - [1.2.2. 数据表的字段属性](#122-数据表的字段属性)
        - [1.2.2.1. 主键](#1221-主键)
        - [1.2.2.2. 外键](#1222-外键)
        - [1.2.2.3. 自动递增](#1223-自动递增)
        - [1.2.2.4. 非空](#1224-非空)
        - [1.2.2.5. 唯一性约束](#1225-唯一性约束)
        - [1.2.2.6. 索引](#1226-索引)
      - [1.2.3. 创建数据表](#123-创建数据表)
      - [1.2.4. 查看数据表](#124-查看数据表)
      - [1.2.5. 删除数据表](#125-删除数据表)
  - [2. sql函数](#2-sql函数)
    - [2.1. replace函数](#21-replace函数)
    - [2.2. current_timestamp函数](#22-current_timestamp函数)
    - [2.3. unix_timestamp函数](#23-unix_timestamp函数)
    - [2.4. from_unixtime函数](#24-from_unixtime函数)
  - [3. mysql和redis之间的同步方案](#3-mysql和redis之间的同步方案)

<!-- /TOC -->

## 1. mysql基础操作

### 1.1. 数据库相关

#### 1.1.1. 连接与断开数据库

**连接本地数据库：**

    // 键入命令，回车然后输入密码即可
    mysql -u 用户名 -p

**连接远程数据库：**

    mysql -h mysql服务器地址 -P 端口号 -u 用户名 -p

    // 若指明服务器地址为 127.0.0.1 或 localhost，则会连接本地数据库
    mysql -h localhost -u 用户名 -p

> 注：<font color=Crimson>-p (小写的) 后输入密码（紧挨着 -p ，中间不能用空格分隔），会直接登录（密码会明文显示，不安全）；不输入密码会要求输入密码（密码不显示，较安全）</font>

**断开连接：**

    exit 或 \q

#### 1.1.2. 查看有哪些数据库

命令如下：（注意是 databases 而不是 database。）

    show databases;

**配合 like 命令使用，可以查找指定名称的数据库：**

    // 列出名称为 'test' 的数据库
    show databases like 'test';

    // 列出名称中包含 'test' 的数据库
    show databases like '%test%';
    
    // 列出名称中以 'test' 开头的数据库
    show databases like 'test%';

    // 列出名称中以 'test' 结尾的数据库
    show databases like '%test';

#### 1.1.3. 创建数据库

语法如下：

    CREATE DATABASE 数据库名;

#### 1.1.4. 选择数据库

语法如下：

    use 数据库名;

### 1.2. 表相关

#### 1.2.1. 数据表的字段类型

##### 1.2.1.1. 数值类型

| 类型 | 大小 | 范围（有符号） | 范围（无符号）| 用途 |
| :--- | :--- | :--- | :--- | :--- |
| TINYINT | 1 Byte | (-128, 127) | (0, 255) | 小整数值 |
| SMALLINT | 2 Byte | (-32768, 32767) | (0, 65535) | 大整数值 |
| MEDIUMINT | 3 Byte | (-8 388 608，8 388 607) | (0，16 777 215) | 大整数值 |
| INT 或 INTEGER | 4 Byte | (-2^31, 2^31 - 1) | (0, 2^32) | 大整数值 |
| BIGINT | 8 Byte | (-2^63, 2^63 - 1) | (0, 2^64) | 极大整数值 |
| FLOAT | 4 Byte | (-3.402 823 466 E+38，-1.175 494 351 E-38)，0，(1.175 494 351 E-38，3.402 823 466 351 E+38) | 0，(1.175 494 351 E-38，3.402 823 466 E+38) | 单精度浮点值 |
| DOUBLE | 8 Byte | (-1.797 693 134 862 315 7 E+308，-2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 双精度浮点值 |
| DECIMAL | 对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2 | 依赖于 M 和 D 的值 | 依赖于 M 和 D 的值 | 小数值 |

##### 1.2.1.2. 日期和时间类型

| 类型 | 大小 | 范围 | 格式 | 用途 |
| :--- | :--- | :--- | :--- | :--- |
| DATE | 3 Byte | 1000-01-01/9999-12-31 | YYYY-MM-DD | 日期值 |
| TIME | 3 Byte | '-838:59:59'/'838:59:59' | "HH:MM:SS" | 时间值或持续时间 |
| YEAR | 1 Byte | 1901/2155 | YYYY | 年份值 |
| DATETIME | 8 Byte | 1000-01-01 00:00:00/9999-12-31 23:59:59 | YYYY-MM-DD HH:MM:SS | 混合日期和时间值 |
| TIMESTAMP | 4 Byte | 1970-01-01 00:00:00/2038 <br> 结束时间是第 2147483647 秒，北京时间 2038-1-19 11:14:07，格林尼治时间 2038年1月19日 凌晨 03:14:07 | YYYYMMDDHHmmSS | 混合日期和时间值，时间戳 |

##### 1.2.1.3. 字符串类型

| 类型 | 大小 | 用途 |
| :--- | :--- | :--- |
| CHAR | 0-255 Byte | 定长字符串 |
| VARCHAR | 0-65535 Byte | 变长字符串 |
| TINYBLOB | 0-255 Byte | 不超过255个字符的二进制字符串 |
| TINYTEXT | 0-255 Byte | 短文本字符串 |
| BLOB | 0-65535 Byte | 二进制形式的长文本字符串 |
| TEXT | 0-65535 Byte | 长文本数据 |
| MEDIUMBLOB | 0-16777215 Byte | 二进制形式的中等长度文本数据 |
| MEDIUMTEXT | 0-16777215 Byte | 中等长度文本数据 |
| LONGBLOB | 0-4 294 967 295 Byte | 二进制形式的极大文本数据 |
| LONGTEXT | 0-4 294 967 295 Byte | 极大文本数据 |

> 注1：<font color=Crimson>char(n) 和 varchar(n) 中括号中 n 代表字符的个数，并不代表字节个数，比如 CHAR(30) 就可以存储 30 个字符。</font>
>
> 注2：CHAR 和 VARCHAR 类型类似，但它们保存和检索的方式不同。它们的最大长度和是否尾部空格被保留等方面也不同。在存储或检索过程中不进行大小写转换。
>
> 注3：BINARY 和 VARBINARY 类似于 CHAR 和 VARCHAR，不同的是它们包含二进制字符串而不要非二进制字符串。也就是说，它们包含字节字符串而不是字符字符串。这说明它们没有字符集，并且排序和比较基于列值字节的数值值。

#### 1.2.2. 数据表的字段属性

##### 1.2.2.1. 主键

主键关键字是：`primary`。可以是表中的一个或多个字段。<font color=Crimson>它的作用是唯一地标识表中的某一条记录。一个表只能有一个主键；并且主键的列不能包含空值和重复值。</font>（主键是可选的）

主键可以在 `CREATE TABLE` 和 `ALTER TABLE` 语句中定义和修改，语法如下：

    PRIMARY KEY (column_name)

建立主键应该遵循的一般原则：

1. 主键应当对是用户没有意义的。
2. 永远不要更新主键，如果主键需要更新，则说明违反了对用户没有意义的原则。
3. 主键不应包含动态变化的数据，如时间戳、创建时间等。
4. 主键应当有系统自动生成。

##### 1.2.2.2. 外键

在两个表的关系中，当一张表（如表A ）的主关键字被包含在另一张表（如表B）中时，A 表中的主关键字便成为 B 表的外键（外关键字）。B 表称为主表，A 表称为从表。

<font color=Crimson>外键主要用于保持数据一致性，完整性，避免冗余数据，使两张或多张表形成关联。</font>

**在创建表时创建外键：**

    CREATE TABLE xxx(
        xxx,
        xxx,
        FOREIGN KEY (column_name) REFERENCES 关联的从表名 (column_name)
    );

**在已存在的表中增加外键：**

    ALTER TABLE 主表名 ADD FOREIGN KEY 外键名 (column_name) REFERENCES 关联的从表名 (column_name);

##### 1.2.2.3. 自动递增

自动递增关键字为：`AUTO_INCREMENT`。<font color=Crimson>该属性只能用在类型为整型的字段中。</font>

##### 1.2.2.4. 非空

非空关键字为：`NOT NULL`。<font color=Crimson>设置为非空的列，虽然不是必须，但最好设定一个默认值，以防止意外的错误和减少增加数据时的 SQL 语句复杂度。当向数据表增加数据记录时，如果设置为非空的字段不写入数据，系统将会以默认值写入。</font>

例子：

    CREATE TABLE xxx(
        xxx,
        username char(20) NOT NULL default '默认用户名',
        xxx
    );

    或者：
    ALTER TABLE user CHANGE username username char(20) NOT NULL DEFAULT ‘defaultName’

##### 1.2.2.5. 唯一性约束

唯一性约束关键字为：`UNIQUE`。

例子：

    // 给表字段增加 UNIQUE
    ALTER TABLE xx ADD UNIQUE (column_name);
    
    // 给表去掉 UNIQUE
    ALTER TABLE xx DROP UNIQUE (column_name);

    // 创建表时增加唯一性约束语法为：UNIQUE key_name (column_name)
    CREATE TABLE xx (
        xx,
        username char(15) NOT NULL default 'xxx',
        xxx,
        UNIQUE KEY username (username)
    );

##### 1.2.2.6. 索引

数据库索引（index）是为了增加查询速度而对字段附加的一种标识。我们对表的某些可能需要经常查询的字段建立适当的索引，那么在查询该字段数据时，便会显著的加快查询速度。

语法：

    KEY key_name (column_name)

例子：

    // 创建表时增加索引
    CREATE TABLE xx (
        xx,
        username char(15) NOT NULL default 'xxx',
        email varchar(40) NOT NULL default '',
        xxx,
        UNIQUE KEY username (username),
        KEY email (email)
    );

#### 1.2.3. 创建数据表

语法如下：

    CREATE TABLE table_name (column_name column_type);

例子：创建一个 runoob_tbl 表：

    CREATE TABLE IF NOT EXISTS `runoob_tbl`(
        `runoob_id` INT UNSIGNED AUTO_INCREMENT,
        `runoob_title` VARCHAR(100) NOT NULL,
        `runoob_author` VARCHAR(40) NOT NULL,
        `submission_date` DATE,
        PRIMARY KEY ( `runoob_id` )
    )ENGINE=InnoDB DEFAULT CHARSET=utf8;

**创建表时，后面附带的属性解释：**

1. ENGINE：表示存储引擎类型，分为 MyISAM 和 InnoDB 两种类型。<font color=DeepPink>默认为 MyISAM。</font>
   - MyISAM：不支持事务等高级处理，强调表的性能，执行比 InnoDB 快。
   - InnoDB：支持外键等高级数据库功能，性能比 MyISAM 差。
2. CHARSET：标识数据库表的字符集，一般为 gbk、utf8 或 big5 等。
3. AUTO_INCREMENT：设置主键默认增长的开始数。

#### 1.2.4. 查看数据表

语法：

    show tables;

#### 1.2.5. 删除数据表

语法如下：

    DROP TABLE table_name

## 2. sql函数

### 2.1. replace函数

语法：

```sql
replace(original-string，search-string，replace-string)

# 样例（最终输出："hello"）
select replace("helloworld world world", "world", "") as 'test';
```

参数：

- original-string： 被搜索的字符串。可为任意长度。
- search-string： 要搜索并被 replace-string 替换的字符串。该字符串的长度不应超过 255 个字节。如果 search-string 是空字符串，则按原样返回原始字符串。
- replace-string： 该字符串用于替换 search-string。可为任意长度。如果 replace-string 是空字符串，则删除出现的所有 search-string。

**作用**：用字符串表达式3替换字符串表达式1中出现的所有字符串表达式2的匹配项。返回新的字符串。如果有某个参数为 NULL，此函数返回 NULL。

### 2.2. current_timestamp函数

语法：

```sql
current_timestamp(毫秒的位数)

# 样例1（输出当前时间：2022-07-15 15:25:53）
select current_timestamp() as 'test';

# 样例2（输出当前时间，有1位的毫秒：2022-07-15 15:26:04.0）
select current_timestamp(1) as 'test';

# 样例3（输出当前时间，有6位的毫秒：2022-07-15 15:25:49.614248）
select current_timestamp(6) as 'test';
```

**作用**：输出当前的日期与事件，格式为：%Y-%m-%d %H:%i:%s.%f。

### 2.3. unix_timestamp函数

语法：

```sql
unix_timestamp([date])

# 样例1（输出：1657870433）
select unix_timestamp(current_timestamp()) as 'test';

# 样例2（输出：1657870436.907463）
select unix_timestamp(current_timestamp(6)) as 'test';
```

**作用**：若不带参数，则返回当前的UNIX时间戳；若带上日期，将日期转换成自 '1970-01-01 00:00:00' 之后的描述。

### 2.4. from_unixtime函数

语法：

```sql
from_unixtime(unix_timestamp, format)

# 样例1（输出：15:08:55.292253）
select from_unixtime(unix_timestamp(current_timestamp(6)) - 1800, '%H:%i:%s.%f') as 'test';
```

format选项如下：

- %M 月名字(January～December)
- %W 星期名字(Sunday～Saturday)
- %D 有英语前缀的月份的日期(1st, 2nd, 3rd, 等等。）
- %Y 年, 数字, 4 位
- %y 年, 数字, 2 位
- %a 缩写的星期名字(Sun～Sat)
- %d 月份中的天数, 数字(00～31)
- %e 月份中的天数, 数字(0～31)
- %m 月, 数字(01～12)
- %c 月, 数字(1～12)
- %b 缩写的月份名字(Jan～Dec)
- %j 一年中的天数(001～366)
- %H 小时(00～23)
- %k 小时(0～23)
- %h 小时(01～12)
- %I 小时(01～12)
- %l 小时(1～12)
- %i 分钟, 数字(00～59)
- %r 时间,12 小时(hh:mm:ss [AP]M)
- %T 时间,24 小时(hh:mm:ss)
- %S 秒(00～59)
- %s 秒(00～59)
- %p AM或PM
- %w 一个星期中的天数(0=Sunday ～6=Saturday ）
- %U 星期(0～52), 这里星期天是星期的第一天
- %u 星期(0～52), 这里星期一是星期的第一天
- %% 一个文字%
- %f 毫秒？

## 3. mysql和redis之间的同步方案

如何保证 mysql 跟 redis 之间的数据一致性？to be continue...

参考链接：

- <https://zhuanlan.zhihu.com/p/365048324>
- <https://cn.bing.com/search?q=%E5%AE%9A%E6%97%B6%E5%90%8C%E6%AD%A5+mysql+%E5%88%B0redis%E4%B8%AD&form=QBLH&sp=-1&pq=%E5%AE%9A%E6%97%B6%E5%90%8C%E6%AD%A5+mysql+%E5%88%B0redis&sc=0-17&qs=n&sk=&cvid=6BD3AB04E84E43DFBBCB8633F1B75A2B>
- <https://cn.bing.com/search?q=%e4%bf%9d%e8%af%81mysql+%e5%92%8c+redis+%e4%b8%80%e8%87%b4%e6%80%a7&qs=RI&pq=redis+%e4%bf%9d%e8%af%81&sc=1-8&cvid=AB4758E0EF2E4379AA8467373767A6D6&FORM=QBLH&sp=1>
