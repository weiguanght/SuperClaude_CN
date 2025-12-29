# PM 智能体模式集成指南 (Integration Guide)

**最后更新**：2025-10-14  
**目标版本**：4.2.0  
**状态**：实施指南

---

## 📋 概览

本指南提供了将 PM 智能体模式集成为 SuperClaude 常驻元层 (Meta-Layer) 的分步流程，包括会话生命周期管理、PDCA 自我评估以及系统化的知识管理。

---

## 🎯 集成目标

1. **会话生命周期**: 启动时自动激活并恢复上下文。
2. **PDCA 引擎**: 自动化执行“计划-执行-检查-行动”循环。
3. **记忆操作**: 集成 Serena MCP 以实现会话持久化。
4. **文档策略**: 系统化地演进项目知识。

---

## 📐 架构集成

### PM 智能体定位

```
┌──────────────────────────────────────────┐
│    PM 智能体模式 (元层)                  │
│    • 始终激活                            │
│    • 会话管理                            │
│    • PDCA 自我评估                       │
└──────────────┬───────────────────────────┘
               ↓
    [专家智能体层]
               ↓
    [命令与模式层]
               ↓
    [MCP 工具层]
```

详情请参阅：[ARCHITECTURE.md](./ARCHITECTURE.md)

---

## 🔧 第二阶段：核心实施

### 文件结构

```
superclaude/
├── Commands/
│   └── pm.md                           # ✅ 已更新
├── Agents/
│   └── pm-agent.md                     # ✅ 已更新
└── Core/
    ├── __init__.py                     # 模块初始化
    ├── session_lifecycle.py            # 🆕 会话管理
    ├── pdca_engine.py                  # 🆕 PDCA 自动化
    └── memory_ops.py                   # 🆕 记忆操作
```

### 实施顺序

1. `memory_ops.py` —— Serena MCP 封装（基础）
2. `session_lifecycle.py` —— 会话管理（依赖于 memory_ops）
3. `pdca_engine.py` —— PDCA 自动化（依赖于 memory_ops）

---

## 1️⃣ memory_ops.py 实施项

### 目的
封装 Serena MCP 的记忆操作，包含错误处理与回退方案。

### 关键函数

```python
class MemoryOperations:
    """Serena MCP 记忆操作封装"""

    def list_memories() -> List[str]:
        """列出所有可用记忆"""

    def read_memory(key: str) -> Optional[Dict]:
        """按键读取记忆"""

    def write_memory(key: str, value: Dict) -> bool:
        """按键写入记忆"""

    def delete_memory(key: str) -> bool:
        """按键删除记忆"""
```

---

## 2️⃣ session_lifecycle.py 实施项

### 目的
在会话开始时自动激活、恢复上下文并生成用户报告。

### 关键内容

- **会话启动钩子**: 调用 `list_memories` 并读取 `pm_context`, `last_session`, `next_actions`。
- **用户报告生成**: 按照“前回/进展/本次/课题”的格式汇报状态。
- **会话结束钩子**: 保存检查点，写入总结与待办事项。

---

## 3️⃣ pdca_engine.py 实施项

### 目的
通过自动化手段执行 PDCA 循环，并自动生成文档。

### 阶段逻辑

- **Plan (计划)**: 生成假说文档 (`docs/temp/hypothesis-*.md`) 并写入记忆。
- **Do (执行)**: 联动 TodoWrite 进行追踪，每 30 分钟保存一次进度检查点，更新日志文档。
- **Check (检查)**: 触发任务符合性与完成度的自我评估思考，生成教训文档。
- **Act (行动)**: 成功则沉淀为 `docs/patterns/`，失败则记录到 `docs/mistakes/`。

---

## 🔌 第三阶段：Serena MCP 集成

### 配置示例
在 `~/.claude/.claude.json` 中添加 Serena 服务器：
```json
{
  "mcpServers": {
    "serena": {
      "command": "uv",
      "args": ["run", "serena-mcp"]
    }
  }
}
```

---

## 📁 第四阶段：文档策略

### 目录结构规划
- `docs/temp/`: 临时文档（7 天生命周期）。
- `docs/patterns/`: 正式模式（永久保存）。
- `docs/mistakes/`: 错误记录（永久保存）。

### 自动化操作
- 创建清理脚本定期删除过期临时文件。
- 创建迁移脚本将成功的实验转为正式模式。

---

## 🚀 第五阶段：自动激活 (需研究)

研究 Claude Code 的初始化钩子，实现无感自动激活 PM 智能体。

---

## ✅ 实施检查清单

- [ ] 实施 `memory_ops.py` 并编写单元测试
- [ ] 实施 `session_lifecycle.py` 并编写单元测试
- [ ] 实施 `pdca_engine.py` 并编写单元测试
- [ ] 安装并配置 Serena MCP 服务器并验证持久性
- [ ] 创建文档目录模板及自动清理脚本

---

**最后验证**: 2025-10-14  
**下一次审查**: 2025-10-21  
**版本**: 4.1.5
