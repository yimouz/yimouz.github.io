<script src="https://cdn.bootcdn.net/ajax/libs/mermaid/10.9.0/mermaid.min.js"></script>

<div class="mermaid" style="text-align: center;">
graph TD
    A[监控到TPS忽高忽低] --> B{查看GC日志}
    B -- 频繁Full GC --> C[优化JVM堆内存]
    B -- GC正常 --> D[检查数据库连接池]
</div>

<script>
  if (window.mermaid) {
    mermaid.initialize({ startOnLoad: true, theme: 'default' });
  }
</script>
