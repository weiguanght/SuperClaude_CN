# MCP 集成策略 (MCP Integration Policy)

SuperClaude 框架中 MCP (Model Context Protocol) 服务器的集成策略与使用指南。

---

## MCP 服务器定义

### 核心 MCP 服务器

#### 记忆与错误学习

**ReflexionMemory (内置，始终可用)**
```yaml
名称: ReflexionMemory
用途: 错误历史存储与学习
类别: 记忆管理 (内置)
自动管理: 是 (内部实现)
PM 智能体角色: 出错时自动使用

能力:
  - 记录过去发生的错误及解决方案
  - 基于关键词搜索相似错误
  - 通过学习防止错误再次发生
  - 项目范围的记忆

实现:
  位置: superclaude/core/pm_init/reflexion_memory.py
  存储: docs/memory/reflexion.jsonl (本地文件)
  搜索: 基于关键词 (50% 重合度阈值)

说明: 这是一个内部实现，而非外部 MCP 服务器。
```

**Mindbase MCP (可选增强方案，通过 airis-mcp-gateway 开启)**
```yaml
名称: mindbase
用途: 全量对话历史的语义搜索
类别: 记忆管理 (可选 MCP)
自动管理: 否 (外部 MCP 服务器 —— 需要安装)
PM 智能体角色: 在可用时由 Claude 自动选择

能力:
  - 持久化所有对话历史 (PostgreSQL + pgvector)
  - 语义搜索 (qwen3-embedding:8b)
  - 跨项目知识共享
  - 从所有过去的对话中学习

工具:
  - mindbase_search: 语义搜索
  - mindbase_store: 对话存储
  - mindbase_health: 健康检查

安装:
  需要: 使用 "recommended" 配置文件的 airis-mcp-gateway
  参考: https://github.com/agiletec-inc/airis-mcp-gateway

配置文件依赖:
  - "recommended" (推荐) 配置：包含 mindbase (适用于长期项目)
  - "minimal" (极简) 配置：不包含 mindbase (适用于轻量、快速任务)

使用模式:
  - 已安装且使用推荐配置：Claude 会自动调用
  - 否则：回退到 ReflexionMemory
  - PM 智能体指令：“搜索过去的错误” (Claude 自动选择工具)

说明: 可选增强方案。SuperClaude 仅凭 ReflexionMemory 即可完全正常工作。
```

#### Serena MCP
```yaml
名称: serena
用途: 用于理解代码库的符号管理
类别: 代码理解
自动管理: 否 (需显式使用)
PM 智能体角色: 在代码理解任务中自动活用

能力:
  - 符号追踪 (函数、类、变量)
  - 代码结构分析
  - 重构辅助
  - 依赖关系映射

生命周期:
  启动: 无需操作
  执行中: 在需要理解代码时使用
  结束: 自动卸载 (会话结束)
  清理: 自动

使用模式:
  使用场景:
    - 重构计划制定
    - 代码结构分析
    - 追踪符号间的关系
    - 探索大型代码库

  不适用于:
    - 任务管理
    - 会话记忆
    - 文档保存
    - 项目知识管理

触发条件:
  - 关键词: "refactor", "analyze code structure", "find all usages"
  - 文件数量: 处理文件数 > 10
  - 复杂度: 涉及到跨文件的符号追踪

示例:
  任务：“重构涉及 15 个文件的身份验证系统”
  → Serena: 追踪身份验证相关的符号
  → PM 智能体: 结合 Serena 的洞察协调重构工作
```

#### Sequential MCP
```yaml
名称: sequential-thinking
用途: 复杂的推理与分步分析
类别: 推理引擎
自动管理: 否 (需显式使用)
PM 智能体角色: 在指挥官 (Commander) 模式下分析复杂任务

能力:
  - 阶段式推理
  - 假设验证
  - 复杂问题拆解
  - 系统设计分析

生命周期:
  启动: 无需操作
  执行中: 在进行复杂分析时使用
  结束: 返回分析结果
  清理: 自动

使用模式:
  使用场景:
    - 架构设计
    - 复杂的 Bug 分析
    - 系统设计评审
    - 权衡 (Trade-off) 分析

  不适用于:
    - 简单任务
    - 凭直觉即可解决的问题
    - 代码生成 (仅用于分析)

触发条件:
  - 关键词: "design", "architecture", "analyze tradeoffs"
  - 复杂度: 多组件系统分析
  - 不确定性: 存在多种有效方案时

示例:
  任务：“为身份验证设计微服务架构”
  → Sequential: 进行分步的设计分析
  → PM 智能体: 将设计决策记录到 docs/patterns/ 中
```

#### Context7 MCP
```yaml
名称: context7
用途: 查阅官方文档与库模式 (Patterns)
类别: 文档参考
自动管理: 否 (需显式使用)
PM 智能体角色: 实施前的确信度自检 (Confidence Check)

能力:
  - 搜索官方在线文档
  - 查阅库的最佳实践
  - 核实 API 规范
  - 学习框架模式

生命周期:
  启动: 无需操作
  执行中: 在需要查阅文档时使用
  结束: 返回信息
  清理: 自动

使用模式:
  使用场景:
    - 确认第三方库的使用方法
    - 参考最佳实践
    - 核实 API 规范
    - 学习官方推荐模式

  不适用于:
    - 项目特有的文档 (使用 docs/ 目录)
    - 公司内部文档
    - 自定义实现模式

触发条件:
  - 实施前: 确信度检查时
  - 关键词: "official docs", "best practices", "how to use [library]"
  - 新库引入: 第一次使用某个库时

示例:
  任务：“使用 jose 库实现 JWT 身份验证”
  → Context7: 获取 jose 的官方文档与模式
  → PM 智能体: 根据官方模式核实实施方案
```

#### Tavily MCP
```yaml
名称: tavily
用途: 网页搜索与实时信息获取
类别: 研究调查
自动管理: 否 (需显式使用)
PM 智能体角色: 在研究 (Research) 模式下收集信息

能力:
  - 网页搜索
  - 获取最新信息
  - 搜索技术文章
  - 搜索报错信息的解决方案

生命周期:
  启动: 无需操作
  执行中: 在研究或调查时使用
  结束: 返回搜索结果
  清理: 自动

使用模式:
  使用场景:
    - 核实库的最新版本
    - 搜索报错信息的解决对策
    - 调研技术趋势
    - 查阅官方文档 (当 Context7 中缺失时)

  不适用于:
    - 搜索项目内部信息 (使用 Grep)
    - 代码库搜索 (使用 Serena)
    - 搜索过去对话 (使用 Mindbase)

触发条件:
  - 关键词: "search", "latest", "current"
  - 报错: 遇到未知的错误消息
  - 研究: 调查新技术

示例:
  任务：“寻找 Next.js 15 App Router 的最新模式”
  → Tavily: 在网页上搜索最新模式
  → PM 智能体: 将发现结果记录到 docs/patterns/ 中
```

---

## MCP 选择矩阵

### 按任务类型分类

```yaml
代码理解:
  首选: Serena MCP
  次选: Grep (用于简单搜索)
  示例: "查找所有与身份验证相关的符号"

复杂分析:
  首选: Sequential MCP
  次选: 原生推理 (简单场景)
  示例: "设计身份验证架构"

文档参考:
  首选: Context7 MCP
  次选: Tavily (如果 Context7 中不存在)
  示例: "如何使用 React Server Components"

研究与调查:
  首选: Tavily MCP
  次选: Context7 (查阅官方文档)
  示例: "2025 年最新的安全最佳实践"

记忆与历史:
  首选: Mindbase MCP (自动执行)
  次选: 无 (完全自动化管理)
  示例: 不适用 (过程透明)

任务管理:
  首选: TodoWrite (内置)
  次选: 无
  示例: 追踪多步骤的实施进度
```

### 按复杂度分类

```yaml
简单 (修改 1-2 个文件，路径清晰):
  MCP: 无 (原生工具已足够)
  工具: Read, Edit, Grep, Bash

中等 (3-10 个文件，有一定复杂度):
  MCP: Context7 (如果是新库)
  工具: MultiEdit, Glob, Grep

复杂 (>10 个文件，涉及架构变更):
  MCP: Serena + Sequential
  协调: PM 智能体指挥官模式
  工具: 任务委派、并行执行引擎

研究 (信息采集):
  MCP: Tavily + Context7
  模式: 深度研究模式 (DeepResearch)
  工具: WebFetch (有针对性地使用)
```

---

## PM 智能体集成准则

### 会话生命周期

```yaml
会话启动:
  自动执行:
    1. git status && git branch
    2. 读取 CLAUDE.md
    3. 读取最新的 5 份 docs/patterns/*.md
    4. Mindbase 自动加载
  使用 MCP:
    - Mindbase: 自动调用 (无需显式指令)
    - 其他: 无 (等待具体任务)
  输出形式: 📍 [分支] | [状态] | 🧠 [token]%

实施前:
  自动执行:
    1. 读取相关的 docs/patterns/
    2. 读取相关的 docs/mistakes/
    3. 确信度检查 (Confidence Check)
  使用 MCP:
    - Context7: 如果引入了新库 (自动触发)
    - Serena: 如果涉及复杂重构 (自动触发)
  决策逻辑:
    高确信度 (>90%): 开始实施
    中确信度 (70-89%): 给用户提供多个选项
    低确信度 (<70%): 停止执行，请求用户澄清

实施中:
  手动/条件触发:
    - TodoWrite: 进度追踪
    - Serena: 代码理解 (需要时)
    - Sequential: 复杂分析 (需要时)
  使用 MCP:
    - Serena: 触发代码复杂度预警时
    - Sequential: 出现分析类关键词时
    - Context7: 出现文档需求时

实施后:
  自动执行:
    1. 自我评估 (四个核心问题测试)
    2. 提炼模式 (Patterns)
    3. 更新文档
  使用 MCP:
    - Mindbase: 自动保存
    - 其他: 无 (基于文件的文档化)
  输出产物:
    - 成功场景 → 更新 docs/patterns/
    - 失败场景 → 更新 docs/mistakes/
    - 全局规则 → 更新 CLAUDE.md
```

### MCP 激活诱导词

```yaml
Serena MCP:
  自动触发词:
    - "refactor" (重构)
    - "analyze code structure" (分析代码结构)
    - "find all usages" (查找引用)
    - "symbol tracking" (符号追踪)
  自动触发条件:
    - 文件数 > 10
    - 跨文件变更
    - 符号重命名
    - 依赖关系分析
  手动覆盖: --serena 标志位

Sequential MCP:
  自动触发词:
    - "design" (设计)
    - "architecture" (架构)
    - "analyze tradeoffs" (权衡分析)
    - "complex problem" (复杂问题)
  自动触发条件:
    - 系统设计任务
    - 存在多个候选方案
    - 实施路径不明
    - 架构决策点
  手动覆盖: --seq 标志位

Context7 MCP:
  自动触发词:
    - "official docs" (官方文档)
    - "best practices" (最佳实践)
    - "how to use [library]" (如何使用某库)
    - 检测到新库引入
  自动触发条件:
    - 实施前的确信度检查
    - package.json 中新增了库
    - 寻找框架模式
  手动覆盖: --c7 标志位

Tavily MCP:
  自动触发词:
    - "search" (搜索)
    - "latest" (最新)
    - "current trends" (当前趋势)
    - "find error solution" (寻找报错对策)
  自动触发条件:
    - 激活研究模式 (Research Mode)
    - 遇到未知的报错信息
    - 版本查证
  手动覆盖: --tavily 标志位
```

---

## 反面模式 (禁止事项)

```yaml
❌ 显式操作 Mindbase:
  原因: 完全自动化管理，PM 智能体不应手动干预。
  对策: 无需操作（它会自动运行）。

❌ 将 Serena 用于任务管理:
  原因: Serena 仅用于代码理解。
  对策: 使用 TodoWrite。

❌ 在代码理解中以外使用 write_memory() / read_memory():
  原因: Serena 并非通用的任务/对话管理工具。
  对策: 使用 TodoWrite + docs/。

❌ 创建 docs/memory/ 目录:
  原因: 与 Mindbase 的自动持久化功能重叠。
  对策: 使用 docs/patterns/ 和 docs/mistakes/。

❌ 在所有任务中使用 Sequential:
  原因: 浪费 Token。
  对策: 仅用于复杂的分析场景。

❌ 将 Context7 用于项目特定的内部文档:
  原因: 它专注于外部官方文档。
  对策: 读取 docs/ 目录。
```

---

## 最佳实践

```yaml
✅ 选对工具做对事:
  简单场景 → 原生工具 (Read, Edit, Grep)
  中等场景 → Context7 (涉及新库时)
  复杂场景 → Serena + Sequential

✅ 延迟激活 (Lazy Evaluation):
  不要预加载所有 MCP。
  仅在需要时激活。
  让 PM 智能体根据逻辑自动触发。

✅ 职责分离:
  对话记忆: Mindbase (全自动)
  知识积累: docs/ (基于文件)
  进度追踪: TodoWrite (基于会话)
  代码洞察: Serena (用于理解)

✅ 文档优先:
  实施前: 参考 Context7 + docs/patterns/。
  执行中: 使用 TodoWrite 进行进度追踪。
  执行后: 及时更新 docs/patterns/ 或 docs/mistakes/。
```
