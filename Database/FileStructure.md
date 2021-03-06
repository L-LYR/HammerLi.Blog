---
Author: HammerLi
Date: 2021-09-07
---

[TOC]

# 文件结构

文件由操作系统作为一种基本结构提供，毕竟数据库是运行在操作系统之上的程序，因此在数据库角度看到的是文件的逻辑结构。块是文件存储和数据传输的基本单位，每个文件都会被分割为定长的块，大多数数据库默认采用`4~8KB`的块大小。一些数据库在初始化时可以显式指定块大小，比如Oracle（[文档](https://docs.oracle.com/cd/B19306_01/server.102/b14237/initparams041.htm#REFRN10031)）。一个块中可以包含多条记录，

## 定长记录

### 缺点

1. 块大小和记录大小无倍数关系，要么块填不满，要么记录被分割
2. 删除记录操作会使得管理十分困难

### 改进

增加文件头，记录块内记录相关的元数据，如记录数目，已删除记录构成的空闲链表

## 变长记录

