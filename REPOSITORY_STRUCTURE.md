# 仓库结构说明

本文档说明 Penetration Testing Skills 仓库的目录结构和组织方式。

## 📋 目录结构

```
pen-test-skills/
├── README.md                          # 项目主文档
├── LICENSE                            # MIT 许可证
├── CONTRIBUTING.md                    # 贡献指南
├── REPOSITORY_STRUCTURE.md            # 仓库结构说明
├── .gitignore                         # Git 忽略文件
└── skills/                            # 技能目录
    ├── xss-penetration-testing/       # XSS 渗透测试技能
    │   ├── SKILL.md                   # 技能主文件
    │   └── references/                 # 参考资料
    │       ├── xss-payload-guide.md   # XSS Payload 和绕过技术完整指南
    │       ├── filter-bypass-cheatsheet.md  # 过滤器绕过速查表
    │       ├── exfiltration-verification-cheatsheet.md  # 数据外带验证速查表
    │       └── penetration-test-report-template.md  # 渗透测试报告模板
    └── sql-injection-penetration-testing/  # SQL 注入渗透测试技能
        ├── SKILL.md                   # 技能主文件
        └── references/                 # 参考资料
            ├── sql-injection-payload-guide.md  # SQL 注入 Payload 和绕过技术完整指南
            ├── filter-bypass-cheatsheet.md  # 过滤器绕过速查表
            ├── data-exfiltration-cheatsheet.md  # 数据外带验证速查表
            ├── mcp-sqlmap-guide.md    # MCP SQLMap 使用指南
            └── penetration-test-report-template.md  # 渗透测试报告模板
```

## 📁 目录说明

### 根目录文件

#### README.md
项目的主文档文件，包含：
- 项目简介和特点
- 技能列表和描述
- 安装和使用指南
- 贡献指南
- 相关资源链接

#### LICENSE
MIT 开源许可证，允许用户：
- 使用、复制、修改和分发软件
- 在商业和非商业项目中使用
- 修改和创建衍生作品

#### CONTRIBUTING.md
详细的贡献指南，包含：
- 行为准则
- 如何报告 bug 和建议新功能
- 开发流程
- 技能开发规范
- 提交指南
- 代码审查流程

#### .gitignore
Git 版本控制忽略文件，排除：
- 操作系统文件（.DS_Store, Thumbs.db）
- 编辑器和 IDE 配置
- Python 缓存和构建文件
- 技能发布文件（*.skill）
- 临时文件和日志

### skills/ 目录

存放所有渗透测试技能的根目录。

#### xss-penetration-testing/
XSS 渗透测试技能目录，包含完整的 XSS 测试能力。

##### SKILL.md
技能的主文件，包含：
- YAML frontmatter（name 和 description）
- 技能概述
- 核心能力
- 测试方法论
- 使用示例
- 安全注意事项

##### references/
存放详细参考资料和文档的目录。

###### xss-payload-guide.md
XSS Payload 和绕过技术完整指南（20 个章节）：
- XSS Payload 基础形式
- 编码绕过技术
- JavaScript 语法绕过
- 标签绕过技术
- 事件处理器绕过
- 过滤绕过技术
- 上下文相关绕过
- 高级绕过技术
- 注释绕过技术
- 冷门知识点汇总
- 实战绕过案例（0x00-0x12）
- 安全修复建议
- WAF 绕过技术
- DOM XSS 技术
- CSP 绕过技术

###### penetration-test-report-template.md
标准化的渗透测试报告模板，包含：
- 测试概览和摘要
- 详细测试结果格式
- 核心知识点总结
- 修复建议汇总
- 结论部分

###### filter-bypass-cheatsheet.md
过滤器绕过速查表，包含：
- 常见过滤器绕过表
- 冷门知识点速查
- 实战案例速查
- 事件处理器速查
- 编码速查
- 测试策略

###### exfiltration-verification-cheatsheet.md
数据外带验证速查表，包含：
- DNS 外带技术
- HTTP 外带技术
- WebSocket 外带技术
- 数据验证方法
- 实战案例

#### sql-injection-penetration-testing/
SQL 注入渗透测试技能目录，包含完整的 SQL 注入测试能力，集成 MCP SQLMap 自动化扫描。

##### SKILL.md
技能的主文件，包含：
- YAML frontmatter（name 和 description）
- 技能概述
- 核心能力
- 测试方法论（4 个阶段）
- MCP SQLMap 自动化扫描流程（8 个步骤）
- 数据库类型识别
- 注入类型分类
- 使用示例
- 参考资料

##### references/
存放详细参考资料和文档的目录。

###### sql-injection-payload-guide.md
SQL 注入 Payload 和绕过技术完整指南（19 个章节）：
- SQL 注入基础
- 经典注入 Payload
- 盲注 Payload
- 时间盲注 Payload
- 二阶注入 Payload
- 堆叠查询注入 Payload
- 数据库特定 Payload
- NoSQL 注入 Payload
- 过滤器绕过技术
- 编码绕过技术
- 空白字符绕过
- 关键字绕过
- 函数绕过
- 注释绕过
- 实战案例（19.1-19.10）
- MCP SQLMap 集成示例
- 数据提取技术
- 安全修复建议

###### filter-bypass-cheatsheet.md
过滤器绕过速查表，包含：
- 常见过滤器绕过表
- 编码绕过速查
- 空白字符绕过速查
- 关键字绕过速查
- 函数绕过速查
- 注释绕过速查
- 实战案例速查

###### data-exfiltration-cheatsheet.md
数据外带验证速查表，包含：
- DNS 外带技术
- HTTP 外带技术
- 文件操作外带
- 错误信息外带
- 时间盲注验证
- 布尔盲注验证
- 实战案例

###### mcp-sqlmap-guide.md
MCP SQLMap 使用指南，包含：
- MCP SQLMap 概述
- 安装和配置
- API 接口详解（submit_scan, check_scan_status, get_scan_result, list_all_tasks）
- 使用示例
- 高级用法（批量扫描、并发处理、结果解析）
- 混合测试策略
- 最佳实践
- 故障排除

###### penetration-test-report-template.md
标准化的渗透测试报告模板，包含：
- 测试概览和摘要
- MCP SQLMap 自动扫描结果
- 详细测试结果格式
- 核心知识点总结
- 修复建议汇总
- MCP SQLMap 扫描记录
- 结论部分

## 🚀 添加新技能

### 步骤 1：创建技能目录

```bash
cd skills
mkdir your-skill-name
cd your-skill-name
mkdir references
```

### 步骤 2：编写 SKILL.md

参考 [CONTRIBUTING.md](../CONTRIBUTING.md#技能开发规范) 中的规范编写技能文件。

**示例结构**：

```markdown
---
name: your-skill-name
description: 清晰描述技能的用途和触发条件
---

# 技能概述

## 核心能力

## 测试方法论

## 参考资料

详细的文档请参考：
- [详细指南](references/guide.md)
- [速查表](references/cheatsheet.md)
- [报告模板](references/template.md)
```

### 步骤 3：添加参考资料

将详细的文档放在 `references/` 目录：

**基础参考资料**：
- `guide.md`：详细指南
- `template.md`：模板文件
- `cheatsheet.md`：速查表

**可选参考资料**：
- `mcp-guide.md`：MCP 工具使用指南（如果技能集成了 MCP 工具）
- `exfiltration-cheatsheet.md`：数据外带验证速查表（如果涉及数据提取）
- `bypass-cheatsheet.md`：过滤器绕过速查表（如果涉及绕过技术）

### 步骤 4：测试技能

在真实场景中测试技能的所有功能：
- 验证所有 Payload
- 测试绕过技术
- 检查文档准确性
- 验证 MCP 工具集成（如果适用）

### 步骤 5：提交更改

```bash
git add skills/your-skill-name/
git commit -m "feat: add your-skill-name

- Implement core functionality
- Add comprehensive documentation
- Include reference materials
- Add MCP tool integration (if applicable)"
```

### 示例：SQL 注入技能结构

```
sql-injection-penetration-testing/
├── SKILL.md
│   ├── 技能概述
│   ├── 核心能力
│   ├── 测试方法论（4 个阶段）
│   ├── MCP SQLMap 自动化扫描流程（8 个步骤）
│   ├── 数据库类型识别
│   ├── 注入类型分类
│   ├── 使用示例
│   └── 参考资料
└── references/
    ├── sql-injection-payload-guide.md      # 19 个章节的完整指南
    ├── filter-bypass-cheatsheet.md         # 过滤器绕过速查表
    ├── data-exfiltration-cheatsheet.md     # 数据外带验证速查表
    ├── mcp-sqlmap-guide.md                 # MCP SQLMap 使用指南
    └── penetration-test-report-template.md # 渗透测试报告模板
```

## 📦 发布技能

### 创建 .skill 文件

使用技能打包工具创建可发布的 .skill 文件：

```bash
# 在技能目录中
cd skills/your-skill-name

# 打包技能（如果使用打包脚本）
python ../../scripts/package_skill.py .
```

### 更新 README.md

在 README.md 中添加新技能的描述：

```markdown
### 你的技能名称 (your-skill-name)

**核心能力**：
- [能力 1]
- [能力 2]
- [能力 3]

**包含内容**：
- [内容 1]
- [内容 2]

详细文档：[查看技能文档](skills/your-skill-name/)
```

## 🔄 维护指南

### 更新技能

当需要更新技能时：

1. 修改技能文件和参考资料
2. 测试所有更改
3. 更新版本号（如果适用）
4. 提交更改并创建 PR

### 更新文档

当更新文档时：

1. 更新相关文档文件
2. 检查所有链接是否有效
3. 更新目录（如果添加了新章节）
4. 提交更改

### 版本控制

使用语义化版本控制：
- 主版本号：不兼容的 API 修改
- 次版本号：向下兼容的功能性新增
- 修订号：向下兼容的问题修正

## 📞 获取帮助

如果你对仓库结构有任何疑问：

- 查看 [CONTRIBUTING.md](../CONTRIBUTING.md)
- 在 [Discussions](https://github.com/applevzxy/pen-test-skills/discussions) 中提问
- 创建 [Issue](https://github.com/applevzxy/pen-test-skills/issues)

---

**最后更新**: 2026-02-11
**版本**: 2.0
