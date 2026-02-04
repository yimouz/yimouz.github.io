---
title: JMeter vars.put 操作详细指南
date: 2026-02-03
tags:
  - JMeter
  - 性能测试
  - 变量操作
categories: 性能测试
author: 性能测试工程师
summary: 本文详细介绍了 JMeter 中 vars.put() 方法的使用方法、相关函数、常见场景及注意事项，帮助测试工程师更高效地管理线程级变量。
---

# JMeter vars.put 操作详细指南

## 1. 概述

`vars` 是 JMeter 内置的**线程级变量操作对象**，用于在当前线程的不同元件间传递数据（如 PreProcessor → Sampler → PostProcessor）。`vars.put()` 是其核心方法之一，用于将数据存储为线程级变量。

> **tags: [作用域]**：`vars` 仅对当前线程可见，不同线程的 `vars` 是完全独立的（线程隔离）。


## 2. vars.put() 基本用法

### 2.1 语法

```java
// 语法：vars.put(变量名, 变量值);
// 说明：变量值必须为字符串类型（非字符串类型需先转换为字符串）
vars.put("key", "value");
```

### 2.2 示例：存储不同类型的数据

#### 示例 1：存储字符串

```groovy
// 存储字符串变量
vars.put("username", "testuser");
vars.put("password", "testpass");

// 验证存储结果
log.info("用户名：" + vars.get("username"));  // 输出：用户名：testuser
```

#### 示例 2：存储数值

> **tags: [类型转换]**：`vars.put()` 仅支持字符串，存储数值时需先转换为字符串。

```groovy
// 方法 1：使用 toString() 转换
int age = 25;
vars.put("userAge", age.toString());

// 方法 2：使用字符串拼接转换
int score = 95;
vars.put("userScore", "" + score);

// 方法 3：使用 String.valueOf() 转换
long timestamp = System.currentTimeMillis();
vars.put("timestamp", String.valueOf(timestamp));
```

#### 示例 3：存储布尔值

```groovy
boolean isActive = true;
vars.put("isUserActive", String.valueOf(isActive));  // 存储为 "true"
```


## 3. vars 相关方法

### 3.1 存储对象：vars.putObject()

当需要存储**非字符串类型**的数据（如数组、列表、Map 等）时，使用 `vars.putObject()`。

```java
// 语法：vars.putObject(变量名, 对象);
vars.putObject("key", object);
```

#### 示例：存储数组和 Map

```groovy
// 存储字符串数组
def ipArray = ["192.168.1.1", "192.168.1.2", "192.168.1.3"];
vars.putObject("ipList", ipArray);

// 存储 Map 对象
def userInfo = ["id": 1, "name": "testuser", "age": 25];
vars.putObject("userMap", userInfo);

// 验证存储结果
log.info("IP 列表长度：" + vars.getObject("ipList").size());  // 输出：3
```

### 3.2 获取变量：vars.get()

用于获取之前存储的**字符串变量**，若变量不存在返回 `null`。

```java
// 语法：vars.get(变量名);
String value = vars.get("key");
```

#### 示例

```groovy
// 获取之前存储的变量
def username = vars.get("username");
def age = vars.get("userAge");

// 使用获取的值
if (username != null) {
    log.info("用户：" + username + "，年龄：" + age);
}
```

### 3.3 获取对象：vars.getObject()

用于获取之前存储的**对象**，需进行类型强转。

```java
// 语法：vars.getObject(变量名);
Object value = vars.getObject("key");
```

#### 示例

```groovy
// 获取数组对象
def ipList = (List<String>) vars.getObject("ipList");
log.info("第一个 IP：" + ipList[0]);  // 输出：192.168.1.1

// 获取 Map 对象
def userMap = (Map<String, Object>) vars.getObject("userMap");
log.info("用户名：" + userMap.get("name"));  // 输出：testuser
```

### 3.4 移除变量：vars.remove()

从当前线程的变量空间中移除指定变量。

```java
// 语法：vars.remove(变量名);
vars.remove("key");
```

#### 示例

```groovy
// 移除变量
vars.remove("username");

// 验证是否移除成功
log.info("用户名是否存在：" + (vars.get("username") == null));  // 输出：true
```

### 3.5 清空所有变量：vars.clear()

清空当前线程的所有变量（谨慎使用，可能影响后续元件）。

```java
// 语法：vars.clear();
vars.clear();
```

#### 示例

```groovy
// 清空所有变量
vars.clear();

// 验证变量数量
log.info("变量数量：" + vars.entrySet().size());  // 输出：0
```


## 4. 常见使用场景

### 场景 1：PreProcessor 中生成数据传递给 Sampler

> **tags: [典型应用]**：在请求前生成动态数据（如唯一 ID、时间戳），并在 Sampler 中使用。

**示例**：生成唯一请求 ID 并在 HTTP Request 中使用

1. **添加 JSR223 PreProcessor**（HTTP Request 的子元件）：

```groovy
// 生成唯一请求 ID
def requestId = "REQ_" + System.currentTimeMillis();
vars.put("requestId", requestId);

// 生成时间戳
def timestamp = String.valueOf(System.currentTimeMillis());
vars.put("timestamp", timestamp);
```

2. **在 HTTP Request 中引用**：
   - **Path** 字段填写：`/api/test?requestId=${requestId}&timestamp=${timestamp}`
   - 实际请求路径会被替换为：`/api/test?requestId=REQ_1678901234567&timestamp=1678901234567`


### 场景 2：PostProcessor 中提取响应数据传递给后续 Sampler

> **tags: [数据流转]**：从响应中提取数据（如 token、用户 ID），供后续请求使用。

**示例**：提取响应中的 token 并在后续请求的 Header 中使用

1. **添加 JSR223 PostProcessor**（第一个 HTTP Request 的子元件）：

```groovy
// 获取响应体
String response = prev.getResponseDataAsString();

// 正则提取 token
def tokenMatcher = response =~ /"token":"([^"]+)"/;
if (tokenMatcher.find()) {
    String token = tokenMatcher.group(1);
    vars.put("authToken", token);
    log.info("已提取 token：" + token);
}
```

2. **在后续 HTTP Request 的 Header 中引用**：
   - 添加 Header：`Authorization` → 值填写：`Bearer ${authToken}`


### 场景 3：存储复杂数据结构

> **tags: [数据结构]**：对于数组、列表等复杂数据，使用 `vars.putObject()` 存储更高效。

**示例**：存储用户列表并在后续元件中遍历使用

1. **在 PreProcessor 中存储用户列表**：

```groovy
// 定义用户列表
def users = [
    ["id": 1, "name": "user1", "age": 20],
    ["id": 2, "name": "user2", "age": 25],
    ["id": 3, "name": "user3", "age": 30]
];

// 存储为对象
vars.putObject("userList", users);
```

2. **在后续 PostProcessor 中使用**：

```groovy
// 获取用户列表
List<Map<String, Object>> userList = (List<Map<String, Object>>) vars.getObject("userList");

// 遍历用户列表
for (int i = 0; i < userList.size(); i++) {
    Map<String, Object> user = userList.get(i);
    log.info("用户 " + (i+1) + "：" + user.get("name") + "，年龄：" + user.get("age"));
}
```


## 5. 注意事项

> **tags: [警告]**：以下事项需特别注意，避免使用错误。

### 5.1 字符串必须加引号

存储字符串时，必须用引号包围（双引号或单引号），否则 Groovy 会将其视为变量名（未定义会报错）。

```groovy
// 错误：test 被视为变量名（未定义会抛出异常）
// vars.put("var", test);  // 抛出 MissingPropertyException

// 正确：用引号包围字符串
vars.put("var", "test");
```

### 5.2 跨线程共享需使用 props

`vars` 仅对当前线程可见，若需跨线程共享数据，应使用 `props`（全局属性）。

```groovy
// 存储为全局属性（所有线程可见）
props.put("globalVar", "value");

// 在其他线程中获取
String value = props.get("globalVar");
```

### 5.3 变量替换仅在支持的字段中生效

`${变量名}` 语法仅在 JMeter 支持变量替换的字段中生效（如 HTTP Request 的路径、参数值、Header 等）。

### 5.4 性能考虑

- 避免在高频执行的元件（如循环中的 Sampler）中频繁调用 `vars.put()`/`vars.get()`，可能影响性能。
- 对于复杂数据结构，优先使用 `vars.putObject()` 存储，减少类型转换开销。


## 6. 与其他变量操作对象的对比

| 操作对象 | 作用域 | 生命周期 | 适用场景 |
|---------|--------|----------|----------|
| `vars` | 当前线程 | 线程执行期间 | 线程内数据传递 |
| `props` | 全局（所有线程） | 测试计划执行期间 | 跨线程数据共享 |
| `ctx` | 当前线程 | 线程执行期间 | 访问 JMeter 上下文（如采样器结果） |
| `prev` | 当前线程 | 当前采样器执行后 | 访问上一个采样器的结果 |


## 7. 完整示例：IP 轮询场景

### 7.1 需求

定义 IP 列表 `["192.168.1.1", "192.168.1.2", "192.168.1.3"]`，并在每次请求时按顺序轮询使用。

### 7.2 实现步骤

1. **添加 JSR223 PreProcessor**（线程组级别）：

```groovy
// 初始化 IP 列表（仅首次执行时初始化）
if (!vars.containsKey("ipListInitialized")) {
    def ipList = ["192.168.1.1", "192.168.1.2", "192.168.1.3"];
    vars.putObject("ipList", ipList);
    vars.put("ipIndex", "0");
    vars.put("ipListInitialized", "true");
    log.info("IP 列表已初始化：" + ipList);
}

// 轮询逻辑
def ipList = vars.getObject("ipList");
int index = Integer.parseInt(vars.get("ipIndex"));
int size = ipList.size();

// 获取当前 IP
String currentIp = ipList[index % size];
vars.put("currentIp", currentIp);

// 更新索引
vars.put("ipIndex", String.valueOf((index + 1) % size));

log.info("当前轮询 IP：" + currentIp);
```

2. **在 HTTP Request 中引用**：
   - **Server Name or IP** 字段填写：`${currentIp}`

### 7.3 执行结果

- 第 1 次请求：使用 IP `192.168.1.1`
- 第 2 次请求：使用 IP `192.168.1.2`
- 第 3 次请求：使用 IP `192.168.1.3`
- 第 4 次请求：循环使用 IP `192.168.1.1`


## 8. 总结

`vars.put()` 是 JMeter 中管理线程级变量的核心方法，通过本文的指南，您应已掌握：

- ✅ `vars.put()` 的基本用法和类型转换
- ✅ `vars` 相关方法（如 `putObject()`、`get()`、`remove()`）的使用
- ✅ 常见使用场景（如数据传递、响应提取、复杂数据存储）
- ✅ 注意事项和性能优化建议

合理使用 `vars` 对象，可大幅提升 JMeter 脚本的灵活性和可维护性，实现更复杂的测试场景。
