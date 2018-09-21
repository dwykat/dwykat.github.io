---
layout: post
title: "深入浅出mysql数据库开发 第二章学习笔记"
description: "深入浅出mysql数据库开发 第二章学习笔记"
categories: [MySQL, 读书笔记]
tags: [mysql]
redirect_from:
  - /2018/09/19/
---

**目录：**
* Kramdown table of contents
{:toc .toc}
* * * 
# 深入浅出mysql数据库开发 第二章学习笔记 {#head}
**motivation - 小知识点老是忘，好记性不如烂笔头。**

sql语句可以划分为三类：
1. DDL(Data Definition Languages)语句：数据定义语言，这些语句定义了不同的数据段、数据库、表、列、索引等数据库对象的定义。常用关键字包括`create`、`drop`、`alter`;
2. DML(Data Manipulation Language)语句：数据操作语句，也就是增删改查。主要有`insert`、`delete`、`update`和`select`等；
3. DCL(Data Control Language)语句：数据控制语句。主要是用于控制许可和访问级别，主要有`grant`, `revoke`。

以下所写语句均为实际操作语句而不是语法说明，例子一直是围绕一个**test**数据库中的**emp**和**dept**表。

## DDL {#DDL}
### 建库切库： {#create-database}
```sql
create database test;
use test;
```

### 建表： {#create-table}
```sql
create table emp(
        ename varchar(20),
        hiredate date,
        sal decimal(10, 2),
        deptno int(2));
```

如果想要简单查看表结构：`desc emp;`

如果想要查看详细建表语句：`show create table emp \G`。

### 修改表: {#modify-table}
```sql
# 修改表名
alter table emp rename emps;

# 增加表字段, first代表新加字段不会被加到最后，而是放到开头
alter table emp add age int(3) first;

# 删除表字段
alter table emp drop age;

# 字段改名或者修改定义
alter table emp change age age1 int(3);

# 修改字段并改变字段顺序
alter table emp modify age int(4) after ename;
```

## DML {#DML}
### 增删改查 {#CRUD}
```sql
# 增
insert into emp values('lisa', '2000-01-01', 2000, 1);
# 部分增
insert into emp(ename, sal) values('dony', 1000);
# 一次性增多个
insert into emp values('bob', '2001-01-01', 3000, 2), ('david', '2002-01-01', 1000, 4);

# 改
update emp set sal=3000 where ename='lisa';

# 删
delete from emp where sal=2000;

# 查
select * from emp;
# 部分查
select ename from emp ;
# 条件查
select ename form emp where sal > 2000;
select * from emp where ename='david';
# 去重查
select distinct deptno from emp;

# 排序
select * from emp order by deptno;
select * from emp order by deptno, sal;

# 限制结果数量
select * from emp limit 1; -- 返回的是从开头开始的1条数据
# limit后面可以加两个参数，第一个为起始索引, 起始索引默认为0，第二个为限制结果行数
select * from emp limit 1, 2; -- 返回的是从第二条数据开始的两条数据
```

#### 聚合 {#group}
很多情况下，用户都需要进行一些汇总操作，比如统计整个公司的人数或者统计每个部分的人数，这个时候就要用到聚合操作，由于聚合操作比较多这里先给出语法：

```sql
select [field1, field2, ...fieldn] fun_name
from tablename
[where where_condition]
[group by field1, field2,...fieldn
[with rollup]]
[having where_condition]
```
参数说明如下：
* `fun_name`表示聚合函数，常用的有`sum`, `count(*)`, `max`, `min`;
* `group by`关键字表示要进行分类聚合的字段；
* `with rollup`是可选的，表明是否对分类聚合后的结果进行再汇总；
* `having`关键字表示对分类后的结果再进行条件的过滤。

**例子：**
```sql
# 统计公司各个部门的人数：
select deptno, count(1) from emp group by deptno;

# 统计公司各个部门人数的同时统计总人数（用到with rollup对分类聚合后的结果进行汇总）
select deptno, count(1) from emp group by deptno with rollup;

# 统计人数大于1的部门：
select deptno, count(1) from emp group by deptno having count(1) > 1;

# 统计员工的薪水总额、最高和最低薪水：
select sum(sal), max(sal), min(sal) from emp;
```

### 表连接 {#join}
从大类上，表连接分为内连接和外连接，它们之间的最主要区别是内连接仅选出两张表中互相匹配的记录，而外连接会选出其他不匹配的记录。最常用的是内连接。

为了测试表连接我们创建一个新表`dept`，包含部门编号和部门名称：
```sql
create table dept(
        deptno int(2),
        deptname varchar(20));
```

```sql
select ename, deptname from emp, dept where emp.deptno=dept.deptno;
```

外连接有分为左连接和右连接，具体定义如下：
* 左连接：包含所有的左边表中的记录甚至是右边表中没有和它匹配的记录；
* 右连接：包含所有的右边表中的记录甚至是左边表中没有和它匹配的记录。

```sql
# 左连接
select ename, deptname from emp left join dept on emp.deptno=dept.deptno;

# 右连接
select ename, deptname from dept right join emp on dept.deptno=emp.deptno;
```

### 子查询 {#subquery}
有些情况下，当进行查询的时候，需要的条件是另外一个`select`语句的结果，这个时候就要用到子查询。用于子查询的关键字主要包括`in`、`not in`、`=`、`!=`、`exits`、`not exists`等。

```sql
# 从emp表中查询出所有部门在dept表中所有记录：
select * from emp where deptno in(select deptno from dept);

# 上面那个子查询有些时候可以转换为表连接
select emp.* from emp, dept where emp.deptno=dept.deptno;
```
### 记录联合 {#union}
语法：
```sql
select * from t1
union | union all
select * from t2
...
union | union all
select * from tn;
```

**`union`和`union all`的区别就是`union`会做去重处理；**

```sql
select deptno from emp
union all
select deptno from dept;
```

其实就是把结果排下来。。不知道这有啥用。。

## DCL {#DCL}
DCL语句主要是用来管理对象权限，给用户z1添加对sakila数据库添加查询和插入权限以及再取消插入权限的语句如下：

```sql
# 赋予select和insert权限
grant select, insert on sakila.* to 'z1'@'localhost' identified by '123';

# 收回insert权限
revoke insert on sakila.* from 'z1'@'localhost';
```

