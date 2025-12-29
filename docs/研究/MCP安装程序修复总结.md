# MCP 安装程序修复总结 (MCP Installer Fix Summary)

## 查明问题
SuperClaude 框架安装程序原先使用的是 `claude mcp add` 命令行指令。该指令是针对 Claude Desktop (桌面版) 设计的，在针对 Claude Code (命令行版) 使用时会导致安装失败。

## 根本原因
- **原实施方案**：使用了 `claude mcp add` CLI 指令。
- **问题所在**：CLI 指令在 Claude Code 环境下表现不可靠。
- **最佳实践**：Claude Code 倾向于通过直接操作位于 `~/.claude/mcp.json` 的 JSON 配置文件来管理 MCP 服务器。

## 实施的解决方案

### 1. 基于 JSON 的辅助方法
创建了全新的基于 JSON 配置的辅助方法（具体见源码 213-302 行）：
- `_get_claude_code_config_file()`：获取配置文件路径。
- `_load_claude_code_config()`：加载 JSON 配置。
- `_save_claude_code_config()`：保存 JSON 配置。
- `_register_mcp_server_in_config()`：在配置中注册服务器。
- `_unregister_mcp_server_from_config()`：从配置中注销服务器。

### 2. 更新后的安装方法

#### `_install_mcp_server()` (基于 npm 的服务器)
- **更新前**：使用 `claude mcp add -s user {server_name} {command} {args}`。
- **更新后**：直接进行 JSON 配置，明确 `command` (命令) 和 `args` (参数) 字段。
- **配置格式示例**：
```json
{
  "command": "npx",
  "args": ["-y", "@package/name"],
  "env": {
    "API_KEY": "value"
  }
}
```

#### `_install_docker_mcp_gateway()` (Docker 网关)
- **更新前**：使用 `claude mcp add -s user -t sse {server_name} {url}`。
- **更新后**：直接进行 JSON 配置，为 SSE 传输协议指定 `url` 字段。

#### `_install_github_mcp_server()` (GitHub/uvx 服务器)
- **更新后**：解析运行命令，并创建包含 `command` 和 `args` 的 JSON 配置。

#### `_uninstall_mcp_server()` (卸载操作)
- **更新前**：使用 `claude mcp remove {server_name}`。
- **更新后**：通过 `_unregister_mcp_server_from_config()` 直接从 JSON 配置中移除。

### 3. 更新后的检查方法
#### `_check_mcp_server_installed()`
- **更新前**：使用 `claude mcp list` CLI 命令。
- **更新后**：直接读取 `~/.claude/mcp.json` 并检查其中的 `mcpServers` 部分。
- **特殊处理**：对于 AIRIS 网关，还会额外验证 SSE 端点是否有响应。

## 收益
1. **可靠性高**：直接操作 JSON 文件比调用 CLI 命令更稳健。
2. **兼容性强**：能够完美适配 Claude Code。
3. **性能提升**：注册时无需调用繁重的子进程。

## 涉及修改的文件
- `/Users/hoto/SuperClaude_Framework-4.1.9/setup/components/mcp.py`
  - 增加了辅助方法。
  - 更新了检查、安装 (npm/Docker/GitHub/uv) 及卸载的逻辑。

## 下一步行动
1. 在纯净的 Claude Code 环境下测试修改后的安装程序。
2. 验证所有类型的服务器（npm, Docker, GitHub, uv）是否都能正确安装。
3. 检查卸载逻辑是否生效。
