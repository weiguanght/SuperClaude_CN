# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在处理本仓库代码时提供指导。

## 🐍 Python 环境规则

**关键要点**：本项目所有 Python 操作均使用 **UV**。严禁直接使用 `python -m`、`pip install` 或 `python script.py`。

### 必需命令

```bash
# 所有 Python 操作必须使用 UV
uv run pytest                    # 运行测试
uv run pytest tests/pm_agent/   # 运行特定测试
uv pip install package           # 安装依赖项
uv run python script.py          # 执行脚本
```

## 📂 项目结构

> **⚠️ 重要提示**：旧文档中描述的 `.claude-plugin/` 目录及 TypeScript 插件系统在 v4.1.9 中**不存在**。
> 该功能计划在 v5.0 中实现 (参见 [issue #419](https://github.com/SuperClaude-Org/SuperClaude_Framework/issues/419))。

**当前 v4.1.9 架构**：带有斜杠命令的 Python 包

```
# Claude Code 配置 (v4.1.9)
.claude/
├── settings.json        # 用户设置
└── commands/            # 斜杠命令 (通过 `superclaude install` 安装)
    ├── pm.md
    ├── research.md
    └── index-repo.md

# Python 包
src/superclaude/         # Pytest 插件 + CLI 工具
├── pytest_plugin.py     # 自动加载的 pytest 集成
├── pm_agent/            # confidence.py, self_check.py, reflexion.py
├── execution/           # parallel.py, reflection.py, self_correction.py
└── cli/                 # main.py, doctor.py, install_skill.py

# 插件开发 (计划在 v5.0 中实现 - 参见 docs/plugin-reorg.md)
plugins/superclaude/     # 未来的插件源 (尚未激活)
├── agents/              # 智能体定义
├── commands/            # 命令定义
├── hooks/               # 钩子配置
├── scripts/             # Shell 脚本
└── skills/              # 技能实现

# 项目文件
tests/                   # Python 测试套件
docs/                    # 文档
scripts/                 # 分析工具 (工作流指标, A/B 测试)
PLANNING.md              # 架构, 绝对规则
TASK.md                  # 当前任务
KNOWLEDGE.md             # 累积的见解
```

## 🔧 开发工作流

### 基础命令

```bash
# 设置
make dev              # 以可编辑模式安装并包含开发依赖项
make verify           # 验证安装 (包、插件、健康状态)

# 测试
make test             # 运行完整测试套件
uv run pytest tests/pm_agent/ -v              # 运行特定目录
uv run pytest tests/test_file.py -v           # 运行特定文件
uv run pytest -m confidence_check             # 按标记运行
uv run pytest --cov=superclaude               # 带覆盖率报告

# 代码质量
make lint             # 运行 ruff 代码检查
make format           # 使用 ruff 格式化代码
make doctor           # 健康检查诊断

# 插件打包
make build-plugin            # 将插件构建产物输出至 dist/
make sync-plugin-repo        # 将产物同步至 ../SuperClaude_Plugin

# 维护
make clean            # 移除构建产物
```

## 📦 核心架构

### Pytest 插件 (自动加载)

通过 `pyproject.toml` 入口点注册,安装后自动可用。

**Fixtures (固件)**：`confidence_checker`, `self_check_protocol`, `reflexion_pattern`, `token_budget`, `pm_context`

**自动标记**：
- `/unit/` 中的测试 → `@pytest.mark.unit`
- `/integration/` 中的测试 → `@pytest.mark.integration`

**自定义标记**：`@pytest.mark.confidence_check`, `@pytest.mark.self_check`, `@pytest.mark.reflexion`

### PM 智能体 - 三大核心模式

**1. ConfidenceChecker (置信度检查器)** (src/superclaude/pm_agent/confidence.py)
- 执行前的置信度评估：≥90% 继续执行，70-89% 提供备选方案，<70% 提问
- 防止错误方向的工作，ROI：节省 25-250 倍 Token

**2. SelfCheckProtocol (自检协议)** (src/superclaude/pm_agent/self_check.py)
- 实施后基于证据的验证
- 不进行推测 - 通过测试/文档进行验证

**3. ReflexionPattern (反思模式)** (src/superclaude/pm_agent/reflexion.py)
- 错误学习与预防
- 跨会话模式匹配

### 并行执行

**Wave → Checkpoint → Wave 模式** (src/superclaude/execution/parallel.py)：
- 比串行执行快 3.5 倍
- 自动依赖分析
- 示例：[并行读取文件] → 分析 → [并行编辑文件]

### TypeScript 插件 (计划在 v5.0 中实现)

> **⚠️ 尚未实现**：下述 TypeScript 插件系统在 v4.1.9 中不存在。
> 该功能计划在 v5.0 中实现。参见 [issue #419](https://github.com/SuperClaude-Org/SuperClaude_Framework/issues/419) 及 `docs/plugin-reorg.md`。

**当前 v4.1.9 命令** (斜杠命令,非插件)：
- 安装方式：`pipx install superclaude && superclaude install`
- 命令安装位置：`~/.claude/commands/`
- 可用命令：`/pm`, `/research`, `/index-repo` (及其他)

**计划中的插件架构** (v5.0 - 尚未可用)：
- 插件源将位于 `plugins/superclaude/`
- `make build-plugin` 将渲染 `.claude-plugin/*` 清单
- 通过 `.claude-plugin/plugin.json` 进行项目本地检测
- 支持市场分发

## 🧪 使用 PM 智能体进行测试

### 带标记的测试示例

```python
@pytest.mark.confidence_check
def test_feature(confidence_checker):
    """执行前置信度检查 - 若 < 70% 则跳过"""
    context = {"test_name": "test_feature", "has_official_docs": True}
    assert confidence_checker.assess(context) >= 0.7

@pytest.mark.self_check
def test_implementation(self_check_protocol):
    """带证据的实施后验证"""
    implementation = {"code": "...", "tests": [...]}
    passed, issues = self_check_protocol.validate(implementation)
    assert passed, f"验证失败: {issues}"

@pytest.mark.reflexion
def test_error_learning(reflexion_pattern):
    """若测试失败,反思模式会记录以供未来预防"""
    pass

@pytest.mark.complexity("medium")  # simple: 200, medium: 1000, complex: 2500
def test_with_budget(token_budget):
    """Token 预算分配"""
    assert token_budget.limit == 1000
```

## 🌿 Git 工作流

**分支结构**：`master` (生产) ← `integration` (测试) ← `feature/*`, `fix/*`, `docs/*`

**标准工作流**：
1. 从 `integration` 创建分支：`git checkout -b feature/your-feature`
2. 带测试的开发：`uv run pytest`
3. 提交：`git commit -m "feat: description"` (遵循约定式提交)
4. 合并至 `integration` → 验证 → 合并至 `master`

**当前分支**：参见会话启动输出中的 git 状态

### 使用 Git Worktrees 进行并行开发

**关键要点**：在并行运行多个 Claude Code 会话时,使用 `git worktree` 以避免冲突。

```bash
# 为 integration 分支创建 worktree
cd ~/github/SuperClaude_Framework
git worktree add ../SuperClaude_Framework-integration integration

# 为 feature 分支创建 worktree
git worktree add ../SuperClaude_Framework-feature feature/pm-agent
```

**优势**：
- 在不同分支上同时运行 Claude Code 会话
- 无分支切换冲突
- 独立的工作目录
- 无状态损坏的并行开发

**用法**：
- 会话 A：打开 `~/github/SuperClaude_Framework/` (当前分支)
- 会话 B：打开 `~/github/SuperClaude_Framework-integration/` (integration)
- 会话 C：打开 `~/github/SuperClaude_Framework-feature/` (feature 分支)

**清理**：
```bash
git worktree remove ../SuperClaude_Framework-integration
```

## 📝 关键文档文件

**PLANNING.md** - 架构、设计原则、绝对规则
**TASK.md** - 当前任务与优先级
**KNOWLEDGE.md** - 累积的见解与故障排除

其他文档位于 `docs/user-guide/`, `docs/developer-guide/`, `docs/reference/`

## 💡 核心开发原则

### 1. 基于证据的开发
**绝不猜测** - 在实施前通过官方文档 (Context7 MCP, WebFetch, WebSearch) 进行验证。

### 2. 置信度优先的实施
在开始前检查置信度：≥90% 继续执行，70-89% 提供备选方案，<70% 提问。

### 3. 并行优先的执行
使用 **Wave → Checkpoint → Wave** 模式 (快 3.5 倍)。示例：`[并行读取文件]` → 分析 → `[并行编辑文件]`

### 4. Token 效率
- 简单 (拼写错误)：200 Token
- 中等 (Bug 修复)：1,000 Token
- 复杂 (功能开发)：2,500 Token
- 置信度检查 ROI：花费 100-200 Token 可节省 5,000-50,000 Token

## 🔧 MCP 服务器集成

通过 **airis-mcp-gateway** 集成多个 MCP 服务器。

**高优先级**：
- **Tavily**：网络搜索 (深度调研)
- **Context7**：官方文档 (防止幻觉)
- **Sequential**：Token 高效推理 (减少 30-50%)
- **Serena**：会话持久化
- **Mindbase**：跨会话学习

**可选**：Playwright (浏览器自动化), Magic (UI 组件), Chrome DevTools (性能)

**用法**：TypeScript 插件和 Python pytest 插件可调用 MCP 服务器。对于文档/调研,始终优先使用 MCP 工具而非推测。

## 🚀 开发与安装

### 当前安装方法 (v4.1.9)

**标准安装**：
```bash
# 选项 1: pipx (推荐)
pipx install superclaude
superclaude install

# 选项 2: 直接从仓库安装
git clone https://github.com/SuperClaude-Org/SuperClaude_Framework.git
cd SuperClaude_Framework
./install.sh
```

**开发模式**：
```bash
# 以可编辑模式安装
make dev

# 运行测试
make test

# 验证安装
make verify
```

### 插件系统 (计划在 v5.0 中实现 - 尚未可用)

> **⚠️ 重要提示**：旧文档中描述的插件系统在 v4.1.9 中**不存在**。
> 这些功能计划在 v5.0 中实现 (参见 [issue #419](https://github.com/SuperClaude-Org/SuperClaude_Framework/issues/419))。

**目前不可用的功能**：
- ❌ `.claude-plugin/` 目录自动检测
- ❌ `/plugin marketplace add` 命令
- ❌ `/plugin install superclaude`
- ❌ `make build-plugin` (已计划但尚未实现)
- ❌ 项目本地插件检测

**未来计划** (v5.0)：
- 插件市场分发
- 基于 TypeScript 的插件架构
- 通过 `.claude-plugin/plugin.json` 自动检测
- 通过 `make build-plugin` 构建工作流

参见 `docs/plugin-reorg.md` 及 `docs/next-refactor-plan.md` 了解实施计划。

## 📊 包信息

**包名称**：`superclaude`
**版本**：0.4.0
**Python**：>=3.10
**构建系统**：hatchling (PEP 517)

**入口点**：
- CLI：`superclaude` 命令
- Pytest 插件：自动加载为 `superclaude`

**依赖项**：
- pytest>=7.0.0
- click>=8.0.0
- rich>=13.0.0
