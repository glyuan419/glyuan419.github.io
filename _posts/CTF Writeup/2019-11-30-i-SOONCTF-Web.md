---
layout: post
title: i-SOONCTF Web Writeup
data: 2019-11-30
categories: Writeups
tags: 
 - CTF
 - Writeup
description:
---

# easy_web

## 解题思路

第一眼会看到URL中的两个字段，"img"和"cmd"，推测会有文件包含和命令执行的漏洞。再看源码，发现页面有一张图片的显示方式为"data:image/gif;base64"，确实是文件包含。HTML源码中还有一句话，"md5 is funny"，暂时不知道有什么用。

然后就开始尝试文件包含了，不出意外"img"字段为包含的文件名，其初始内容为"TXpVek5UTTFNbVUzTURabE5qYz0"。特征猜测为Base64，解一次为"MzUzNTM1MmU3MDZlNjc="，再解一次为"3535352e706e67"，这个是ASCII编码，解密结果为"555.png"。可以提前扫一遍目录，这样看到"353535"会比较留意。

然后尝试读取"index.php"的内容，如下：

```php
<?php
error_reporting(E_ALL || ~ E_NOTICE);
header('content-type:text/html;charset=utf-8');
$cmd = $_GET['cmd'];
if (!isset($_GET['img']) || !isset($_GET['cmd'])) 
    header('Refresh:0;url=./index.php?img=TXpVek5UTTFNbVUzTURabE5qYz0&cmd=');
$file = hex2bin(base64_decode(base64_decode($_GET['img'])));

$file = preg_replace("/[^a-zA-Z0-9.]+/", "", $file);
if (preg_match("/flag/i", $file)) {
    echo '<img src ="./ctf3.jpeg">';
    die("xixi～ no flag");
} else {
    $txt = base64_encode(file_get_contents($file));
    echo "<img src='data:image/gif;base64," . $txt . "'></img>";
    echo "<br>";
}
echo $cmd;
echo "<br>";
if (preg_match("/ls|bash|tac|nl|more|less|head|wget|tail|vi|cat|od|grep|sed|bzmore|bzless|pcre|paste|diff|file|echo|sh|\'|\"|\`|;|,|\*|\?|\\|\\\\|\n|\t|\r|\xA0|\{|\}|\(|\)|\&[^\d]|@|\||\\$|\[|\]|{|}|\(|\)|-|<|>/i", $cmd)) {
    echo("forbid ~");
    echo "<br>";
} else {
    if ((string)$_POST['a'] !== (string)$_POST['b'] && md5($_POST['a']) === md5($_POST['b'])) {
        echo `$cmd`;
    } else {
        echo ("md5 is funny ~");
    }
}

?>
<html>
<style>
  body{
   background:url(./bj.png)  no-repeat center center;
   background-size:cover;
   background-attachment:fixed;
   background-color:#CCCCCC;
}
</style>
<body>
</body>
</html>
```

看到主要是两个部分，命令执行的绕过和执行条件的检验。绕过的话，已经过滤了大部分的读文件命令，这里可以使用"php flag"命令来实现读取。执行条件可以通过md5碰撞来达成。

## 备注

1. 读文件：php、ca\t、rev
2. md5碰撞

# easy_serialize_php

## 解题思路

根据提示可以看到题目源码：

```php
 <?php

$function = @$_GET['f'];

function filter($img){
    $filter_arr = array('php','flag','php5','php4','fl1g');
    $filter = '/'.implode('|',$filter_arr).'/i';
    return preg_replace($filter,'',$img);
}


if($_SESSION){
    unset($_SESSION);
}

$_SESSION["user"] = 'guest';
$_SESSION['function'] = $function;

extract($_POST);

if(!$function){
    echo '<a href="index.php?f=highlight_file">source_code</a>';
}

if(!$_GET['img_path']){
    $_SESSION['img'] = base64_encode('guest_img.png');
}else{
    $_SESSION['img'] = sha1(base64_encode($_GET['img_path']));
}

$serialize_info = filter(serialize($_SESSION));

if($function == 'highlight_file'){
    highlight_file('index.php');
}else if($function == 'phpinfo'){
    eval('phpinfo();'); //maybe you can find something in here!
}else if($function == 'show_image'){
    $userinfo = unserialize($serialize_info);
    echo file_get_contents(base64_decode($userinfo['img']));
} 
```

可以发现这是一个由过滤引起的序列化逃逸，构造Payload，实现任意文件读取。

## 备注

序列化逃逸原理：

过滤前的序列化结构如下：

```
a:2:{
    s:4:"evil";a:2:{
        i:1;s:12:"foofoofoofoo";
        i:2;s:100:";i:2;s:62:"0123456789
                              0123456789
                              0123456789
                              0123456789
                              0123456789
                              0123456789
                               01";}s:4:"file";s:8:"flag.php";
    }
    s:4:"file";s:11:"notflag.php";
}
```

"evel"字段为我们上传的Payload，此时"file"字段指向文件"notflag.php"。当"foofoofoofoo"被过滤后，其结构变为如下：

```
a:2:{
    s:4:"evil";a:2:{
        i:1;s:12:"";i:2;s:100:";
        i:2;s:62:"0123456789
                  0123456789
                  0123456789
                  0123456789
                  0123456789
                  0123456789
                  01";
    }
    s:4:"file";s:8:"flag.php";
}
s:4:"file";s:11:"notflag.php";}
```

根据结构，"file"字段指向了文件"flag.php"。最后一行的内容被“挤”了出去，成为冗余，会被反序列化函数忽略。

