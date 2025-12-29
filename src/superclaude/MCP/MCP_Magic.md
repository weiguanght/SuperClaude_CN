# Magic MCP 服务器 (Magic MCP Server)

**用途**：利用 21st.dev 的设计模式生成集成了设计系统的现代 UI 组件。

## 触发场景
- 请求 UI 组件：按钮 (Button)、表单 (Form)、弹窗 (Modal)、卡片 (Card)、表格 (Table)、导航 (Nav) 等。
- 设计系统实施需求。
- 使用 `/ui` 或 `/21` 命令。
- 前端特定关键词：响应式 (Responsive)、无障碍性 (Accessible)、交互式 (Interactive)。
- 组件增强或精细化调整请求。

## 何时选择
- **生成 UI 组件时**：请使用 Magic，而非原生 HTML/CSS 生成。
- **优于手动编码**：当您需要生产就绪且符合无障碍标准的组件时。
- **针对设计系统**：当保持与现有模式的连贯性至关重要时。
- **针对现代框架**：遵循当前最佳实践的 React, Vue, Angular 代码生成。
- **不适用于后端任务**：API 逻辑、数据库查询、服务器配置。

## 最佳协同组合
- **Context7**：Magic 提供来自 21st.dev 的 UI 模式 → Context7 提供框架集成支持。
- **Sequential (顺序推理)**：由 Sequential 分析 UI 需求 → 由 Magic 实施结构化组件。

## 示例
```
"创建一个登录表单" → Magic (UI 组件生成)
"构建一个响应式的导航栏" → Magic (具备无障碍性的 UI 模式)
"添加一个带排序功能的数据报表" → Magic (复杂的 UI 组件)
"让这个组件符合无障碍标准" → Magic (UI 增强)
"编写一个 REST API" → 原生 Claude (后端逻辑)
"修复数据库查询" → 原生 Claude (非 UI 任务)
```
