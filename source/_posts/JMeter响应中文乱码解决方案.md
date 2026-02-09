---
title: JMeter响应中文乱码解决方案
date: 2021-06-08
tags: [JMeter, 乱码处理, 性能测试]
categories: [JMeter]
summary: 解决JMeter响应数据中文乱码的四种方法：修改配置文件、强制编码、BeanShell后置处理等。
---

# JMeter 响应中文乱码解决方案

JMeter 响应数据出现中文乱码，通常是因为 JMeter 默认使用 `ISO-8859-1` 编码，而接口返回的数据是 `UTF-8` 编码。

以下是几种常用的解决方案，按**推荐程度**排序：

### 1. 修改配置文件（全局生效，最推荐）
这是最彻底的解决方法，修改一次后对所有脚本生效。

1.  进入 JMeter 安装目录的 `bin` 文件夹。
2.  找到并打开 `jmeter.properties` 文件。
3.  搜索关键字 `sampleresult.default.encoding`。
4.  找到该行，去掉行首的 `#` 号，并将值修改为 `UTF-8`：
    ```properties
    # 修改前
    #sampleresult.default.encoding=ISO-8859-1

    # 修改后
    sampleresult.default.encoding=UTF-8
    ```
5.  **保存文件并重启 JMeter**。

---

### 2. 在 HTTP 请求中指定编码（局部生效）
如果你没有权限修改配置文件，或者只想针对某个特定接口生效：

1.  在 JMeter 界面中选中你的 **HTTP Request (HTTP 请求)**。
2.  找到 **Content encoding (内容编码)** 字段。
3.  手动输入 `utf-8`（不区分大小写）。

---

### 3. 使用 BeanShell 后置处理器（脚本强制转换）
如果以上方法都无效，可以使用脚本强制修改响应数据的编码：

1.  在对应的 HTTP 请求节点上右键，选择 **Add (添加)** -> **Post Processors (后置处理器)** -> **BeanShell PostProcessor**。
2.  在 **Script** 区域输入以下代码：
    ```java
    prev.setDataEncoding("UTF-8");
    ```

---

### 4. 针对请求体（Body）乱码
如果是你发送给服务器的中文变成乱码（而不是接收乱码），请检查：
*   **HTTP 请求头**：确保添加了 `Content-Type: application/json;charset=UTF-8`。
*   **勾选编码**：在 HTTP 请求的 Parameters 中，勾选 "URL Encode"。
