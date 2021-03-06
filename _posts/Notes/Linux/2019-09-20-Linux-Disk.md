---
layout: post
title: Linux磁盘管理
data: 2019-09-20
categories: Notes
tags: 
 - Linux
description:
excerpt_separator: ""
---

# 关于硬盘

硬盘根据不同的接口主要可分为IDE、SCSI、SATA三类，使用IDE接口的设备被称作hd，使用SCSI和SATA接口的设备被称作sd，以英文字母标示顺序，以sd为例，第一块硬盘被称作sda，第二块被称作sdb，以此类推。

Linux规定一个硬盘最多有4个主分区，分别被命名为sda1、sda2、sda3和sda4。逻辑分区则从5开始标示，逐个递增1，数量没有限制。

一个硬盘上最多有4个分区，即主分区和扩展分区，若主分区有4个则不设扩展分区。扩展分区不直接使用，在其上可划分若干个逻辑分区以供使用。

# Linux文件系统

在Windows下，有NTFS和FAT两种文件系统，Linux下也有自己的文件系统。

## ext3fs系统和ext4fs系统

ext4fs(Fourth Extended File System)是对ext3fs(Third Extended File System)的扩展和改善。ext4fs改良了日志功能，也大大地增加了文件系统的可靠性。

日志功能是基于灾难恢复的需求而诞生的。文件系统预留一块区域来保存日志文件，当对文件进行写操作时，所有修改先写入日志文件，随后才会做出实际修改。这样当系统崩溃后就可以根据日志来恢复文件系统。

## 常用文件系统的表示方法

以下是常用文件系统的表示方法：

| 表示方法 | 描述                         |
| -------- | ---------------------------- |
| ext2     | Linux的ext2文件系统          |
| ext3     | Linux的ext3文件系统          |
| ext4     | Linux的ext4文件系统          |
| vfat     | Windows的FAT16/FAT32文件系统 |
| ntfs     | Windows的NTFS文件系统        |
| iso9660  | CD-ROM光盘的标准文件系统     |

## swap分区

swap不是文件系统，它是Linux下的一个分区，称作交换分区，作用类似于虚拟内存。即当物理内存不够用时，临时使用虚拟内存来放置一部分不常用的信息。

# 挂载文件系统

## Linux下设备的表示方法

Linux下所有的设备都被当作文件来操作，每个设备都被映射为一个特殊文件，称作“设备文件”，被放置在/dev目录下，对上层引用而言所有的设备操作都通过文件读写实现。

这些文件大多是块设备文件和字符设备文件，块设备可以随机地读写，如硬盘，字符设备只能按顺序地接受字符流，如打印机。

用户不能通过直接通过操作文件访问存储设备，所有的存储设备在使用前都要挂载到某个目录下，然后就可以像操作目录一样使用这个设备了。

## 挂载文件系统

挂载是指由操作系统使一个存储设备上的计算机文件和目录可供用户通过计算机文件系统访问的一个过程。

通过mount命令可以挂载文件系统。如下命令会将sda3硬盘分区挂载到/foo目录下。

```bash
mount /dev/sda3 /foo
```

mount会自动检测设备上的文件系统，并以相应的类型进行挂载。可以使用-r或-w选择，分别指定以只读或是读写模式挂载设备，-w是默认选项。

## 卸载文件系统

使用umount命令来卸载文件系统。如下命令将会卸载/dev/sda3上的文件系统。

```bash
umount /dev/sda3
```

文件系统只有在没有被使用的情况下才可以被卸载，使用-r参数，可以在设备正忙时以只读方式重新挂载文件系统。

# 查看磁盘的使用情况

# 检查和修复文件系统

# 在磁盘上建立文件系统

# 使用USB设备

# 压缩工具

## gzip工具

使用gzip命令来压缩文件。以下命令会将foo文件压缩为foo.gz文件。

```bash
gzip foo
```

使用gunzip命令来解压文件。以下命令会将foo.gz文件解压为foo文件。

```bash
gunzip foo.gz
```
解压时仅支持.gz、.Z、-gz、.z和-z这样的扩展名。

可以使用-l选项来查看压缩文件的压缩效果。-t选项来测试压缩文件的完整性，文件完整则不会有任何返回。增加-v选项以在详细模式下获得返回信息。

## bzip2工具

bzip2提供了更高的压缩率。文件压缩的语法与gzip相同，它也包含了与gzip相同的-t和-v选项。解压使用-d选项或使用bunzip2命令。以下两条命令均可解压foo.bz2文件为foo文件。

```bash
bzip2 -d foo.bz2
bunzip foo.bz2
```

如果压缩文件不以.bz2、.bz、.tbz2、.tbz或.bzip2结尾，解压得到的文件将增加“.out”作为扩展名。

# 存档工具

## tar工具

使用tar命令来打包文件。以下命令会将当前目录下的foo子目录打包为bar.tar文件。

```bash
tar -c -f bar.tar foo
```

以下是tar命令常用的选项：

| 选项 | 描述                                         |
| ---- | -------------------------------------------- |
| -c   | 打包文件的选项                               |
| -x   | 解开文件的选项                               |
| -f    | 指定归档文件的文件名                         |
| -v   | 显示该命令的执行过程                         |
|-w   | 每次将单个文件加入或抽出时询问用户意见       |
| -z   | 打包文件后或解开文件前使用gzip压缩或解压文件 |
| -j    | 打包文件后或解开文件前使用bzip2压缩或解压文件 |

# 安装硬盘和分区

# 高级硬盘管理

# 备份工作和系统
