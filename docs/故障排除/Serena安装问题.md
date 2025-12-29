# Serena MCP 安装故障排除 (Troubleshooting)

## 常见问题与解决方案

### 问题: "Failed to spawn: serena" 错误

**症状:**
```
error: Failed to spawn: `serena`
Caused by: No such file or directory (os error 2)
```

**根因:**
SuperClaude 安装程序错误地配置了 `uv run serena` 而非 `uvx`。

**解决方案:**

1. **移除现有损坏的安装:**
   ```bash
   claude mcp remove serena
   ```

2. **使用正确的 uvx 方式安装:**
   ```bash
   uvx --from git+https://github.com/oraios/serena serena --help
   ```

3. **重新注册到 Claude CLI:**
   ```bash
   claude mcp add serena -- uvx --from git+https://github.com/oraios/serena serena start-mcp-server --context ide-assistant
   ```

4. **验证安装:**
   ```bash
   claude mcp list
   ```

### 问题: uv 与 uvx 的混淆

**区别:**
- `uv run serena`: 从本地项目依赖运行 serena (若本地未安装则失败)。
- `uvx ... serena`: 直接从 GitHub 仓库运行 serena。

**正确用法:**
始终对 Serena 使用 `uvx`，因为它是设计为与远程 GitHub 仓库配合使用的。

### GitHub Codespace 特定问题

**问题: UV 安装方式**
如果使用 curl 安装 UV，请确保 `uvx` 可用 (`uvx --version`)。如果不可用，请使用 `pip install uv` 安装。

### 验证步骤

1. **检查 MCP 连接:** `claude mcp list`
2. **功能测试:** 启动 Claude Code 验证 Serena 是否出现在可用服务器列表中。
3. **检查日志:** 查看 `~/.cache/claude-cli-nodejs/*/mcp-logs-serena/latest.txt`。

### 手动配置 (`~/.claude.json`)

如果自动安装失败，可手动配置：

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

### 获取帮助
1. 查看 [Serena 文档](https://github.com/oraios/serena)
2. 验证 uvx 安装: `uvx --version`
3. 向 [SuperClaude Framework](https://github.com/SuperClaude-Org/SuperClaude_Framework/issues) 报告问题
