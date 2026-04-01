<script src="https://unpkg.com/mermaid@10.9.0/dist/mermaid.min.js"></script>

<pre class="mermaid">
graph TD
    A[监控到TPS忽高忽低] --> B{查看GC日志}
    B -- 频繁Full GC --> C[优化JVM堆内存]
    B -- GC正常 --> D[检查数据库连接池]
</pre>

<script>
  setTimeout(() => {
    if (window.mermaid) {
      mermaid.initialize({ startOnLoad: true, theme: 'neutral' });
      mermaid.init();
    }
  }, 500);
</script>
