---
layout: post
published: false
title: 深入浅出mysql数据库开发 第三章 基本数据类型 学习笔记
guid: 81e2b6557fa546fdb3ea50e82a93c311
categories: [MySQL, 读书笔记]
tags: [mysql]
redirect_from:
    - /2018/09/26/
---

**目录：**
* Kramdown table of contents
{:toc .toc}
* * * 
# 基本数据类型

|整数类型|字节|
|-----|_-----|
|TINYINT|1|
|SMALLINT|2|
|MEDIUMINT|3|
|INT, INTEGER|4|
|浮点数类型|字节|
|FLOAT|4|
|DOUBLE|8|
|定点数类型|字节|
|DEC(M,D)|M+2|
|DECIMAL(M,D)|M+2|
|位类型|字节|
|BIT(M)|1~8| 


## 日期时间类型

|日期和时间类型|字节|最小值|最大值|
|-----|-----|-----|-----|
|DATE|4|1000-01-01|9999-12-31|
|DATETIME|8|1000-01-01 00:00:00|9999-12-31 23:59:59|
|TIMESTAMP|4|1970010180001|2038年某个时刻|
|TIME|3|-838:59:59|838:59:59|
|YEAR|1|1901|2155|

主要区别如下：
1. 如果要用来表示年月日，通常用DATE表示；
2. 如果要用来表示年月日时分秒，通常用DATETIME表示；
3. 如果只用来表示时分秒，通常用TIME来表示；
4. 如果需要经常插入或者更新日期为当前系统时间，则通常使用TIMESTAMP来表示。TIMESTAMP值返回后显示为“YYYY-MM-DD HH:MM:SS”格式的字符串，显示宽度固定为19个字符。如果想要获得数字值，应在TIMESTAMP列添加+0.
5. 如果只是表示年份，可以用YEAR来表示，它比DATE占用更少的空间。YEAR有2位或4位格式的年。默认是4位格式。


