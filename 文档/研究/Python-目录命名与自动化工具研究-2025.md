# 研究：Python 目录命名与自动化工具 (Python Directory Naming & Automation Tools)

**研究日期**：2025-10-14
**研究课题**：PEP 8 目录命名合规性、自动化 Lint 工具以及 Git 大小写敏感重命名的最佳实践。

---

## 🎯 执行摘要 (Executive Summary)

### 核心发现

1. **PEP 8 标准 (2025)**：
   - 包/目录：仅允许**全小写**，虽然不推荐使用下划线但在实践中被广泛接受。
   - 模块/文件：**全小写**，允许使用下划线以提高可读性。
   - 现状违规：`Developer-Guide`, `Getting-Started` 等使用了大写和连字符。

2. **自动化 Lint 工具**：**Ruff** 是 2025 年的行业标准（Rust 编写，比 Flake8 快 10-100 倍）。但需注意，Ruff 主要验证代码内容而非目录结构。

3. **Git 大小写重命名**：在 macOS (APFS) 等大小写不敏感的文件系统上，推荐使用 **两步法 `git mv`**（例如：`git mv Foo tmp && git mv tmp foo`）以避免同步问题。

4. **自动化策略**：建议使用自定义的 **pre-commit hooks** 来强制执行目录命名约束。

---

## 📚 详细调研结果

### 1. PEP 8 目录命名惯例
- **官方标准**：Python 包应使用简短且全小写的名称。
- **现状违规**：项目中 `docs/Reference`, `docs/Templates` 等目录均违反了小写原则。
- **文档目录例外**：虽然 `docs/` 下的子目录并非 Python 包，但为了可读性和行业惯例，推荐使用全小写 + 连字符（如 `getting-started`）。

### 2. 自动化工具推荐 (2025)

#### Ruff - 现代代码检查标杆
- **功能**：集成了代码格式化 (replaces Black)、导入排序 (replaces isort) 以及 800 余项 Lint 规则。
- **配置**：通过 `pyproject.toml` 统一管理。
- **局限**：校验对象是 Python 源码文件，无法自动更名目录。

#### validate-pyproject
- **用途**：验证 `pyproject.toml` 是否符合 PEP 标准。
- **集成**：可作为 pre-commit hook 运行，但不负责目录命名。

---

### 3. Git 大小写重命名最佳实践
由于 macOS 文档系统通常不区分大小写，直接更名 (如 `mv Foo foo`) Git 可能无法检测到。

**最安全的方法 (两步 `git mv`)**：
```bash
git mv docs/User-Guide docs/user-guide-tmp
git mv docs/user-guide-tmp docs/user-guide
git commit -m "重构：将 User-Guide 重命名为全小写"
```

---

### 4. 强制执行命名的自动化脚本 (Hooks)

为了防止未来再次出现违规命名，建议在 `scripts/` 下编写 Python 校验脚本，并集成到 `pre-commit` 环境中：
- 脚本逻辑：遍历目录，若发现非小写开头的 Python 包或文档子目录，则拦截 `git commit` 操作。

---

### 5. 2025 现代 Python 项目结构 (uv)
推荐采用 **基于 src 的布局 (src-based layout)**：
```
project-root/
├── pyproject.toml
├── uv.lock
├── src/
│   └── package_name/ (全小写)
├── tests/
└── docs/ (全小写+连字符)
```

---

## 🚀 对 SuperClaude 框架的建议行动

1. **手动完成重命名**：使用“两步法”对剩余的 6 个违规目录执行 `git mv`。
2. **配置 Ruff**：在 `pyproject.toml` 中开启 `N` (命名规范) 系列规则。
3. **设置 pre-commit hooks**：引入 `check-case-conflict` (检测大小写冲突) 和自定义的目录命名校验脚本。
4. **CI/CD 验证**：在 GitHub Actions 中增加命名检查步骤，确保合并前的合规性。

---

**研究完成时间**：2025-10-14
**置信度**：高
**最终结论**：目录重命名无法完全自动化（尤其在 macOS 上），必须通过“手动重命名 + 自动化预防”的混合模式来维持 PEP 8 合规性。
