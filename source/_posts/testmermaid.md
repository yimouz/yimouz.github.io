```mermaid
graph TD
    A[监控到TPS忽高忽低] --> B{查看GC日志}
    B -- 频繁Full GC --> C[优化JVM堆内存/回收器]
    B -- GC正常 --> D{检查数据库连接池}
    D -- 连接耗尽 --> E[增加连接池大小或优化SQL]
    D -- 连接充足 --> F[检查网络丢包/带宽限制]
