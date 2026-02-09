# Penetration Testing Skills

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Skills](https://img.shields.io/badge/skills-XSS%20Testing-green.svg)](skills/)
[![Claude](https://img.shields.io/badge/platform-Claude%20AI-orange.svg)](https://claude.ai)

> 专业的渗透测试技能集合，为 Claude AI 提供专业的安全测试能力。

## 📋 目录

- [项目简介](#项目简介)
- [技能列表](#技能列表)
- [安装方法](#安装方法)
- [使用指南](#使用指南)
- [贡献指南](#贡献指南)
- [许可证](#许可证)

## 🎯 项目简介

本项目是一个专业的渗透测试技能集合，旨在为 Claude AI 提供专业的安全测试能力。每个技能都经过精心设计，包含完整的测试方法论、Payload 库和实战案例。

### 特点

- ✅ **专业级质量**：基于实际渗透测试经验开发
- ✅ **完整文档**：详细的测试流程和 Payload 参考
- ✅ **实战导向**：包含真实场景的绕过技术和案例
- ✅ **持续更新**：根据最新的安全威胁和防御技术更新
- ✅ **易于使用**：清晰的文档和示例，快速上手

## 🛠️ 技能列表

### XSS 渗透测试 (xss-penetration-testing)

专业的 XSS（跨站脚本攻击）渗透测试技能，用于识别和利用 Web 应用程序中的 XSS 漏洞。

**核心能力**：
- 漏洞识别（反射型、存储型、DOM 型）
- Payload 生成和优化
- 过滤绕过技术
- 自动化测试流程
- 详细报告生成

**包含内容**：
- 完整的 XSS Payload 库
- 20+ 种绕过技术
- 实战案例分析（0x00-0x12）
- 渗透测试报告模板
- 过滤器绕过速查表

**适用场景**：
- Web 应用程序安全审计
- 漏洞评估和渗透测试
- 内容安全策略 (CSP) 评估
- 安全研究和学习

详细文档：[查看技能文档](skills/xss-penetration-testing/)

## 📦 安装方法

### 方法 1：直接下载

1. 下载所需的 `.skill` 文件
2. 在 Claude AI 中导入技能文件
3. 开始使用

### 方法 2：克隆仓库

```bash
git clone https://github.com/yourusername/pen-test-skills.git
cd pen-test-skills
```

### 方法 3：从 GitHub 安装

1. 访问 [Releases](https://github.com/yourusername/pen-test-skills/releases) 页面
2. 下载最新版本的 `.skill` 文件
3. 在 Claude AI 中导入

## 🚀 使用指南

### 基本使用

1. **导入技能**：在 Claude AI 中导入 `.skill` 文件
2. **触发技能**：使用相关关键词或描述触发技能
3. **参考文档**：查看技能目录下的参考资料文件
4. **执行测试**：按照技能提供的流程进行测试

### 示例

#### XSS 渗透测试示例

```
用户：测试 https://example.com/search?q=test 是否存在 XSS 漏洞

Claude AI 将：
1. 分析目标 URL 和输入点
2. 选择合适的 Payload 进行测试
3. 根据过滤机制调整 Payload
4. 验证漏洞并生成报告
```

### 技能开发指南

如果你想创建自己的技能，请参考：

- [技能创建指南](https://github.com/anthropics/claude-ai-skill-creator)
- [现有技能结构](skills/)
- [最佳实践](CONTRIBUTING.md)

## 🤝 贡献指南

我们欢迎所有形式的贡献！

### 如何贡献

1. **Fork 本仓库**
   ```bash
   git clone https://github.com/yourusername/pen-test-skills.git
   ```

2. **创建特性分支**
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. **提交更改**
   ```bash
   git commit -m 'Add some feature'
   ```

4. **推送到分支**
   ```bash
   git push origin feature/your-feature-name
   ```

5. **提交 Pull Request**

### 贡献类型

- 🐛 **Bug 修复**：修复现有技能的问题
- ✨ **新技能**：添加新的渗透测试技能
- 📝 **文档改进**：改进文档和示例
- 🎨 **代码优化**：优化现有代码和脚本
- ⚡ **性能提升**：提升技能执行效率

### 技能开发规范

1. **遵循技能结构**：
   ```
   skill-name/
   ├── SKILL.md
   ├── references/
   │   ├── guide.md
   │   └── template.md
   └── scripts/
       └── tool.py
   ```

2. **编写清晰的文档**：
   - 详细的技能描述
   - 使用示例
   - 参考资料链接

3. **测试技能**：
   - 在真实场景中测试
   - 验证所有功能
   - 检查文档准确性

## 📄 许可证

本项目采用 [MIT 许可证](LICENSE) 开源。

```
MIT License

Copyright (c) 2026 Penetration Testing Skills

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## 🔗 相关资源

- [Claude AI Skills](https://claude.ai/skills)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [HackerOne](https://www.hackerone.com/)
- [Bugcrowd](https://www.bugcrowd.com/)

## 📞 联系方式

- **问题反馈**：[GitHub Issues](https://github.com/yourusername/pen-test-skills/issues)
- **功能建议**：[GitHub Discussions](https://github.com/yourusername/pen-test-skills/discussions)
- **安全问题**：请通过私有渠道报告

## ⚠️ 免责声明

本技能仅供授权的安全测试使用。未经授权的渗透测试是非法的。使用者应：

- 仅在授权范围内进行测试
- 遵守相关法律法规
- 及时报告发现的漏洞
- 避免对生产环境造成破坏

---

<div align="center">

**如果觉得这个项目有用，请给个 ⭐️**

Made with ❤️ by the Penetration Testing Community

</div>
