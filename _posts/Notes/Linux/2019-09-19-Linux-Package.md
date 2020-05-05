---
layout: post
title: Linux软件包管理
data: 2019-09-19
categories: Notes
tags: 
 - Linux
description:
excerpt_separator: ""
---

# 软件包管理系统概述

## 常用的软件包格式

RPM：Red Hat Package Manager的缩写，大部分的Linux发行版支持的格式。

DEB：Debian上使用的软件包格式，也被使用在Ubuntu上。

## 高级软件包管理工具

APT：Advanced Package Tool的缩写，现今最成熟的软件包管理器，支持以上两种软件包格式，可以自动搜寻依赖关系并执行安装。

YUM：Yellow dog Updater, Modified的缩写，仅支持RPM格式。

# 管理DEB软件包

## 安装软件包

使用dpkg命令和-i(--install)选项来安装DEB软件包。以下命令将安装foo.deb软件包。

```bash
dpkg -i foo.deb
```

-i选项将自动删除系统上原有的旧版本。

如果安装过程出现依赖和兼容问题，该命令会报错并停止，可以使用--force-选项忽略它们，并强制安装，不过这并不被推荐。

## 查看已安装的软件包

使用dpkg命令和-l选项来查看已安装的软件包，可以结合grep命令查找特定的软件包。如下命令将查找系统中的PHP版本信息。

```bash
dpkg -l | grep php
```

也可以使用-S(--search)选项相关软件包内容的安装位置。如下命令将查看系统PHP软件包所有文件的位置。

```bash
dpkg -S php
```

## 卸载软件包

使用dpkg命令和-r(--remove)选项来卸载软件包。如下命令将卸载foo软件包。

```bash
dpkg -r foo
```

所卸载的软件包可能包含其它软件所需要的依赖，请谨慎使用。也可以使用高级软件包工具。

# 管理RPM软件包

## 安装软件包

使用rpm命令和-i选项来安装软件包。rpm提供了-v和-h选项，前者用于显示该命令正在进行的工作，后者将使用一系列的“#”符号来表示安装进度。如下命令将安装foo.rpm软件包，并显示安装内容和进度。

```bash
rpm -i -v -h foo.rpm
```

rpm命令也提供了--force选项，与dpkg命令的--force-选项类似。

## 升级软件包

使用rpm命令和-U选项来升级软件包。如下命令会将foo软件升级到2.0版本。

```bash
rpm -U foo-2.0.rpm
```

## 查看已安装的软件包

使用rpm命令和-q选项来查看软件包信息，用户应指定软件包的名字而非安装文件的名字。以下命令将列出foo软件的具体版本信息。

```bash
rpm -q foo
foo-2.0
```

增加-a选项可以列出系统上所有的软件包信息，结合grep命令可以查找特定的软件包。

## 卸载软件包

使用rpm命令和-e选项来卸载软件包。以下命令将卸载foo软件包。

```bash
rpm -e foo
```

在相互依赖关系的问题上rpm命令与dpkg命令类似，不过rpm命令会及时停止卸载，可以使用--nodeps继续完成卸载。

该命令提供了--test选项，该选项下rpm工具将模拟卸载操作，而不真的执行。同时结合-vv选项来输出完整的调试信息。

# 高级软件包工具

## 下载和安装软件包

使用apt-get命令来下载软件包，不过在下载前需要对软件包信息进行更新。如下命令将下载最新的foo软件包。

```bash
apt-get update
apt-get foo
```

以下是apt-get的常用命令：

| 命令            | 描述                               |
| --------------- | ---------------------------------- |
| apt-get install | 下载并安装特定的软件包             |
| apt-get upgrade | 将系统上所有的软件包更新至最新版本 |
| apt-get remove  | 卸载特定的软件包                   |
| apt-get source  | 下载特定包的源代码文件             |
| apt-get clean   | 删除所有已下载的包文件             |

## 默认安装位置

通过apt-get安装的应用程序一般被安装/usr目录下。

二进制文件在/usr/bin

文档在/usr/share

配置文件在/etc

共享库文件在/usr/lib

## 查询软件包信息

使用apt-cache search命令可根据软件包名字的一部分来从软件包列表中检索已安装和可安装的软件包。如下命令将检索带“foo”字样的软件包。

```bash
apt-cache search foo
```

使用apt-cache depends命令可列出特定软件包的依赖关系。

# 配置APT

APT工具从网络上的一些地址下载软件包，它们被称作安装源。源列表被放在/etc/apt/sources.list中，这是一个文本文件，可自由编辑。

# 使用图形化的APT

Ubuntu上提供了“新立得软件包管理器”，可以使用图形化的APT工具。安装步骤一般为：搜索所需的软件包；双击软件包以标记；根据提示标记相应的依赖；应用标记的选项以完成下载；下载结束后系统将自动安装和配置该软件。

# 从源代码编译软件

## 下载和解压软件包

通过官方渠道下载到相应的软件源代码。下载到的文件一般以“.tar.bz2”或“.tar.gz”这样的压缩格式打包，可以使用tar zxvf命令解压。如下命令将解压foo.tar.gz压缩包。

```bash
tar zxvf foo.tar.gz
```

## 正确配置软件

Linux上所有软件都使用configure脚本来配置以源代码形式发布的软件。configure脚本根据用户提供的相关参数生成对应的makefile文件，后者指导了make命令的编译。

configure脚本一般都提供--prefix选项，用于指定软件安装的位置。如下命令指定将软件安装到/foo目录下。

```bash
./configure --prefix=/foo
```

不同的软件提供了不同的configure脚本选项，可以通过查看软件的安装文档了解。它们一般被命名为README或INSTALL。

输入配置命令后如果没有任何输出即代表配置成功了，在Linux上我们相信没有消息就是好消息。

## 编译源代码

直接在源文件目录下使用make命令即可完成编译，如果没有任何报错的话。make命令将会根据makefile文件中的规则调用合适的编译器来编译源代码。

## 安装软件到硬盘

最后在源文件目录下使用make install命令安装软件。该命令会将需要的文件复制到之前指定好的安装目录。如果之前没有指定安装目录默认将会安装到/usr/local目录下。
