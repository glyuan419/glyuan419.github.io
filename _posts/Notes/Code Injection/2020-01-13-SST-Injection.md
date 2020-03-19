---
layout: post
title: SST注入总结
data: 2020-01-13
categories: Notes
tags: 
 - SST
 - Injection
description:
excerpt_separator: ""
---

# 模板引擎

模板引擎是为了使用户界面与业务数据分离而产生的，它可以将特定格式的文档，生成一个标准的HTML文档。

常见的置换型模板，其运作方式与格式化字符串类似，就是利用正则表达式匹配特定的符号，进而生成文档。

# 常见的模板引擎

## PHP：Smarty

## PHP：Twig

# 服务端模板注入（SSTI）

## 简介

为了完成注入，要求模板的内容，至少一部分是可控的，然后向SQL注入一样修改模板语句就可以了。看下面的例子：

```php
<?php
include_once("../libs/Smarty.class.php");
$smarty=new Smarty();

$ssti = "{{7*7}}";
$username = "pepelon";

$smarty->assign("username", $username);

$file = "Hello, {\$username}. This is $ssti.";
$file = "data:," . $file;

$smarty->display($file);
?>

```

假设`$ssti`和`$username`都是用户可控的，但是`$username`的内容在传入模板的时候会被引擎自动转义，所以暂时意义不大。我们主要关注`$ssti`的部分，它是直接插入模板中的，所以我们可以利用一些模板语法。

上例将会输出：

```
Hello, pepelon. This is 49.
```

`{{7*7}}`成功被执行了。根据这个原理我们可以进行其它操作。

## 引擎检测

```python
if "${7*7}" == "49" {
	if "a{*comment*}b" == "ab" {
		Smarty
	} else {
		if "${'z'.join('ab')}" == "zab" {
			Mako
		} else {
			Unknowm
		}
	}
} else {
	if "{{7*7}}" == "49" {
		if "{{7*'7'}}" == "7777777" {
            Jinja2
		} else if "{{7*'7'}}" == "49" {
            Twig
        } else {
            Unknown
        }
	} else {
        Not vulnerable
    }
}
```

