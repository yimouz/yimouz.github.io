---
title: 压测 TPS 波动排查逻辑test
date: 2026-04-01
mermaid: true
---

```mermaid
graph TD
    A[监控到TPS忽高忽低] --> B{查看GC日志}
    B -- 频繁Full GC --> C[优化JVM堆内存]
    B -- GC正常 --> D[检查数据库连接池]
