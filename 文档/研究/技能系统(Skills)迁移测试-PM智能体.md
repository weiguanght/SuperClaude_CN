# 技能系统 (Skills) 迁移测试 - PM 智能体 (Skills Migration Test)

**日期**：2025-10-21
**目标**：验证 PM 智能体在迁移至技能系统后，是否实现了“零足迹 (Zero-footprint)”及按需加载功能。

---

## 试验设置

### 迁移前（现状）
- `~/.claude/superclaude/agents/pm-agent.md`：始终加载（约 2500 Token）。
- **启动影响**：Claude Code 启动时会自动读取所有相关 Markdown 文件。

### 迁移后
- `~/.claude/skills/pm/SKILL.md`：仅包含描述（约 50 Token）。
- `~/.claude/skills/pm/implementation.md`：包含完整逻辑（仅在 `/sc:pm` 调用时加载）。
- **启动影响**：Claude Code 启动时基本不占用 Token。

---

## 预期结果

### 启动 Token 消耗
- **迁移前**：~2,500 Token。
- **迁移后**：**0 Token** (技能在启动时不被加载)。
- **节省率**：**100%**。

### 使用 `/sc:pm` 时
- 加载描述 + 实现逻辑：约 2,550 Token。
- **后续调用**：由系统缓存，不再重复消耗。

### 净收益
- **不使用 PM 的会话**：节省 2,500 Token。
- **使用 PM 的会话**：增加 50 Token 指令开销 (约 2% 的开销增加)。
- **盈亏平衡点**：只要有超过 2% 的会话不使用 PM 功能，整体就是净正向收益。

---

## 测试规程

1. **备份当前状态**：`cp -r ~/.claude/superclaude ~/.claude/superclaude.backup`。
2. **创建技能结构**：建立 `SKILL.md` 和 `implementation.md` 分层文件。
3. **更新斜杠命令**：将 `/sc:pm` 指向新的 `skill: pm`。
4. **执行测试**：
   - 测试 1：不调用命令启动，通过系统通知验证没有加载 PM 智能体。
   - 测试 2：执行 `/sc:pm`，验证技能是否按需加载且功能完整。
   - 测试 3：多次会话，验证缓存机制是否生效。

---

## 成功基准
- ✅ 启动 Token 为 0。
- ✅ `/sc:pm` 能够成功唤起 2,500+ Token 的完整逻辑。
- ✅ 100% 保留原有功能。
- ✅ 在不使用 PM 的会话中，Token 节省率 > 90%。

---

## 回滚计划
如果迁移失败，删除 `~/.claude/skills/pm` 并从备份还原 `~/.claude/superclaude`。

---

## 下一步行动
若测试成功，将全面推进其他智能体（任务、研究等）以及模式（编排、头脑风暴等）的技能系统化迁移。
