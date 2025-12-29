# Python src 布局研究 - 仓库与包的命名 (Python Src Layout Research)

**日期**：2025-10-21
**研究课题**：`superclaude` 仓库是否应使用 `src/superclaude/`（双重嵌套）结构，还是应采用更简洁的结构？
**置信度**：高 (90%) - 基于 PyPA 官方文档及主流开源项目实例。

---

## 🎯 执行摘要 (Executive Summary)

**结论**：`src/superclaude/` 的双重嵌套结构是**正确**的，虽然它不是强制选项。

**你的直觉没错**：
- 通常情况下，仓库名 = 包名。
- 虽然推荐使用 `src/` 布局，但包名的重复感确实存在。
- 然而，PyPA 官方示例明确使用了 `src/package_name/` 这种结构。

**可选路径**：
1. **标准型 (PyPA 推荐)**：`src/superclaude/` ← **当前采用的结构**。
2. **简约型 (可行)**：仅保留 `src/`，模块直接置于其下（不推荐，会造成命名空间混淆）。
3. **扁平型 (旧式)**：包直接位于仓库根目录下（如 `superclaude/`）。

---

## 📚 调查结果

### 1. PyPA 官方指南

**来源**：[https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/](https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/)

**官方示例结构**：
```
project_root/
├── src/
│   └── awesome_package/    # ← 使用包名进行双重嵌套
│       ├── __init__.py
│       └── module.py
├── pyproject.toml
└── README.md
```

**PyPA 的核心建议**：
- **强烈建议** ("strongly suggested") 使用 `src/` 布局。
- **理由**：
  1. ✅ 防止在安装前发生错误的本地导入。
  2. ✅ 及早发现打包错误。
  3. ✅ 确保测试是在用户最终安装的形式下运行的。

**重要发现**：PyPA 官方将 `src/package_name/` 作为**标准示例**。

---

### 2. 现实世界项目调研

| 项目 | 仓库名 | 采用结构 | 包名 | 备注 |
| :--- | :--- | :--- | :--- | :--- |
| **Click** | `click` | ✅ `src/click/` | `click` | 完全符合 PyPA 推荐 |
| **FastAPI** | `fastapi` | ❌ 扁平型 `fastapi/` | `fastapi` | 位于根目录下 |
| **setuptools** | `setuptools` | ❌ 扁平型 `setuptools/` | `setuptools` | 位于根目录下 |

**观察结论**：
- 所有项目均遵循 **仓库名 = 包名**。
- Click 等现代项目倾向于采用 `src/` 布局。
- FastAPI/setuptools 采用扁平化结构多出于历史原因。

---

### 3. 为什么要采用双重嵌套？

**逻辑如下**：
1. **仓库名** (`superclaude`)：GitHub 上的项目名称，代表整个工程。
2. **包名** (`src/superclaude/`)：在 Python 中通过 `import superclaude` 调用的名称。
3. **命名一致性**：将两者的名称保持一致是**行业标准模式**，尽管这在物理目录上看起来像是重复。

**与单仓库 (Monorepo) 的区别**：
- 单仓库：在 `src/` 下包含多个包 (如 `src/pkg1/`, `src/pkg2/`)。
- SuperClaude：目前是单包项目，因此表现为仓库名与包名重合。

---

## 🔀 备选方案评估

### 选项 1：当前结构 (PyPA 推荐)
**优点**：
- ✅ 完全遵循 PyPA 官方最佳实践。
- ✅ 与 Click 等现代知名项目结构一致。
- ✅ 对打包工具最友好。

**缺点**：
- ❌ 路径较长：如 `src/superclaude/pm_agent/confidence.py`。
- ❌ 直观上显得有些冗余。

---

### 选项 2：扁平 src/ 结构 (非标准)
```
superclaude/                 # 仓库
├── src/
│   ├── __init__.py          # ← 直接将 src 视为包
│   └── ...
```
**评价**：❌ **不推荐**。这会导致 `import src` 这种极具迷惑性的代码，偏离了包命名的初衷。

---

### 选项 3：扁平布局 (不推荐)
```
superclaude/                 # 仓库
├── superclaude/             # 直接放在根目录下
│   └── ...
```
**评价**：❌ **PyPA 不推荐**。开发阶段容易发生本地源码与已安装版本的冲突风险。

---

## 💡 最终建议

**结论：维持当前结构不变。**

**理由**：
1. ✅ 符合 PyPA 官方准则。
2. ✅ 遵循现代 Python 开发的最佳实践（参考 Click）。
3. ✅ 与主流打包工具结合最紧密。
4. ✅ 方便未来可能的单仓库 (Monorepo) 扩展。

**对你疑问的回应**：
- 双重嵌套是**刻意设计的标准**。
- 仓库名（项目概念）与包名（Python 导入概念）虽然名字相同，但属于不同层级的命名空间。

---

## 🎓 经验总结

1. **src/ 布局的真谛**：强制要求在测试前进行安装，从而在早期发现环境与路径问题。
2. **双重嵌套的逻辑**：`src/` 负责物理上的发布内容分离，内部的 `package_name/` 负责定义 import 时的标识。
3. **行业趋势**：新创建的项目应优先选择 `src/package_name/` 结构。

---

**研究完成时间**：2025-10-21
**置信度**：高 (90%)
**行动建议**：**无需修改**。当前 `src/superclaude/` 的结构正是目前 Python 界的最佳实践。
