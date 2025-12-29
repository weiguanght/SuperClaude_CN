# PR 文档：将 PM 模式设为默认 - 第一阶段实施 (PR Documentation)

**状态**：✅ 已就绪，待评审
**测试覆盖率**：26 个测试，全部通过
**破坏性变更**：无

---

## 📋 摘要 (Summary)

本 PR 实现了“PM 优先”架构的**第一阶段**：**PM 模式初始化**与**验证基础设施**。

### 本 PR 启用的功能

- ✅ **自动生成上下文合约 (Context Contract)** (针对项目的特定规则)
- ✅ **反思记忆系统 (Reflexion Memory)** (从错误中学习)
- ✅ **5 个核心验证器** (安全性、依赖项、运行时、测试、合约)
- ✅ **四阶段工作流基础** (规划 PLANNING / 任务列表 TASKLIST / 执行 DO / 行动 ACTION)

---

## 🎯 解决的问题

### 变更前
- PM 模式是**可选的**，很少被使用。
- 无法强制执行项目特定的规则（如禁用 .env 文件、强制使用 Kong/Infisical）。
- 同样的错误由于没有学习系统而反复出现。
- 缺乏执行前的验证（导致实现方案违反规则）。

### 变更后
- PM 模式在会话开始时**自动初始化**。
- 上下文合约在执行前**强制执行规则**。
- 反思记忆系统**防止错误再次发生**。
- 验证器在执行前**拦截有问题的代码**。

---

## 🏗️ 架构 (Architecture)

### 1. PM 模式初始化钩子 (Init Hook)

**路径**：`superclaude/core/pm_init/`

```python
from superclaude.core.pm_init import initialize_pm_mode

# 会话开始时自动运行
init_data = initialize_pm_mode()
# 返回：上下文合约 + 反思记忆 + 项目结构
```

**功能**：
- Git 仓库检测。
- 轻量级结构扫描（仅获取路径，不读取内容）。
- 上下文合约自动生成。
- 反思记忆加载。

---

### 2. 上下文合约 (Context Contract)

**路径**：`docs/memory/context-contract.yaml` (自动生成)

**目的**：强制执行项目特定规则。

```yaml
version: 1.0.0
principles:
  use_infisical_only: true
  no_env_files: true
  outbound_through: kong
runtime:
  node:
    manager: pnpm
    source: lockfile-defined
validators:
  - deps_exist_on_registry
  - tests_must_run
  - no_env_file_creation
  - outbound_through_proxy
```

**检测逻辑**：
- 发现 Infisical → `no_env_files: true`
- 发现 Kong → `outbound_through: kong`
- 发现 Traefik → `outbound_through: traefik`
- 发现 pnpm-lock.yaml → `manager: pnpm`

---

### 3. 反思记忆 (Reflexion Memory)

**路径**：`docs/memory/reflexion.jsonl`

**目的**：从错误中学习，防止复发。

```jsonl
{"ts": "2025-10-19T...", "task": "auth", "mistake": "forgot kong routing", "rule": "all services route through kong", "fix": "added kong route", "tests": ["test_kong.py"], "status": "adopted"}
```

**功能**：
- 添加条目：`memory.add_entry(ReflexionEntry(...))`
- 搜索类似错误：`memory.search_similar_mistakes("kong routing")`
- 获取规则：`memory.get_rules()`

---

### 4. 验证器 (Validators)

**路径**：`superclaude/validators/`

#### ContextContractValidator
- 强制执行项目规则。
- 检查 .env 文件创建（禁止）。
- 检测硬编码的密钥。
- 验证 Kong/Traefik 路由。

#### DependencySanityValidator
- 验证 `package.json` / `pyproject.toml`。
- 检查包名格式。
- 检测版本不一致。

#### RuntimePolicyValidator
- 验证 Node.js / Python 版本。
- 检查引擎规格说明。
- 确保 lockfile 一致性。

#### TestRunnerValidator
- 检测改动中的测试文件。
- 自动运行测试。
- 测试不通过则报错。

#### SecurityRoughcheckValidator
- 检测硬编码的密钥 (Stripe, Supabase, OpenAI, Infisical)。
- 拦截 .env 文件创建。
- 对不安全模式（eval, exec, shell=True）发出警告。

---

## 📊 测试覆盖率

**总计**：26 个测试，全部通过。

### PM 初始化测试 (11 个测试)
- ✅ Git 仓库检测。
- ✅ 结构扫描。
- ✅ 上下文合约生成 (Infisical, Kong, Traefik)。
- ✅ 运行时检测 (Node, Python, pnpm, uv)。
- ✅ 反思记忆 (加载、添加、搜索)。

### 验证器测试 (15 个测试)
- ✅ 上下文合约验证。
- ✅ 依赖项健全性检查。
- ✅ 运行时策略验证。
- ✅ 安全性初步检查 (密钥, .env, 不安全模式)。
- ✅ 验证链工作流 (全过, 或提早中止)。

```bash
# 运行测试
uv run pytest tests/core/pm_init/ tests/validators/ -v

# 结果
============================== 26 passed in 0.08s ==============================
```

---

## 🔄 下一步计划 (第二阶段)

**本 PR 不包含以下内容** (将在第二阶段实现)：

1. **规划 (PLANNING)** 阶段 (`commands/pm/plan.py`)：生成 3-5 个计划 → 自我批判 → 剪除糟糕计划。
2. **任务列表 (TASKLIST)** 阶段 (`commands/pm/tasklist.py`)：拆分为并行/串行任务。
3. **执行 (DO)** 阶段 (`commands/pm/do.py`)：在验证门限的加持下执行。
4. **反思 (ACTION)** 阶段 (`commands/pm/reflect.py`)：实现后的反思与学习。

---

## ✅ 检查清单

- [x] PM 初始化钩子已实现
- [x] 上下文合约自动生成功能
- [x] 反思记忆系统
- [x] 实现了 5 个核心验证器
- [x] 编写并通过了 26 个测试
- [x] 文档已补全
- [ ] 经过代码评审
- [ ] 合并到 integration 分支

---

**评审就绪**：本 PR 为“PM 优先”奠定了基础。所有测试均通过，无破坏性变更。
