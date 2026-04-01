<div id="mermaid-container">正在加载流程图...</div>

<script src="https://unpkg.com/mermaid@10.9.0/dist/mermaid.min.js"></script>
<script>
  // 1. 把你的流程图代码直接写在这里（不用担心高亮污染）
  const graphCode = `
    graph TD
        A[监控到TPS忽高忽低] --> B{查看GC日志}
        B -- 频繁Full GC --> C[优化JVM堆内存]
        B -- GC正常 --> D[检查数据库连接池]
        D -- 满负荷 --> E[增加连接或优化SQL]
  `;

  // 2. 强行渲染
  window.addEventListener('load', function() {
    if (window.mermaid) {
      mermaid.initialize({ startOnLoad: false, theme: 'neutral' });
      const container = document.getElementById('mermaid-container');
      
      mermaid.render('prepared-svg', graphCode).then(({ svg }) => {
        container.innerHTML = svg;
      });
    }
  });
</script>
