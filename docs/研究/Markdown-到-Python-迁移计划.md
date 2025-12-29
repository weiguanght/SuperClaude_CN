# Markdown → Python 迁移计划

**日期**：2025-10-20
**问题**：基于 Markdown 的模式在每次会话中加载时会消耗 41,000 个 Token，且缺乏强制约束力。
**解决方案**：采用 Python 优先的实施方案，并提供向技能系统 (Skills API) 迁移的路径。

## 当前的 Token 浪费情况

### 每次会话加载的 Markdown 文件

**最消耗 Token 的文件**：
```
pm-agent.md                    16,201 字节  (约 4,050 Token)
rules.md (框架规则)            16,138 字节  (约 4,034 Token)
socratic-mentor.md             12,061 字节  (约 3,015 Token)
MODE_Business_Panel.md         11,761 字节  (约 2,940 Token)
business-panel-experts.md       9,822 字节  (约 2,455 Token)
config.md (研究文档)            9,607 字节  (约 2,401 Token)
examples.md (专家示例)          8,253 字节  (约 2,063 Token)
symbols.md (符号系统)           7,653 字节  (约 1,913 Token)
flags.md (框架标志)             5,457 字节  (约 1,364 Token)
MODE_Task_Management.md         3,574 字节  (约 893 Token)

总计：约 164KB = 每次会话约 41,000 个 Token
```

**年度成本** (按每年 200 次会话计算)：
- Token：每年 8,200,000 个。
- 成本：即使只是读取文档，每年也要花费约 20-40 美元。

## 迁移战略

### 第一阶段：验证器 (已完成 ✅)

**已实施内容**：
```python
superclaude/validators/
├── security_roughcheck.py  # 密钥等硬编码残留探测
├── context_contract.py     # 项目规则强制执行
├── dep_sanity.py           # 依赖项合规验证
├── runtime_policy.py       # 运行时版本检查
└── test_runner.py          # 测试执行
```

**收益**：
- ✅ 通过 Python 强制执行（而不仅仅是文档说明）。
- ✅ 26 个测试点证明了其正确性。
- ✅ 设立了执行前的验证门禁。

### 第二阶段：模式强制执行 (下一步)

**当前问题**：
```markdown
# MODE_Orchestration.md (2,759 字节)
- 工具选择矩阵
- 资源管理
- 并行执行触发器
= 每次都要读，但缺乏强制执行力
```

**Python 解决方案**：
```python
# superclaude/modes/orchestration.py

from enum import Enum
from typing import Literal, Optional
from functools import wraps

class ResourceZone(Enum):
    GREEN = "0-75%"   # 全能力运作
    YELLOW = "75-85%" # 效率模式
    RED = "85%+"      # 仅核心功能

class OrchestrationMode:
    """智能工具选择与资源管理"""

    @staticmethod
    def select_tool(task_type: str, context_usage: float) -> str:
        """
        工具选择矩阵 (在运行时强制执行)

        重构前 (Markdown): "在开发 UI 组件时使用 Magic MCP" (无强制力)
        重构后 (Python): 当 task_type="ui" 时，自动路由到 Magic MCP
        """
        if context_usage > 0.85:
            # 红色区域：仅保留核心工具
            return "native"

        tool_matrix = {
            "ui_components": "magic_mcp",
            "deep_analysis": "sequential_mcp",
            "pattern_edits": "morphllm_mcp",
            "documentation": "context7_mcp",
            "multi_file_edits": "multiedit",
        }

        return tool_matrix.get(task_type, "native")

    @staticmethod
    def enforce_parallel(files: list) -> bool:
        """
        自动触发并行执行

        重构前 (Markdown): "处理 3 个以上文件时应使用并行模式"
        重构后 (Python): 处理 3 个以上文件时自动强制执行并行化
        """
        return len(files) >= 3

# 用于激活该模式的装饰器
def with_orchestration(func):
    """为函数应用编排模式"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        # 执行编排规则
        mode = OrchestrationMode()
        # ... 执行逻辑 ...
        return func(*args, **kwargs)
    return wrapper
```

**Token 节省**：
- 重构前：每次会话 2,759 字节 (约 689 Token)。
- 重构后：仅在被使用时导入 (约 50 Token)。
- 节省：93%。

### 第三阶段：项目管理智能体 (PM Agent) 的 Python 实现

**当前**：
```markdown
# pm-agent.md (16,201 字节 = 4,050 Token)

实施前置信度检查
实施后自我检查
反思模式
带有内省的并行模式
```

**Python 化后**：
```python
# superclaude/agents/pm.py

from dataclasses import dataclass
from typing import Optional
from superclaude.memory import ReflexionMemory
from superclaude.validators import ValidationGate

@dataclass
class ConfidenceCheck:
    """实施前的置信度验证"""
    requirement_clarity: float  # 0-1
    context_loaded: bool
    similar_mistakes: list

    def should_proceed(self) -> bool:
        """强制执行：仅在置信度 >70% 时继续"""
        return self.requirement_clarity > 0.7 and self.context_loaded

class PMAgent:
    """带有工作流强制执行力的项目管理智能体"""

    def __init__(self, repo_path: Path):
        self.memory = ReflexionMemory(repo_path)
        self.validators = ValidationGate()

    def execute_task(self, task: str) -> Result:
        """
        四阶段工作流 (自动执行，而非仅记录在档)
        """
        # 第一阶段：规划 (带位置信度检查)
        confidence = self.check_confidence(task)
        if not confidence.should_proceed():
            return Result.error("低置信度 - 需要进一步澄清需求")

        # 第二阶段：任务列表
        tasks = self.decompose(task)

        # 第三阶段：执行 (带有验证门禁)
        for subtask in tasks:
            if not self.validators.validate(subtask):
                return Result.error(f"验证失败：{subtask}")
            self.execute(subtask)

        # 第四阶段：反思
        self.memory.learn_from_execution(task, tasks)

        return Result.success()
```

**Token 节省**：
- 重构前：每次会话 16,201 字节 (约 4,050 Token)。
- 重构后：仅在调用 `/sc:pm` 命令时导入 (约 100 Token)。
- 节省：97%。

### 第四阶段：技能系统 (Skills API) 迁移 (长期规划)

**按需加载的技能**：
```
skills/pm-mode/
  SKILL.md (200 字节)      # 仅包含标题和描述
  agent.py (16KB)          # 完整实施代码
  memory.py (5KB)          # 反思记忆
  validators.py (8KB)      # 验证门禁

会话开始时：加载 200 字节
使用 /sc:pm 时：按需加载完整 29KB 内容
从未被使用时：始终只占 200 字节
```

**Token 对比**：
```
当前 Markdown 模式：每次会话 16,201 字节 = 4,050 Token
Python 导入模式：    仅导入头部 = 100 Token
技能系统 (Skills)：  按需加载 = 50 Token (仅描述信息)

收益：使用技能系统后，Token 节省达 98.8%。
```

## 实施优先级

### 近期 (本周)

1. ✅ **索引命令** (`/sc:index-repo`)
   - 已创建。
   - 在安装时自动运行。
   - Token 节省 94%。

2. ✅ **设置自动索引**
   - 已集成至 `knowledge_base.py`。
   - 在安装过程中运行。
   - 创建 `PROJECT_INDEX.md`。

### 短期 (2-4 周)

3. **编排模式 Python 化**
   - 编写 `superclaude/modes/orchestration.py`。
   - 工具选择矩阵 (强制化)。
   - 资源管理 (自动化)。
   - **节省**：689 Token → 50 Token (93%)。

4. **PM 智能体 Python 核心化**
   - 编写 `superclaude/agents/pm.py`。
   - 置信度检查 (强制化)。
   - 四阶段工作流 (自动化)。
   - **节省**：4,050 Token → 100 Token (97%)。

### 中期 (1-2 个月)

5. **全模式 → Python 化**
   - 头脑风暴、内省、任务管理模式。
   - **整体节省**：约 10,000 Token → 约 500 Token (95%)。

6. **技能原型设计** (Issue #441)
   - 将 1-2 个模式转化为技能。
   - 测量按需加载的效率。
   - 向主线进行报告。

### 长期 (3 个月以上)

7. **全量技能系统迁移**
   - 所有模式 → 技能。
   - 所有智能体 → 技能。
   - **目标**：实现 98% 的 Token 削减。

## 代码示例对比

### 重构前 (基于 Markdown 模式)

```markdown
# MODE_Orchestration.md

## 工具选择矩阵
| 任务类型 | 最佳工具 |
|-----------|-----------|
| UI | Magic MCP |
| 分析 | Sequential MCP |

## 资源管理
绿色区域 (0-75%): 全能力运作
黄色区域 (75-85%): 效率模式
红色区域 (85%+): 仅核心功能
```

**存在的问题**：
- ❌ 每次会话都要消耗 689 个 Token。
- ❌ 缺乏强制执行力。
- ❌ 无法测试规则是否被遵循。
- ❌ 各模式之间存在大量的重复内容。

### 重构后 (Python 强制执行)

```python
# superclaude/modes/orchestration.py

class OrchestrationMode:
    TOOL_MATRIX = {
        "ui": "magic_mcp",
        "analysis": "sequential_mcp",
    }

    @classmethod
    def select_tool(cls, task_type: str) -> str:
        return cls.TOOL_MATRIX.get(task_type, "native")

# 使用示例
tool = OrchestrationMode.select_tool("ui")  # 返回 "magic_mcp" (强制性结果)
```

**收益**：
- ✅ 导入时仅消耗约 50 个 Token。
- ✅ 在运行时自动强制隔离。
- ✅ 可以使用 pytest 进行系统化测试。
- ✅ 消除冗余 (遵循 DRY 原则)。

## 迁移检查清单

### 各模式迁移
- [ ] 阅读现有的 Markdown 模式文档。
- [ ] 提取规则和行为逻辑。
- [ ] 设计 Python 类结构。
- [ ] 使用类型注解实施开发。
- [ ] 编写测试 (覆盖率 >80%)。
- [ ] 进行 Token 使用量基准测试。
- [ ] 更新命令处理系统以使用 Python 版本。
- [ ] 将 Markdown 保留为说明性文档。

### 测试策略
```python
# tests/modes/test_orchestration.py

def test_tool_selection():
    """验证工具选择矩阵"""
    assert OrchestrationMode.select_tool("ui") == "magic_mcp"
    assert OrchestrationMode.select_tool("analysis") == "sequential_mcp"

def test_parallel_trigger():
    """验证并行执行自动触发器"""
    assert OrchestrationMode.enforce_parallel([1, 2, 3]) == True
    assert OrchestrationMode.enforce_parallel([1, 2]) == False

def test_resource_zones():
    """验证资源管理强制性"""
    mode = OrchestrationMode(context_usage=0.9)
    assert mode.zone == ResourceZone.RED
    assert mode.select_tool("ui") == "native"  # 红色区域：仅限核心功能
```

## 预期成果

### Token 效率

**迁移前**：
```
单次会话消耗：
- 各类模式：26,716 Token
- 各类智能体：40,000+ Token (PM 智能体及其他)
- 总计：每次会话约 66,000 个 Token

年度累计 (按 200 次计算)：
- 总计：13,200,000 个 Token
- 成本：每年约 26-50 美元
```

**Python 迁移后**：
```
单次会话消耗：
- 模式导入：约 500 Token
- 智能体导入：约 1,000 Token
- 项目索引 (PROJECT_INDEX)：3,000 Token
- 总计：每次会话约 4,500 个 Token

年度累计 (按 200 次计算)：
- 总计：900,000 个 Token
- 成本：每年约 2-4 美元

节省幅度：Token 削减 93%，成本降低 90% 以上。
```

**技能系统迁移后**：
```
单次会话消耗：
- 技能描述：约 300 Token
- 项目索引：3,000 Token
- 按需加载：波动
- 总计：每次会话约 3,500 个 Token (针对不常使用的模式)

节省幅度：Token 削减 95% 以上。
```

### 质量提升对比

**Markdown 模式**：
- ❌ 无强制执行力（仅为文档）。
- ❌ 无法验证合规性。
- ❌ 无法测试其有效性。
- ❌ 容易随着时间产生偏差。

**Python 模式**：
- ✅ 在运行时自动强制执行。
- ✅ 100% 可测试。
- ✅ 基于类型注解的安全性。
- ✅ 单一事实来源 (SSOT)。

## 风险与缓解

**风险 1**：破坏现有工作流。
- **缓解**：保留 Markdown 文档作为手册兜底。

**风险 2**：技能系统 (Skills API) 尚不成熟。
- **缓解**：现阶段以 Python 优先，技能系统待后续逐步接入。

**风险 3**：实施复杂度高。
- **缓解**：采用增量迁移策略（一次仅迁移一个模式）。

## 结论

**推荐路径**：
1. ✅ **已完成**：索引命令 + 自动索引 (节省 94%)。
2. **下一步**：编排模式 → Python 化 (节省 93%)。
3. **随后**：PM 智能体 → Python 化 (节省 97%)。
4. **长远**：技能原型开发 + 全量迁移 (节省 98%)。

**最终预期节省**：实现 93-98% 的 Token 削减。

---

**开始日期**：2025-10-20
**目标完成日期**：2026-01-20 (全量迁移预计用时 3 个月)
**短期目标 (Quick Win)**：编排模式迁移 (预计用时 1 周)
