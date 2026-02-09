# MCP SQLMap 使用指南

## 目录

1. [MCP SQLMap 概述](#mcp-sqlmap-概述)
2. [安装和配置](#安装和配置)
3. [API 接口详解](#api-接口详解)
4. [使用示例](#使用示例)
5. [高级用法](#高级用法)
6. [与手动测试结合](#与手动测试结合)
7. [最佳实践](#最佳实践)
8. [故障排除](#故障排除)

---

## MCP SQLMap 概述

### 什么是 MCP SQLMap

MCP SQLMap 是一个基于 FastMCP 的 SQLMap 扫描服务器，提供了自动化 SQL 注入检测能力。它通过 MCP（Model Context Protocol）协议与 AI 助手集成，实现了异步任务处理和完整的 HTTP 请求分析。

### 核心特性

- ✅ **自动化检测**：自动识别 SQL 注入漏洞
- ✅ **异步处理**：支持后台并发扫描
- ✅ **HTTP 请求分析**：完整的请求解析和处理
- ✅ **智能识别**：自动识别 4 种注入类型
- ✅ **结果分析**：提供详细的漏洞报告

### 支持的注入类型

1. **Boolean 盲注**：基于布尔响应的盲注
2. **报错注入**：基于错误信息的注入
3. **时间盲注**：基于响应时间的盲注
4. **联合查询注入**：基于 UNION 查询的注入

---

## 安装和配置

### 前置要求

- Python 3.7+
- SQLMap 工具
- MCP 兼容的 AI 平台（如 Claude Code）

### 安装步骤

#### 1. 克隆仓库

```bash
git clone https://github.com/godzeo/mcp-sqlmap.git
cd mcp-sqlmap
```

#### 2. 安装依赖

```bash
pip install -r requirements.txt
```

#### 3. 配置 MCP 服务器

**方式一：Claude Code 命令行集成**

```bash
claude mcp add sqlmap-scanner uv --directory /path/to/mcp-sqlmap run sqlmap-mcp
```

重启 Claude Code 使配置生效：

```bash
claude mcp list
```

**方式二：手动配置 MCP**

复制配置文件：

```bash
cp .mcp.json ~/.config/claude-code/mcp.json
```

修改路径，更新配置中的绝对路径：

```json
{
  "mcpServers": {
    "sqlmap-scanner": {
      "type": "stdio",
      "command": "python3",
      "args": ["/your/path/to/mcp-sqlmap/sqlmap_mcp_server.py"],
      "cwd": "/your/path/to/mcp-sqlmap"
    }
  }
}
```

**方式三：其他 MCP 兼容平台**

支持 MCP 协议的其他 AI 平台可以使用相同的配置：

**Cline (VSCode 扩展)**

```json
{
  "mcpServers": {
    "sqlmap-scanner": {
      "type": "stdio",
      "command": "python3",
      "args": ["sqlmap_mcp_server.py"],
      "cwd": "/path/to/mcp-sqlmap"
    }
  }
}
```

**Continue.dev**

```json
{
  "mcpServers": {
    "sqlmap-scanner": {
      "type": "stdio",
      "command": "python3",
      "args": ["sqlmap_mcp_server.py"],
      "cwd": "/path/to/mcp-sqlmap"
    }
  }
}
```

### 环境变量配置

可选的环境变量设置：

```json
{
  "env": {
    "SQLMAP_TIMEOUT": "120",
    "SQLMAP_LEVEL": "2",
    "SQLMAP_RISK": "2",
    "DEBUG": "false"
  }
}
```

**环境变量说明**：

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| SQLMAP_TIMEOUT | 扫描超时时间（秒） | 120 |
| SQLMAP_LEVEL | 测试等级（1-5） | 2 |
| SQLMAP_RISK | 风险等级（1-3） | 2 |
| DEBUG | 调试模式 | false |

---

## API 接口详解

### 1. submit_scan - 提交扫描任务

**功能**：提交 SQL 注入扫描任务

**参数**：
- `http_request` (str): HTTP 请求报文原文

**返回值**：
- `task_id` (str): 任务 ID，用于后续查询状态和结果

**http_request 格式**：

**GET 请求示例**：

```
GET /Less-2/?id=1 HTTP/1.1\nHost: localhost:8888\nUser-Agent: Mozilla/5.0\n\n
```

**POST 请求示例**：

```
POST /Less-11/ HTTP/1.1\nHost: localhost:8888\nContent-Type: application/x-www-form-urlencoded\nContent-Length: 40\nUser-Agent: Mozilla/5.0\n\nuname=123&passwd=test&submit=Submit
```

**格式说明**：
- 换行符使用 `\n`（JSON 转义格式）
- 必须包含 `Host:` 头部
- GET 请求以 `\n\n` 结束
- POST 请求头部后用 `\n\n` 分隔，然后跟请求体

**使用示例**：

```python
task_id = submit_scan(
    http_request="GET /Less-2/?id=1 HTTP/1.1\nHost: localhost:8888\nUser-Agent: Mozilla/5.0\n\n"
)
```

### 2. check_scan_status - 查询扫描状态

**功能**：查询扫描任务的当前状态

**参数**：
- `task_id` (str): 任务 ID

**返回值**：
- `status` (str): 任务状态（pending/running/completed/failed）

**使用示例**：

```python
status = check_scan_status(task_id)
```

**状态说明**：

| 状态 | 说明 |
|------|------|
| pending | 任务已提交，等待执行 |
| running | 任务正在执行中 |
| completed | 任务已完成 |
| failed | 任务执行失败 |

### 3. get_scan_result - 获取扫描结果

**功能**：获取扫描任务的完整结果

**参数**：
- `task_id` (str): 任务 ID

**返回值**：
- `result` (str): 扫描结果，包含检测到的漏洞信息

**使用示例**：

```python
result = get_scan_result(task_id)
```

**结果格式**：

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

### 4. list_all_tasks - 列出所有任务

**功能**：列出所有扫描任务及其状态

**参数**：无

**返回值**：
- `tasks` (str): 所有任务列表

**使用示例**：

```python
tasks = list_all_tasks()
```

**返回格式**：

```
任务列表：
1. task_123456 - running
2. task_123457 - completed
3. task_123458 - pending
```

---

## 使用示例

### 示例 1：基础扫描流程

```python
# 1. 准备 HTTP 请求
http_request = "GET /Less-2/?id=1 HTTP/1.1\nHost: localhost:8888\nUser-Agent: Mozilla/5.0\n\n"

# 2. 提交扫描任务
task_id = submit_scan(http_request=http_request)
print(f"任务 ID: {task_id}")

# 3. 查询扫描状态
while True:
    status = check_scan_status(task_id)
    print(f"任务状态: {status}")
    
    if status == "completed":
        break
    elif status == "failed":
        print("扫描失败")
        break
    
    time.sleep(5)

# 4. 获取扫描结果
result = get_scan_result(task_id)
print(f"扫描结果:\n{result}")
```

### 示例 2：批量扫描

```python
# 准备多个 HTTP 请求
requests = [
    "GET /Less-1/?id=1 HTTP/1.1\nHost: localhost:8888\nUser-Agent: Mozilla/5.0\n\n",
    "GET /Less-2/?id=1 HTTP/1.1\nHost: localhost:8888\nUser-Agent: Mozilla/5.0\n\n",
    "GET /Less-3/?id=1 HTTP/1.1\nHost: localhost:8888\nUser-Agent: Mozilla/5.0\n\n"
]

# 提交多个扫描任务
task_ids = []
for req in requests:
    task_id = submit_scan(http_request=req)
    task_ids.append(task_id)
    print(f"提交任务: {task_id}")

# 查询所有任务状态
tasks = list_all_tasks()
print(f"所有任务:\n{tasks}")

# 等待所有任务完成
completed_tasks = []
while len(completed_tasks) < len(task_ids):
    for task_id in task_ids:
        if task_id not in completed_tasks:
            status = check_scan_status(task_id)
            if status == "completed":
                result = get_scan_result(task_id)
                print(f"任务 {task_id} 完成:\n{result}")
                completed_tasks.append(task_id)
            elif status == "failed":
                print(f"任务 {task_id} 失败")
                completed_tasks.append(task_id)
    time.sleep(5)
```

### 示例 3：POST 请求扫描

```python
# 准备 POST 请求
http_request = "POST /Less-11/ HTTP/1.1\nHost: localhost:8888\nContent-Type: application/x-www-form-urlencoded\nContent-Length: 40\nUser-Agent: Mozilla/5.0\n\nuname=123&passwd=test&submit=Submit"

# 提交扫描任务
task_id = submit_scan(http_request=http_request)
print(f"任务 ID: {task_id}")

# 查询扫描状态
status = check_scan_status(task_id)
print(f"任务状态: {status}")

# 获取扫描结果
result = get_scan_result(task_id)
print(f"扫描结果:\n{result}")
```

### 示例 4：带 Cookie 的请求扫描

```python
# 准备带 Cookie 的请求
http_request = "GET /admin/dashboard HTTP/1.1\nHost: example.com\nCookie: session_id=abc123; user_token=xyz789\nUser-Agent: Mozilla/5.0\n\n"

# 提交扫描任务
task_id = submit_scan(http_request=http_request)
print(f"任务 ID: {task_id}")

# 获取扫描结果
result = get_scan_result(task_id)
print(f"扫描结果:\n{result}")
```

---

## 高级用法

### 1. 自定义扫描参数

通过环境变量自定义扫描参数：

```json
{
  "env": {
    "SQLMAP_TIMEOUT": "300",
    "SQLMAP_LEVEL": "3",
    "SQLMAP_RISK": "3"
  }
}
```

**参数说明**：

- **SQLMAP_LEVEL**：测试等级（1-5）
  - 1: 基础测试
  - 2: 默认测试
  - 3: 扩展测试
  - 4: 深度测试
  - 5: 全面测试

- **SQLMAP_RISK**：风险等级（1-3）
  - 1: 低风险
  - 2: 中风险
  - 3: 高风险

### 2. 并发扫描

```python
import threading

def scan_url(http_request):
    task_id = submit_scan(http_request=http_request)
    print(f"提交任务: {task_id}")
    
    while True:
        status = check_scan_status(task_id)
        if status in ["completed", "failed"]:
            break
        time.sleep(2)
    
    result = get_scan_result(task_id)
    return result

# 准备多个请求
requests = [
    "GET /url1/?id=1 HTTP/1.1\nHost: example.com\n\n",
    "GET /url2/?id=1 HTTP/1.1\nHost: example.com\n\n",
    "GET /url3/?id=1 HTTP/1.1\nHost: example.com\n\n"
]

# 并发扫描
threads = []
for req in requests:
    thread = threading.Thread(target=scan_url, args=(req,))
    thread.start()
    threads.append(thread)

for thread in threads:
    thread.join()
```

### 3. 结果解析和分析

```python
import json

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

# 使用示例
result = get_scan_result(task_id)
parsed = parse_scan_result(result)
print(json.dumps(parsed, indent=2))
```

---

## 与手动测试结合

### 1. 自动化扫描 + 手动验证

```python
# 步骤 1：使用 MCP SQLMap 自动扫描
http_request = "GET /product?id=1 HTTP/1.1\nHost: example.com\n\n"
task_id = submit_scan(http_request=http_request)
result = get_scan_result(task_id)

# 步骤 2：分析自动扫描结果
if "SUCCESS" in result:
    print("自动扫描发现漏洞")
    
    # 步骤 3：手动验证和深入测试
    print("开始手动验证...")
    
    # 使用手动 Payload 验证
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

### 2. 手动测试 + 自动化扫描

```python
# 步骤 1：手动测试发现潜在漏洞
manual_test_url = "http://example.com/product?id=1"
manual_payload = "' OR 1=1--"

# 步骤 2：确认漏洞存在后，使用 MCP SQLMap 深入扫描
if manual_test_successful:
    http_request = f"GET /product?id=1 HTTP/1.1\nHost: example.com\n\n"
    task_id = submit_scan(http_request=http_request)
    result = get_scan_result(task_id)
    
    # 步骤 3：分析详细扫描结果
    print("详细扫描结果:")
    print(result)
```

### 3. 混合测试策略

```python
def hybrid_testing_strategy(url):
    # 阶段 1：快速手动测试
    print("阶段 1：快速手动测试")
    quick_tests = [
        "' OR 1=1--",
        "' AND 1=2--",
        "' UNION SELECT 1--"
    ]
    
    vulnerable = False
    for payload in quick_tests:
        if test_payload(url, payload):
            vulnerable = True
            break
    
    # 阶段 2：如果发现漏洞，使用 MCP SQLMap 深入扫描
    if vulnerable:
        print("阶段 2：深入自动化扫描")
        http_request = build_http_request(url)
        task_id = submit_scan(http_request=http_request)
        result = get_scan_result(task_id)
        print(result)
        
        # 阶段 3：基于扫描结果进行手动利用
        print("阶段 3：手动利用")
        manual_exploit(url, result)
    else:
        print("未发现明显漏洞")
```

---

## 最佳实践

### 1. 请求准备

- **完整的 HTTP 头部**：包含所有必要的头部信息
- **正确的换行符**：使用 `\n` 作为换行符
- **Host 头部**：必须包含 Host 头部
- **请求体分隔**：POST 请求使用 `\n\n` 分隔头部和请求体

### 2. 任务管理

- **记录任务 ID**：保存任务 ID 以便后续查询
- **定期检查状态**：定期查询任务状态，避免长时间轮询
- **批量处理**：合理控制并发任务数量
- **结果保存**：及时保存扫描结果

### 3. 结果分析

- **仔细阅读结果**：完整阅读扫描结果
- **验证漏洞**：对发现的漏洞进行手动验证
- **评估风险**：根据实际情况评估风险等级
- **生成报告**：使用扫描结果生成详细报告

### 4. 性能优化

- **合理设置超时**：根据目标响应时间设置超时
- **调整测试等级**：根据需要调整测试等级和风险等级
- **使用并发扫描**：合理使用并发扫描提高效率
- **避免重复扫描**：避免对同一目标重复扫描

### 5. 安全考虑

- **授权测试**：仅在授权范围内进行测试
- **避免破坏性操作**：避免对生产环境造成破坏
- **保护敏感信息**：保护测试过程中获取的敏感信息
- **及时报告漏洞**：及时向目标报告发现的漏洞

---

## 故障排除

### 常见问题

#### 1. 任务一直处于 pending 状态

**原因**：
- SQLMap 未正确安装
- MCP 服务器未正确配置
- 网络连接问题

**解决方案**：
```bash
# 检查 SQLMap 是否安装
sqlmap --version

# 检查 MCP 服务器配置
claude mcp list

# 重启 MCP 服务器
```

#### 2. 扫描失败

**原因**：
- HTTP 请求格式不正确
- 目标服务器不可访问
- 网络连接问题

**解决方案**：
```bash
# 检查 HTTP 请求格式
# 确保使用 \n 作为换行符
# 确保包含 Host 头部

# 测试目标服务器可访问性
curl -I http://example.com
```

#### 3. 无法获取扫描结果

**原因**：
- 任务 ID 不正确
- 任务未完成
- 结果文件损坏

**解决方案**：
```python
# 检查任务状态
status = check_scan_status(task_id)
print(f"任务状态: {status}")

# 列出所有任务
tasks = list_all_tasks()
print(f"所有任务:\n{tasks}")
```

#### 4. 扫描超时

**原因**：
- 目标服务器响应慢
- 网络延迟高
- 超时时间设置过短

**解决方案**：
```json
{
  "env": {
    "SQLMAP_TIMEOUT": "300"
  }
}
```

### 调试技巧

#### 1. 启用调试模式

```json
{
  "env": {
    "DEBUG": "true"
  }
}
```

#### 2. 检查日志

```bash
# 查看 MCP 服务器日志
tail -f /path/to/mcp-sqlmap/logs/server.log

# 查看 SQLMap 日志
tail -f /path/to/mcp-sqlmap/logs/sqlmap.log
```

#### 3. 手动测试 SQLMap

```bash
# 手动运行 SQLMap 测试
sqlmap -u "http://example.com/product?id=1" --batch
```

---

## 附录

### A. HTTP 请求格式示例

#### GET 请求

```
GET /path?param=value HTTP/1.1\nHost: example.com\nUser-Agent: Mozilla/5.0\nAccept: */*\n\n
```

#### POST 请求

```
POST /path HTTP/1.1\nHost: example.com\nContent-Type: application/x-www-form-urlencoded\nContent-Length: 20\nUser-Agent: Mozilla/5.0\n\nparam1=value1&param2=value2
```

#### 带 Cookie 的请求

```
GET /path HTTP/1.1\nHost: example.com\nCookie: session_id=abc123; user_token=xyz789\nUser-Agent: Mozilla/5.0\n\n
```

#### 带 Authorization 的请求

```
GET /path HTTP/1.1\nHost: example.com\nAuthorization: Bearer token123\nUser-Agent: Mozilla/5.0\n\n
```

### B. 配置文件位置

| 平台 | 配置文件位置 |
|------|------------|
| Claude Code | ~/.config/claude-code/mcp.json |
| Cline | VS Code 设置中的 MCP 配置 |
| Continue.dev | .continue/config.json |
| 其他 MCP 客户端 | 参考各自文档 |

### C. 参考资料

- [SQLMap 官方文档](http://sqlmap.org/)
- [MCP 协议文档](https://modelcontextprotocol.io/)
- [FastMCP 文档](https://github.com/jlowin/fastmcp)

---

**文档版本**: 1.0
**最后更新**: 2026-02-09
