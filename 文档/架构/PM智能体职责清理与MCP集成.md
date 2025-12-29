# PM 智能体职责清理与 MCP 集成 (PM Agent Responsibility Cleanup)

## 问题梳理

### 1. 与现有行为模式 (MODE) 的功能重叠

**`MODE_Task_Management.md` 与 `pm-agent.md` 存在严重的功能重叠**：
- 两者都在进行内存读写操作（`write_memory()` / `read_memory()`）。
- 依赖项冲突（Serena MCP vs 本地文件）。
- 都在进行并行读取和 TodoWrite 管理。
- **结论**：功能完全重叠，必须进行整合。

### 2. 内存 (Memory) 管理职责不明确
- `docs/memory/` 目录下的文件缺乏清理规则，文件库与 MCP 内存的界限模糊。
- Serena MCP 的 `write_memory()` 何时使用、何时删除并不明确。

### 3. MCP 角色分工模糊
- **Serena**：应专注于代码理解。
- **Mindbase**：应专注于对话历史的长期保存。
- 目前现状是各组件职责混杂。

## 解决方案：明确职责划分

### 内存管理策略 (Memory Management Strategy)

1. **第一层：会话记忆 (Mindbase MCP)**
   - **用途**：长期保存对话历史（Claude Code 标准功能）。
   - **技术**：Mindbase MCP。
   - **生命周期**：持久化（自动管理）。
   - **场景**：搜索过去对话、长期模式学习、跨项目知识共享。

2. **第二层：项目文档 (基于文件)**
   - **用途**：项目特有的知识库。
   - **技术**：`docs/` 下的 Markdown 文件。
   - **生命周期**：Git 管理（直到手动删除前保持持久）。
   - **位置**：成功模式 (`docs/patterns/`)、失败记录 (`docs/mistakes/`)、全局规则 (`CLAUDE.md`)。

3. **第三层：任务状态 (Serena MCP)**
   - **用途**：辅助代码库理解的符号管理。
   - **技术**：Serena MCP。
   - **生命周期**：会话结束自动删除。
   - **场景**：理解代码结构、追踪符号关系、重构支持。

4. **第四层：TodoWrite (内置功能)**
   - **用途**：管理当前任务进度。
   - **技术**：Claude Code 标准功能。
   - **生命周期**：任务完成即删除。

### MCP 角色明确化
- **Mindbase MCP (对话历史)**：由 Claude Code 自动管理，透明运行。
- **Serena MCP (代码理解)**：仅在需要分析代码结构、追踪符号或重构时自动激活。
- **Sequential MCP (复杂推理)**：在指挥官模式下，用于架构设计、深度调试或系统分析。
- **Context7 MCP (文档引用)**：用于在实施前的信心检查阶段查阅官方库文档或 API 规范。

## 整合后的 PM 智能体架构

### 应删除的内容
- ❌ **`docs/memory/` 整个目录**：其功能与 Mindbase 重复，且生命周期不明确。
- ❌ **`MODE_Task_Management.md` 中的内存操作部分**：与 `pm-agent.md` 重复。
- ❌ **`pm-agent.md` 对 `docs/memory/` 的引用**：一律整合至 Mindbase。
- ❌ **滥用 `write_memory()` / `read_memory()`**：Serena 应专属于代码理解。

### 整合后的职责
1. **会话生命周期管理**：启动时确认 Git 状态并加载 `CLAUDE.md` 及最近模式；结束时更新文档并由 Mindbase 自动保存。
2. **文档守护者**：实施前强制读取模式和失败记录，并进行信心检查。
3. **指挥官 (处理复杂任务)**：利用 TodoWrite 管理任务，并调用 Sequential 进行深度分析。
4. **实施后文档化**：将成功经验归纳为模式，将失败教训记录为错误文档。
5. **错误处理程序 (反思机制)**：自动搜索并应用历史修复方案，并记录新错误的成因。

## 益处

- **简洁性**：管理层级分明（Mindbase / 文件 / TodoWrite）；MCP 角色明确（代码 / 推理 / 文档）；移除了冗余的目录和逻辑。
- **可维护性**：生命周期清晰（持久 vs 临时）；职责分离（对话归 Mindbase，知识归文件，进度归 TodoWrite）。
- **高效性**：各司其职，仅在必要时读取文件或调用相关的 MCP。

## 结论

**删除**：`docs/memory/` 全体、无节制的 `write_memory()` 调用、`MODE_Task_Management.md` 的冗余部分。
**整合**：Mindbase (对话) + `docs/` (知识) + TodoWrite (进度) + Serena (代码)。
**简化**：通过明确职责，消除不必要的复杂度。

这样，PM 智能体将变得更加简洁且强大。
