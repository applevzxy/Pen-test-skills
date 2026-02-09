---
name: xss-penetration-testing
description: 专业的 XSS（跨站脚本攻击）渗透测试技能，用于识别和利用 Web 应用程序中的 XSS 漏洞。支持反射型、存储型和 DOM 型 XSS 测试，包含完整的 Payload 库、过滤绕过技术和自动化测试流程。适用于安全审计、漏洞评估和渗透测试场景。
---

# XSS 渗透测试技能

## 概述

本技能提供专业的 XSS 渗透测试能力，帮助识别和利用 Web 应用程序中的跨站脚本漏洞。基于实际渗透测试经验，包含完整的测试方法论、Payload 库和绕过技术。

## 何时使用此技能

- 对 Web 应用程序进行 XSS 漏洞评估
- 测试输入验证和输出编码机制
- 评估内容安全策略 (CSP) 的有效性
- 进行安全审计和渗透测试
- 学习和研究 XSS 攻击技术

## 核心能力

### 1. 漏洞识别
- 识别反射型 XSS 漏洞
- 识别存储型 XSS 漏洞
- 识别 DOM 型 XSS 漏洞
- 分析输入点和输出上下文

### 2. Payload 生成
- 基础 Script 标签注入
- 事件处理器注入
- HTML 属性注入
- JavaScript 伪协议
- 编码绕过技术

### 3. 过滤绕过
- 关键字过滤绕过
- 引号过滤绕过
- 圆括号过滤绕过
- 空格过滤绕过
- 正则表达式绕过

### 4. 测试执行
- 自动化 Payload 测试
- 手动验证和确认
- 漏洞利用和演示
- 报告生成

## 测试方法论

### 阶段 1：信息收集
1. 识别所有用户输入点
2. 分析输入验证机制
3. 确定输出上下文（HTML、JavaScript、URL 等）
4. 检查内容安全策略 (CSP)

### 阶段 2：Payload 测试
1. 使用基础 Payload 进行探测
2. 根据过滤机制调整 Payload
3. 尝试编码和绕过技术
4. 验证漏洞可利用性

### 阶段 3：漏洞利用
1. 构造完整的利用链
2. 演示业务影响
3. 收集证据和截图
4. 记录复现步骤

### 阶段 4：报告生成
1. 整理发现的漏洞
2. 评估风险等级
3. 提供修复建议
4. 生成详细报告

## 参考资料

详细的 Payload 库和绕过技术请参考：
- [XSS Payload 和绕过技术完整指南](references/xss-payload-guide.md)
- [渗透测试报告模板](references/penetration-test-report-template.md)
- [常见过滤器绕过表](references/filter-bypass-cheatsheet.md)

## 使用示例

### 示例 1：基础反射型 XSS 测试
```
用户：测试 https://example.com/search?q=test 是否存在 XSS 漏洞

执行步骤：
1. 访问目标 URL
2. 在搜索参数中注入基础 Payload：<script>alert(1)</script>
3. 观察响应中是否包含未过滤的脚本标签
4. 如果存在过滤，尝试绕过技术
5. 验证漏洞并记录证据
```

### 示例 2：过滤绕过测试
```
用户：测试输入点过滤了 <script> 标签，如何绕过？

执行步骤：
1. 尝试替代标签：<img src=x onerror=alert(1)>
2. 尝试事件处理器：<svg onload=alert(1)>
3. 尝试编码绕过：\u003Cscript\u003Ealert(1)\u003C/script\u003E
4. 尝试大小写绕过：<ScRiPt>alert(1)</ScRiPt>
5. 验证成功的 Payload
```

### 示例 3：生成渗透测试报告
```
用户：为发现的 XSS 漏洞生成详细报告

执行步骤：
1. 整理漏洞信息（位置、类型、Payload）
2. 评估风险等级（CVSS 评分）
3. 提供修复建议
4. 生成包含证据的详细报告
```

## 安全注意事项

- 仅在授权范围内进行测试
- 避免对生产环境造成破坏
- 及时报告发现的漏洞
- 遵守相关法律法规
- 保护测试过程中获取的敏感信息

## 工具推荐

- 浏览器开发者工具
- Burp Suite / OWASP ZAP
- XSS Hunter
- DalFox
- XSStrike

## 学习资源

- OWASP XSS 防护速查表
- PortSwigger XSS 实验室
- XSS Hunter 博客
- HackerOne 漏洞报告
