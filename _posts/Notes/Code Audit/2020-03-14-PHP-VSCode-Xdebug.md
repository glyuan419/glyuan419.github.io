---
layout: post
title: 使用VSCode+Xdebug
data: 2020-03-14
categories: 
 - Notes
 - Notes-Code Audit
tags: 
 - PHP
 - IDE
description: 
excerpt_separator: <!-- more -->

---

关于IDE的事情拖了很久，Debug一直也用的是print大法。正好上次比赛遇到了一个审计PHP代码的题目，代码量实在是太大了，所以看看现代IDE会不会有所帮助。令人遗憾的是仅仅配置相关环境就花掉了很多时间。

<!-- more -->

# PHP安装Xdebug插件

主要是要适配PHP的版本，其实只要把PHP的信息提交到Xdebug官网，后台就会自动返回详细的安装步骤。我这里还是记录一下吧。

## 安装步骤

1. 在Terminal输入以下命令，并将输出完全粘贴到Xdebug官网的向导中：

    ``` bash
    php -i
    ```

    官网返回了我的PHP主要信息：

    ```
    Xdebug installed: 2.9.3
    Server API: Command Line Interface
    Windows: no
    Zend Server: no
    PHP Version: 7.2.24-0
    Zend API nr: 320170718
    PHP API nr: 20170718
    Debug Build: no
    Thread Safe Build: no
    OPcache Loaded: no
    Configuration File Path: /etc/php/7.2/cli
    Configuration File: /etc/php/7.2/cli/php.ini
    Extensions directory: /usr/lib/php/20170718
    ```

2. 根据提示下载并解压相应的源码包

3. 安装编译PHP扩展所需要的软件，用如下命令：

    ``` bash
    apt-get install php-dev autoconf automake
    ```

4. 运行phpize：

    ``` bash
    phpize
    ```

5. 配置并编译Xdebug源码

6. 将目录modules/下生成的文件xdebug.so复制到扩展目录（见PHP信息）

7. 向PHP配置文件中加入如下信息（见PHP信息）：

    ```
    zend_extension = {PHP扩展目录}/xdebug.so
    ```

8. 根据需要更新其它的PHP配置文件

## 备注

1. `php -i`返回的内容与`phpinfo()`相同，包含了PHP版本、配置文件位置等重要信息
2. php.ini是PHP的配置文件，PHP可能有多个配置文件，以针对不同的服务，如Apache2服务、Command Line等
3. php-dev是一个依赖库，是开发、编译PHP扩展的必要组件。
4. autoconf、automake是用于自动配置和编译源码的工具
5. phpize是一个PHP扩展安装工具，可以根据源码生成配置文件
6. Zend引擎是PHP的内核

----

# 安装VSCode插件

## PHP Debug

这个扩展是一个在VSCode和Xdebug之间的适配器。

## PHP IntelliSense

PHP智能感知，提供了语法提示、定义跳转等功能。

----

# 利用VSCode调试

1. 在VSCode中打开目标文件和其所在的目录

2. 第一次使用需要配置launch.json文件，插入如下内容：

    ``` json
    "configurations": [
        {
            "name": "Listen for XDebug",
            "type": "php",
            "request": "launch",
            "port": 9000
        },
        {
            "name": "Launch currently open script",
            "type": "php",
            "request": "launch",
            "program": "${file}",
            "cwd": "${fileDirname}",
            "port": 9000
        }
    ]
    ```

3. 以上内容配置了两个功能，"Listen for XDebug"和"Launch currently open script"。前者启动后将监视PHP的CLI或其它服务，PHP服务运行后程序将在断点处停止。后者则直接在VSCode中运行PHP程序。















