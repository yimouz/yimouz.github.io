# Clash 自定义脚本：将域名绑定到策略组

## 功能说明

脚本用于在 Clash 订阅配置中，示例指定域名如（`anyrouter.top`）自动绑定到当前订阅中的手动选择策略组。        

## 执行流程

1. **深度嗅探**：在配置中查找第一个 `type === 'select'` 的策略组（通常是机场订阅中存放节点的手动选择组）。
2. **确定目标**：
   - 若找到 select 组，使用其名称；
   - 否则查找名为 `Proxy` 的组；
   - 最终兜底为 `DIRECT`。
3. **定义规则**：构建自定义规则列表。
4. **注入规则**：将自定义规则插入到原规则列表的最前面，确保优先匹配。
5. **调试日志**：在软件的 Logs 中输出当前绑定的策略组名称。

## 代码

```javascript
function main(config) {
  // 1. 深度嗅探：找到当前订阅中第一个类型为 'select' 的策略组
  // 这是大多数机场订阅中存放节点的手动选择组
  const proxyGroup = config['proxy-groups'].find(
    (g) => g.type === 'select' && g.name !== 'GLOBAL'
  );

  // 2. 确定目标：如果找到了就用它，找不到就找名为 'Proxy' 的组，最后兜底用 'DIRECT'
  const target = proxyGroup
    ? proxyGroup.name
    : (config['proxy-groups'].find(g => g.name.toLowerCase() === 'proxy')?.name || 'DIRECT');

  // 3. 定义你的自定义规则
  const myRules = [
    `DOMAIN-SUFFIX,anyrouter.top,${target}`,
    // 你可以在这里继续添加其他全局规则
  ];

  // 4. 注入规则：确保你的规则排在最前面
  config.rules = [...myRules, ...config.rules];

  // 5. 调试日志：你可以在软件的 Logs 里看到当前规则到底绑定到了哪个组
  console.log(`[Custom Script] anyrouter.top 已成功绑定到策略组: ${target}`);

  return config;
}
```

## 使用方法

配置支持 JavaScript 自定义脚本的 Clash 客户端（如 Clash Verge、Stash 等），订阅加载时会自动执行 `main` 函数处理配置。

## 自定义扩展

如添加更多规则，可在 `myRules` 数组中按以下格式追加：

```javascript
const myRules = [
  `DOMAIN-SUFFIX,anyrouter.top,${target}`,
  `DOMAIN-SUFFIX,example.com,${target}`,
  `DOMAIN-KEYWORD,google,${target}`,
];
```
