---
title: JMeter-prev
date: 2024-12-03 11:10:30
tags:
  - JMeter
categories: 性能笔记
author: 是啊蓁阿
summary: "`prev` 对象是一个非常重要的变量，通常用于在 JSR223 Sampler 或者 BeanShell Sampler 中获取上一个 Sampler 的结果"
---



 JMeter 中，`prev` 对象是一个非常重要的变量，通常用于在 JSR223 Sampler 或者 BeanShell Sampler 中获取上一个 Sampler 的结果。记录下 `prev` 对象的常用 API。

### 什么是 `prev` 对象？

`prev` 对象是一个 `SampleResult` 类型的实例，它包含了上一个 Sampler 的所有信息和结果。通过使用 `prev` 对象，我们可以访问上一个请求的响应数据、响应时间、响应代码等信息。

### 常用的 `prev` API

下面是 `prev` 对象的一些常用 API 和属性：

1. **getTime()**
   - 返回采样器的响应时间（以毫秒为单位）。
   ```java
   long responseTime = prev.getTime();
   ```

2. **getResponseCode()**
   - 返回采样器的响应代码（通常是 HTTP 状态码）。
   ```java
   String responseCode = prev.getResponseCode();
   ```

3. **getResponseMessage()**
   - 返回采样器的响应消息。
   ```java
   String responseMessage = prev.getResponseMessage();
   ```

4. **getResponseDataAsString()**
   - 以字符串形式返回响应数据。
   ```java
   String responseData = prev.getResponseDataAsString();
   ```

5. **getResponseHeaders()**
   - 返回响应头信息。
   ```java
   String responseHeaders = prev.getResponseHeaders();
   ```

6. **getRequestHeaders()**
   - 返回请求头信息。
   ```java
   String requestHeaders = prev.getRequestHeaders();
   ```

7. **getSamplerData()**
   - 返回采样器的请求数据。
   ```java
   String samplerData = prev.getSamplerData();
   ```

8. **getSampleLabel()**
   - 返回采样器的标签。
   ```java
   String sampleLabel = prev.getSampleLabel();
   ```

9. **isSuccessful()**
   - 返回采样器的执行结果是否成功。
   ```java
   boolean isSuccess = prev.isSuccessful();
   ```

10. **getURL()**
    - 返回采样器的请求 URL。
    ```java
    String url = prev.getURL();
    ```

11. **getStartTime()**
    - 返回采样器开始时间（以毫秒为单位）。
    ```java
    long startTime = prev.getStartTime();
    ```

12. **getEndTime()**
    - 返回采样器结束时间（以毫秒为单位）。
    ```java
    long endTime = prev.getEndTime();
    ```

### 使用示例

假设我们在测试一个 RESTful API，并且需要在每个请求之后检查响应数据是否包含特定的字符串。我们可以使用 JSR223 Sampler 来实现这个功能。以下是一个示例脚本：

```java
// 获取上一个采样器的响应数据
String responseData = prev.getResponseDataAsString();

// 检查响应数据是否包含特定字符串
if (responseData.contains("code")) {
    // 如果包含，记录日志并继续
    log.info("Response contains the expected string.");
} else {
    // 如果不包含，记录日志并标记测试失败
    log.error("Response does not contain the expected string.");
    AssertionResult.setFailure(true);
    AssertionResult.setFailureMessage("Response does not contain the expected string.");
}
```


就算是随便记录也希望能够帮助我们更好理解，不要健忘。