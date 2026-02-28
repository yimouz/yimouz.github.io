---
title: Groovy (JSR223) 字符串语法
date: 2026-02-28
tags: [Groovy, JSR223, 字符串语法, BeanShell, 对比]
categories: [groove]
description: Groovy与BeanShell字符串语法的详细对比
---

# Groovy (JSR223) 的字符串语法 ⭐⭐⭐⭐⭐

## 1. 单引号字符串（不支持插值）

```groovy
def str = 'Hello World'
def name = 'Tom'
def str2 = 'Hello ${name}'  // 输出: Hello ${name} (不会插值)
```

## 2. 双引号字符串（支持插值）

```groovy
def name = 'Tom'
def str = "Hello ${name}"  // 输出: Hello Tom
def str2 = "Line1\nLine2"  // 支持转义
```

## 3. 三单引号（多行字符串，不转义，不插值）⭐⭐⭐

```groovy
def json = '''
{
  "name": "张三",
  "path": "C:\\Users\\test\\file.txt",
  "regex": "\\d+\\.\\d+",
  "quote": "He said 'hello'"
}
'''
// 所有字符都按原样输出，无需转义
```

## 4. 三双引号（多行字符串，不转义，支持插值）⭐⭐⭐⭐⭐

```groovy
def name = "张三"
def userId = "10001"

def json = """
{
  "name": "${name}",
  "userId": "${userId}",
  "path": "C:\\Users\\test\\file.txt",
  "regex": "\\d+\\.\\d+",
  "message": "He said 'hello' and \"goodbye\""
}
"""
// 支持变量插值，且无需转义反斜杠和引号
```

## 5. 斜杠字符串（正则表达式专用）

```groovy
def pattern = /\d+\.\d+/  // 无需转义反斜杠
def path = /C:\Users\test\file.txt/  // 路径也很方便
```

# 二、BeanShell 的字符串语法（受限）

## 1. 只支持双引号字符串

```java
String str = "Hello World";
String name = "Tom";
String str2 = "Hello " + name;  // 必须用 + 拼接
```

## 2. 必须转义特殊字符

```java
// ❌ BeanShell 不支持三引号
String json = """
{
  "name": "test"
}
""";  // 语法错误！

// ✅ 必须这样写（痛苦）
String json = "{\n" +
              "  \"name\": \"test\",\n" +
              "  \"path\": \"C:\\\\Users\\\\test\\\\file.txt\"\n" +
              "}";
```

## 3. BeanShell 的多行字符串解决方案

### 方案A: 字符串拼接（传统方式）

```java
String json = "{" +
    "\"code\": 0," +
    "\"message\": \"success\"," +
    "\"data\": {" +
        "\"userId\": \"10001\"," +
        "\"name\": \"张三\"," +
        "\"path\": \"C:\\\\Users\\\\test\\\\file.txt\"" +
    "}" +
"}";
```

### 方案B: StringBuilder（稍好一点）

```java
StringBuilder sb = new StringBuilder();
sb.append("{\n");
sb.append("  \"code\": 0,\n");
sb.append("  \"message\": \"success\",\n");
sb.append("  \"data\": {\n");
sb.append("    \"userId\": \"10001\",\n");
sb.append("    \"name\": \"张三\",\n");
sb.append("    \"path\": \"C:\\\\Users\\\\test\\\\file.txt\"\n");
sb.append("  \n");
sb.append("}");

String json = sb.toString();
```

### 方案C: 使用 JSON 库（推荐）⭐⭐⭐

```java
import org.json.JSONObject;

JSONObject json = new JSONObject();
json.put("code", 0);
json.put("message", "success");

JSONObject data = new JSONObject();
data.put("userId", "10001");
data.put("name", "张三");
data.put("path", "C:\Users\test\file.txt");  // 只需转义一次

json.put("data", data);

String jsonStr = json.toString(2);  // 2 表示缩进
```

# 三、实际对比示例

## 场景：返回包含特殊字符的 JSON

### Groovy (JSR223) - 优雅 ⭐⭐⭐⭐⭐

```groovy
def userId = vars.get("userId") ?: "10001"
def filePath = "C:\Users\test\data.txt"

// 方式1: 三双引号（最推荐）
def json = """
{
  "code": 0,
  "message": "success",
  "data": {
    "userId": "${userId}",
    "name": "张三",
    "path": "${filePath}",
    "regex": "\\d+\\.\\d+",
    "quote": "He said 'hello' and \"goodbye\"",
    "sql": "SELECT * FROM users WHERE name = 'Tom'"
  }
}
"""

SampleResult.setResponseData(json, "UTF-8")
SampleResult.setSuccessful(true)

// 方式2: 三单引号（无插值）
def json = '''
{
  "code": 0,
  "message": "success",
  "data": {
    "path": "C:\Users\test\data.txt",
    "regex": "\\d+\\.\\d+",
    "quote": "He said 'hello'"
  }
}
'''
```

### BeanShell - 痛苦 ⭐

```java
String userId = vars.get("userId");
if (userId == null) userId = "10001";

// 方式1: 字符串拼接（痛苦）
String json = "{" +
    "\"code\": 0," +
    "\"message\": \"success\"," +
    "\"data\": {" +
        "\"userId\": \"" + userId + "\"," +
        "\"name\": \"张三\"," +
        "\"path\": \"C:\\\\Users\\\\test\\\\data.txt\"," +
        "\"regex\": \"\\\\d+\\\\.\\\\d+\"," +
        "\"quote\": \"He said 'hello' and \\\"goodbye\\\"\"" +
    "}" +
"}";

ResponseData = json;
IsSuccess = true;

// 方式2: 使用 JSON 库（推荐）
import org.json.JSONObject;

JSONObject response = new JSONObject();
response.put("code", 0);
response.put("message", "success");

JSONObject data = new JSONObject();
data.put("userId", userId);
data.put("name", "张三");
data.put("path", "C:\Users\test\data.txt");
data.put("regex", "\\d+\\.\\d+");
data.put("quote", "He said 'hello' and \"goodbye\"");

response.put("data", data);

ResponseData = response.toString(2);
IsSuccess = true;
```

# 四、转义字符对比表

| 字符 | Groovy 单/双引号 | Groovy 三引号 | BeanShell |
|------|------------------|---------------|-----------|
| 换行 \n | \\n | 直接换行 | \\n |
| 反斜杠 \ | \\ | \ | \\ |
| 双引号 " | \" | " | \\" |
| 单引号 ' | ' | ' | ' |
| 制表符 \t | \\t | 直接 Tab | \\t |
| 路径 C:\test | C:\\test | C:\test | C:\\\\test |
| 正则 \d+ | \\d+ | \d+ | \\\\d+ |

# 五、完整实战对比

## 需求：返回包含 SQL、正则、路径的复杂 JSON

### Groovy 实现（简洁）

```groovy
def userId = vars.get("userId") ?: "10001"
def userName = vars.get("userName") ?: "张三"

def response = """
{
  "code": 0,
  "message": "查询成功",
  "data": {
    "userId": "${userId}",
    "userName": "${userName}",
    "sql": "SELECT * FROM users WHERE name = 'Tom' AND age > 18",
    "regex": "^\\d{3}-\\d{4}-\\d{4}$",
    "filePath": "C:\Users\test\documents\report.xlsx",
    "jsonPath": "$..books[?(@.price < 10)]",
    "xpath": "//div[@class='content']//p[1]",
    "description": "He said: \"Hello, I'm Tom\" and she replied 'Hi!'"
  },
  "timestamp": ${System.currentTimeMillis()}
}
"""

SampleResult.setResponseData(response, "UTF-8")
SampleResult.setResponseCode("200")
SampleResult.setSuccessful(true)
```

### BeanShell 实现（复杂）

```java
import org.json.JSONObject;

String userId = vars.get("userId");
if (userId == null) userId = "10001";
String userName = vars.get("userName");
if (userName == null) userName = "张三";

JSONObject response = new JSONObject();
response.put("code", 0);
response.put("message", "查询成功");

JSONObject data = new JSONObject();
data.put("userId", userId);
data.put("userName", userName);
data.put("sql", "SELECT * FROM users WHERE name = 'Tom' AND age > 18");
data.put("regex", "^\\d{3}-\\d{4}-\\d{4}$");
data.put("filePath", "C:\Users\test\documents\report.xlsx");
data.put("jsonPath", "$..books[?(@.price < 10)]");
data.put("xpath", "//div[@class='content']//p[1]");
data.put("description", "He said: \"Hello, I'm Tom\" and she replied 'Hi!'");

response.put("data", data);
response.put("timestamp", System.currentTimeMillis());

ResponseData = response.toString(2);
ResponseCode = "200";
IsSuccess = true;
```

# 六、对比

| 特性 | Groovy (JSR223) | BeanShell |
|------|-----------------|-----------|
| 三引号支持 | ✅ 支持 ''' 和 """ | ❌ 不支持 |
| 字符串插值 | ✅ "Hello ${name}" | ❌ 必须用 + 拼接 |
| 多行字符串 | ✅ 原生支持 | ❌ 需要 + 或 StringBuilder |
| 转义复杂度 | ⭐ 简单 | ⭐⭐⭐⭐⭐ 复杂 |
| 代码可读性 | ⭐⭐⭐⭐⭐ 优秀 | ⭐⭐ 较差 |
| 推荐度 | ⭐⭐⭐⭐⭐ | ⭐ 不推荐 |

## 建议
- **新项目**: 强烈推荐使用 JSR223 + Groovy 的三引号语法
- **旧项目**: 如果必须用 BeanShell，使用 JSON 库而不是字符串拼接
- **迁移**: 将 BeanShell 脚本迁移到 JSR223，可以大幅简化代码
