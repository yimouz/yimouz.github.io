---
title: JMeter POST 请求缺失 Header 需要手动添加
date: 2021-06-10
tags: [JMeter, HTTP Header]
category: JMeter
summary: 在进行接口测试（特别是 POST 请求）时，如果没有显式添加 `Content-Type` 头，服务端往往无法正确解析请求体，从而返回 `415 Unsupported Media Type` 或 `400 Bad Request` 错误。
---

# JMeter POST 请求缺失 Header 的解决方案

## 1. 问题背景
在进行接口测试（特别是 POST 请求）时，如果没有显式添加 `Content-Type` 头，服务端往往无法正确解析请求体，从而返回 `415 Unsupported Media Type` 或 `400 Bad Request` 错误。
最常见的场景是发送 JSON 数据，但未设置 `Content-Type: application/json`。

## 2. 核心解决方案：使用 HTTP Header Manager
这是解决该问题最标准、最灵活的方法，适用于 JSON、XML 等各种数据格式。

### 操作步骤
1.  **添加组件**：
    *   选中 **Test Plan (测试计划)** 或 **Thread Group (线程组)**。
    *   右键菜单选择：`Add` -> `Config Element` -> `HTTP Header Manager` (HTTP信息头管理器)。
2.  **配置参数**：
    *   点击底部的 `Add` 按钮新增一行。
    *   **Name**: `Content-Type`
    *   **Value**: `application/json` (如果是其他格式，填写对应的值，如 `application/xml`)

### 作用范围说明
*   **全局生效**：如果在 Test Plan 根节点下添加，所有请求都会默认带上该 Header。
*   **局部生效**：如果在某个 Thread Group 或具体的 HTTP Request 下添加，仅对该范围内的请求生效。

## 3. 常见误区：修改 jmeter.properties
**注意**：修改配置文件中的 `post_add_content_type_if_missing` 无法解决 JSON 格式的 Header 缺失问题。

*   **局限性**：该配置项开启后，**只能** 自动补全 `application/x-www-form-urlencoded`。
*   **无效操作**：将该配置项的值改为 `json` 或 `true` 都无法让 JMeter 自动添加 `application/json`。
*   **结论**：请完全忽略此配置项，始终使用 **HTTP Header Manager** 来管理请求头。

### 补充说明：隐式行为 (Implicit Behavior)
如果开启了该配置 (`true`)，其补全行为是 **隐式的**：
1.  **GUI 不可见**：你**不会**在测试计划界面看到任何自动生成的 Header Manager 组件。
2.  **结果可见**：只有在运行测试后，查看 **View Results Tree (查看结果树)** -> **Request** -> **Request Headers** 时，才会发现 `Content-Type: application/x-www-form-urlencoded` 被默默添加到了请求中。

### 补充说明：优先级与冲突 (Priority)
如果开启了自动补全，同时你又手动设置了 Header，结果是：**以你的手动设置为准**。
该配置项的名称 `..._if_missing` 揭示了其逻辑：
- **检查机制**：JMeter 发送请求前会检查是否已有 `Content-Type`。
- **用户优先**：只要检测到你设置了该 Header（无论值是 json、xml 还是其他），隐式补全行为就**直接跳过**。
- **最终结果**：**不会覆盖**，也**不会出现两个 Header**。你的设置完全生效，隐式行为完全失效。



## 4. 常见 HTTP Header 及使用场景指南

在接口测试中，正确设置 Header 是请求成功的关键。以下是高频使用的 Header 及其对应值。

### 4.1 数据格式类 (Content-Type)
告诉服务端**你发送的数据是什么格式**。

| 场景 | Header 值 | 说明 |
| :--- | :--- | :--- |
| **JSON 接口** (最常见) | `application/json` | RESTful API 标准格式 |
| **表单提交** | `application/x-www-form-urlencoded` | 类似 HTML `<form>` 提交，键值对形式 |
| **文件上传** | `multipart/form-data` | **注意**：通常不需要手动设置，勾选 JMeter "Use multipart/form-data" 即可自动处理 |
| **XML 接口** | `application/xml` 或 `text/xml` | 旧式 SOAP 或 XML API |

### 4.2 认证鉴权类 (Authorization)
告诉服务端**你是谁**。

| 场景 | Header 值示例 | 说明 |
| :--- | :--- | :--- |
| **Token 认证** (JWT/OAuth) | `Bearer <your_token_string>` | 最通用的现代 API 认证方式 |
| **Basic 认证** | `Basic <base64_user_pass>` | 较旧，用户名密码 Base64 编码 |
| **自定义 Key** | `X-API-KEY: abc12345` | 部分服务商特定的 Header 名称 |

### 4.3 客户端协商类
告诉服务端**你想要什么**或**你用什么设备**。

| Header 名称 | 常见值 | 说明 |
| :--- | :--- | :--- |
| **Accept** | `application/json` | 期望服务端返回 JSON 格式数据 |
| **User-Agent** | `Mozilla/5.0 ...` | 模拟浏览器（Chrome/Safari）或手机（Android/iOS），防止被识别为爬虫/机器人 |
| **Referer** | `https://www.example.com/login` | 来源页面，防盗链检查常需要此字段 |

### 4.4 必备 Header 清单 (Checklist)

#### 针对 REST API 接口测试：
- [x] **Content-Type**: `application/json` (如果是 POST/PUT)
- [x] **Authorization**: `Bearer ...` (如果需要登录)
- [ ] **Accept**: `application/json` (推荐加上)

#### 针对 模拟浏览器行为：
- [x] **User-Agent**: 必须模拟真实浏览器字符串
- [x] **Referer/Origin**: 涉及跨域或安全检查时必须
- [x] **Cookie**: 详见下方说明（推荐使用 Cookie Manager，但也支持手动添加）

### 4.5 特殊 Header：Cookie 的处理
虽然 Cookie 本质上也是 Header (`Cookie: name=value`)，但 JMeter 提供了两种处理方式：

#### 方式一：使用 HTTP Cookie Manager (强烈推荐)
这是最接近真实浏览器行为的方式。
*   **自动维护**：自动解析服务端返回的 `Set-Cookie` 并在后续请求中携带，无需人工干预。
*   **手动添加**：支持在组件界面下方 "User-Defined Cookies" 表格中添加自定义 Cookie（如预埋 Token）。

#### 方式二：使用 HTTP Header Manager (手动强注)
*   **操作**：直接添加 Name=`Cookie`, Value=`session_id=xyz123`。
*   **特点**：简单粗暴，强制生效。
*   **缺点**：**不会**自动更新服务端下发的新 Cookie，**不会**自动维护 Session 上下文。仅适用于简单的单次请求调试或固定 Cookie 场景。

