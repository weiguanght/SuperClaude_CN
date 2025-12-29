# Morphllm MCP 服务器 (Morphllm MCP Server)

**用途**：基于模式的代码编辑引擎，针对批量转换进行了 Token 优化。

## 触发场景
- 需要一致模式的多文件编辑操作。
- 框架更新、代码规范强制执行、代码清理。
- 跨多个文件的批量文本替换。
- 针对特定范围的自然语言编辑指令。
- 需要优化 Token 消耗（效率提升 30-50%）。

## 何时选择
- **优于 Serena**：执行基于模式的编辑，而非符号 (Symbol) 操作。
- **针对批量操作**：规范执行、框架升级、文本替换。
- **当 Token 效率至关重要时**：追求快速应用 (Fast Apply) 及压缩需求的场景。
- **针对中低复杂度任务**：少于 10 个文件、转换逻辑清晰。
- **不适用于语义操作**：符号重命名、依赖追踪、LSP 集成。

## 最佳协同组合
- **Serena**：由 Serena 分析语义上下文 → 由 Morphllm 执行精确编辑。
- **Sequential (顺序推理)**：由 Sequential 规划编辑策略 → 由 Morphllm 应用系统性变更。

## 示例
```
"将所有 React 类组件更新为 Hooks" → Morphllm (模式转换)
"在整个项目中强制执行 ESLint 规则" → Morphllm (代码规范应用)
"将所有的 console.log 替换为记录器 (logger) 调用" → Morphllm (批量文本替换)
"在所有地方重命名 getUserData 函数" → Serena (符号操作)
"分析代码架构" → Sequential (复杂分析)
"解释此算法" → 原生 Claude (基础解释)
```
