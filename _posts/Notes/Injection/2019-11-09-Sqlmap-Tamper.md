---
layout: post
title: Sqlmap Tamper脚本使用
data: 2019-11-09
categories: Notes
tags: 
 - Sqlmap
description:
excerpt_separator: ""
---

# Tamper 介绍

tamper是sqlmap自带的脚本模块，用于对payload的内容进行修改。使用方式如下：

`sqlamp -u [url] --tamper [tamper]`

sqlmap自带的tamper脚本在'usr/share/sqlmap/tamper'目录下。也可以根据需求编写定制的脚本。

# Tamper脚本编写

## 基本内容

以下是一个tamper脚本的基本构成：

{% highlight python linenos %}
#!/usr/bin/env python

"""
Copyright (c) 2006-2018 sqlmap developers (http://sqlmap.org/)
See the file 'LICENSE' for copying permission
"""

from lib.core.enums import PRIORITY

__priority__ = PRIORITY.LOWEST

def dependencies():
    pass

def tamper(payload, **kwargs):
    """
    Replaces apostrophe character with its illegal double unicode counterpart

    >>> tamper("1 AND '1'='1")
    '1 AND %00%271%00%27=%00%271'
    """

    return payload.replace('\'', "%00%27") if payload else payload
{% endhighlight %}

主要有三个部分，'\_\_priority\_\_'、'dependencies()'和'tamper()'。

## \_\_priority\_\_

'\_\_priority\_\_'表示本脚本的优先级，用于'--tamper'参数后面接了多个脚本的情况。它的值有四种基本情况，也可以利用'-100~100'进行自定义：

```
__priority__ = PRIORITY.LOWEST
__priority__ = PRIORITY.LOWER
__priority__ = PRIORITY.LOW
__priority__ = PRIORITY.NORMAL
__priority__ = PRIORITY.HIGH
__priority__ = PRIORITY.HIGHER
__priority__ = PRIORITY.HIGHEST
```

## dependencies()

## tamper()

'tamper()'是整个脚本的主体。主要用于修改原本的payload，返回值为替换后的payload。 比如Kzone中通过Unicode编码关键字中的字符来绕过waf。

它接受'payload'和'**kwargs'两个参数。