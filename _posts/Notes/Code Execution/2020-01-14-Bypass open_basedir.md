---
layout: post
title: 绕过open_basedir限制
data: 2020-01-14
categories: Notes
tags: 
 - PHP
 - Execution
description:
excerpt_separator: ""
---

# open_basedir简介

open_basedir将php文件可访问的范围限制在指定目录树中，对于超出范围的文件，php文件无法访问。

# 设置方法

## 在程序中设置

```
ini_set("open_basedir", "指定目录");
```

## 在php.ini文件中设置（PHP配置文件）

```
open_basedir="指定目录"
```

## 在httpd.conf文件中的Directory和VritualHost设置（Apache配置文件）

```
php_admin_value open_basedir “指定目录”
```

## 在fastcgl.conf文件中设置（Nginx配置文件）

```
fastcgi_param PHP_VALUE "open_basedir=指定目录"
```

# 绕过方法

