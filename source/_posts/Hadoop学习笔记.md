---
title: Hadoop学习笔记
date: 2020-06-07 10:45:51
description: Hadoop学习笔记
categories: Hadoop
tags: 
	- Hadoop
	- 大数据
---

# Hadoop基础

## HDFS

![HDFS Architecture](https://cdn.jsdelivr.net/gh/Wanfengyueluo/images/hdfsarchitecture.png)

### 1. NameNode

- 管理HDFS的名称空间
- 配置副本策略
- 管理数据块（Block）映射信息
- 处理客户端读写请求

### 2. DataNode

- 存储实际的数据块
- 执行数据块的读写操作

### 3. Client

- 文件切分，文件上传HDFS时，CLient将文件切分成Block，然后进行上传
- 与NameNode交互，获取文件的位置信息
- 与DataNode交互，读取或写入数据
- Client提供一些命令来管理HDFS，比如NameNode格式化
- Client通过一些命令来访问HDFS，比如对HDFS增删改查操作

### 4.Secondary NameNode

- 辅助NameNode，分担工作量，比如定期合并fsimage和edits，并推送给NameNode
- 紧急情况下，辅助恢复nameNode
- 并非NameNode的热备，不能马上替换挂掉的NameNode并提供服务

## MapReduce

## Yarn