# Serena MCP 安装疑难解答 (Troubleshooting)

## 常见问题与解决方案

### 问题：出现 "Failed to spawn: serena" 错误

**现象：**
```
error: Failed to spawn: `serena`
Caused by: No such file or directory (os error 2)
```

**根本原因：**
SuperClaude 安装程序之前被错误地配置为使用 `uv run serena` 而非 `uvx` 来安装 Serena MCP 服务器。

**解决方案：**

1. **移除现有的损坏安装：**
   ```bash
   claude mcp remove serena
   ```

2. **使用正确的 uvx 方法安装 Serena：**
   ```bash
   uvx --from git+https://github.com/oraios/serena serena --help
   ```

3. **在 Claude CLI 中进行注册：**
   ```bash
   claude mcp add serena -- uvx --from git+https://github.com/oraios/serena serena start-mcp-server --context ide-assistant
   ```

4. **验证安装状态：**
   ```bash
   claude mcp list
   ```

---

### 问题：关于 uv 与 uvx 的混淆

**区别：**
- `uv run serena` —— 从本地项目依赖中运行 serena（如果本地未安装该包则会失败）。
- `uvx --from git+https://github.com/oraios/serena serena` —— 直接从 GitHub 仓库运行 serena。

**正确做法：**
始终为 Serena 使用 `uvx`，因为它被设计为直接与远程 GitHub 仓库协同工作。

---

### GitHub Codespace 特定问题

**问题：UV 安装方式**
如果您是使用 curl 方式安装的 UV：
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```
请确保 `uvx` 可用：
```bash
uvx --version
```
如果不可用，请通过 pip 安装 uv：
```bash
pip install uv
```

---

### 验证步骤

安装成功后，请验证 Serena 是否正常工作：

1. **检查 MCP 连接**：运行 `claude mcp list`。
2. **测试基本功能**：启动 Claude Code，确认 Serena 出现在可用的 MCP 服务器列表中。
3. **查看日志排错**：
   ```bash
   ls ~/.cache/claude-cli-nodejs/*/mcp-logs-serena/
   cat ~/.cache/claude-cli-nodejs/*/mcp-logs-serena/latest.txt
   ```

---

### 环境特定说明

**GitHub Codespaces:**
- UV 通常是预装的，但可能不包含 `uvx`。
- 默认 Python 环境可能需要通过安装 `uv` 包来补全。
- 需要具备访问 `git+https://` 链接的网络权限。

**本地开发:**
- 确保已安装 `uvx`：运行 `pip install uv` 或 `pipx install uv`。
- 验证对 GitHub 仓库的 Git 访问权限。

**WSL/Linux:**
- 确保对 `~/.claude/` 目录具备正确的读写权限。
- 检查 Python 环境的兼容性。

---

### 手动配置

如果自动安装失败，可手动配置 `~/.claude.json` 文件：

```json
{
  "mcpServers": {
    "serena": {
      "command": "uvx",
      "args": [
        "--from",
        "git+https://github.com/oraios/serena",
        "serena",
        "start-mcp-server",
        "--context",
        "ide-assistant"
      ]
    }
  }
}
```

---

### 获取进一步帮助

如果问题依然存在：
1. 请查阅 [Serena 官方文档](https://github.com/oraios/serena)。
2. 验证 uvx 安装情况：`uvx --version`。
3. 测试直接安装：`uvx --from git+https://github.com/oraios/serena serena --help`。
4. 向 [SuperClaude 框架](https://github.com/SuperClaude-Org/SuperClaude_Framework/issues)提交 Issue。
