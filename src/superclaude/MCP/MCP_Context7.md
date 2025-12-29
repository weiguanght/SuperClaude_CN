# Context7 MCP 服务器 (Context7 MCP Server)

**用途**：查询官方库文档及框架模式指南。

## 触发场景
- 出现导入语句：`import`, `require`, `from`, `use` 等。
- 涉及框架关键词：React, Vue, Angular, Next.js, Express 等。
- 关于 API 或最佳实践的特定库问题。
- 需要官方文档定义的模式而非通用解决方案。
- 特定版本的实施要求。

## 何时选择
- **优于网页搜索 (WebSearch)**：当您需要经过筛选的、特定版本的官方文档时。
- **优于原生知识**：当实施过程必须严格遵循官方模式时。
- **针对框架**：React Hooks, Vue Composition API, Angular 服务等。
- **针对库**：正确的 API 用法、身份验证流程、配置项等。
- **针对合规性**：当强制要求遵守官方标准时。

## 最佳协同组合
- **Sequential (顺序推理)**：由 Context7 提供文档 → 由 Sequential 分析实施策略。
- **Magic**：由 Context7 提供模式 → 由 Magic 生成符合框架规范的组件。

## 示例
```
"实现 React useEffect" → Context7 (官方 React 模式)
"使用 Auth0 添加身份验证" → Context7 (官方 Auth0 文档)
"迁移到 Vue 3" → Context7 (官方迁移指南)
"优化 Next.js 性能" → Context7 (官方优化模式)
"解释一下这个函数" → 原生 Claude (无需外部文档)
```
