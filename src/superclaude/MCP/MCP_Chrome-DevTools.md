# Chrome DevTools MCP 服务器 (Chrome DevTools MCP Server)

**用途**：性能分析、调试及实时浏览器审查。

## 触发场景
- 需要进行性能审计与分析。
- 调试布局问题（例如：CLS - 累计布局偏移）。
- 调查加载缓慢的原因（例如：LCP - 最大内容绘制）。
- 分析控制台错误及网络请求。
- 对 DOM 和 CSS 进行实时审查。

## 何时选择
- **进行深度性能分析时**：当您需要深入了解性能瓶颈时使用。
- **进行实时调试时**：审查网页的运行时状态并调试在线问题。
- **进行网络分析时**：检查网络请求并识别诸如 CORS 错误等问题。
- **不适用于端到端 (E2E) 测试**：端到端测试场景请使用 Playwright。
- **不适用于静态分析**：代码审查和逻辑验证请使用 Claude 的原生能力。

## 最佳协同组合
- **Sequential (顺序推理)**：由 Sequential 规划性能改进策略 → 由 Chrome DevTools 分析并验证改进效果。
- **Playwright**：由 Playwright 自动化用户操作流 → 由 Chrome DevTools 分析该操作流的性能。

## 示例
```
"分析此页面的性能" → Chrome DevTools (性能分析)
"为什么这个页面加载这么慢？" → Chrome DevTools (性能分析)
"调试此元素的布局偏移问题" → Chrome DevTools (实时调试)
"检查首页的控制台错误" → Chrome DevTools (实时调试)
"哪些网络请求失败了？" → Chrome DevTools (网络分析)
"测试登录流程" → Playwright (浏览器自动化)
"审查此函数的逻辑" → 原生 Claude (静态分析)
```
