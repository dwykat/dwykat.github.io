---
layout: post
published: false
title: "MySQL技术内幕第五章阅读笔记"
description: "MySQL技术内幕第五章阅读笔记"
categories: [MySQL]
tags: [mysql, 阅读笔记]
redirect_from:
  - /2018/09/11/
---
**题目：**
* Kramdown table of contents
{:toc .toc}
* * * 

# MySQL技术内幕 第五章 索引与算法
## 综述
InnoDB存储引擎支持一下几种常见的索引：
* B+树索引
* 全文索引
* 哈希索引

**InnoDB存储引擎支持的哈希索引是自适应的，InnoDB存储引擎会根据表的使用情况自动为表生成哈希索引，不能认为干预在一张表中生成哈希索引。**

**B+树就是传统意义上的索引，B代表balance而不是binary，B+树索引并不能找到一个给定键值的具体行。B+树索引能找到的只是被查找数据行所在的页。**

## 数据结构与算法：
### 二分查找
每页Page Directory中的槽是按照主键的顺序存放的，对于某一条具体记录的查询是通过对Page Directory进行二分查到得到的。

### 二叉查找树和平衡二叉树


### B+树

