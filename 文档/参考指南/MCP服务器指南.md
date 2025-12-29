# MCP 服务器故障排除指南 (MCP Server Troubleshooting Guide)

**核心关注点**：Model Context Protocol (MCP) 服务器为 Claude 提供增强能力，如文档查询 (Context7)、UI 生成 (Magic) 和高级推理 (Sequential)。本指南涵盖了所有 MCP 服务器的安装、配置和运行故障排除。

**特定服务器方案**：每个 MCP 服务器都有其独特的要求和常见的失败模式。本指南针对每种服务器类型提供了专门的解决方案及通用 MCP 排除策略。

---

## MCP 服务器概览

### 可用服务器列表
- **Context7**: 官方文档查询与框架模式。
- **Sequential**: 多步推理与复杂分析。
- **Magic**: 基于 21st.dev 模式的现代 UI 组件生成。
- **Playwright**: 浏览器自动化与 E2E 测试。
- **Morphllm**: 基于模式的代码编辑与 Token 优化。
- **Serena**: 语义化代码理解与项目长期记忆。

### 服务器运行要求
- **Node.js**: 16.0.0 或更高版本。
- **npm/yarn**: 包管理工具。
- **网络连接**: 部分服务器需要稳定的网络环境。
- **系统内存**: 建议 2GB+。

---

## 安装与配置问题

### Node.js 和 npm 相关问题

#### 问题：Node.js 版本不兼容
**错误消息**：`ERROR: MCP servers require Node.js 16+ but found Node.js 14.x`

**解决方案（使用 nvm 升级）**：
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
nvm install node
nvm use node
```

#### 问题：npm 权限问题
**错误消息**：`ERROR: EACCES: permission denied, access '/usr/local/lib/node_modules'`

**解决方案（配置用户级 npm 目录）**：
```bash
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.profile
source ~/.profile
```

---

## 连接与通信问题

### 服务器连接失败

#### 问题：Context7 服务器无法连接
1. **检查版本**：`node --version`（需 16+）。
2. **强制重装**：`python3 -m SuperClaude install --components mcp --force`。
3. **重启会话**：完全重启 Claude Code 终端，这会强制所有 MCP 服务器重启。

#### 问题：MCP 服务器通信超时 (Timeout)
1. **简化任务**：将复杂的单次请求拆分为多个小步骤。
2. **检查内存**：确保系统有足够的剩余内存。
3. **禁用特定服务器**：在操作时加上 `--no-mcp` 标志。

---

## 特定服务器故障排除

### Context7 (文档查询)
- **现象**：已激活但无法返回文档。
- **对策**：运行 `npm update -g @context7/mcp-server` 更新到最新版；确保网络能访问相关的官方文档域名。

### Sequential (顺序推理)
- **现象**：内部错误或响应极慢。
- **对策**：重启会话以释放内存；如果问题持续，尝试减少 `--think-hard` 标志的使用。

### Magic (UI 生成)
- **现象**：不产生预期的 UI 输出。
- **对策**：在请求时明确包含 "UI"、"Component" 等关键词；确保提供了该服务器所需的 API Key（如果适用）。

### Playwright (自动化)
- **现象**：浏览器未安装。
- **对策**：手动运行 `npx playwright install` 以补全所需的浏览器二进制文件。

### Serena (项目记忆)
- **现象**：无法在会话间保留项目上下文。
- **对策**：确保在退出前正确执行了保存操作；检查 `~/.claude/sessions/` 目录的读写权限。

---

## 性能与优化

### 响应缓慢
- **对策**：采用**选择性激活**。不要次次都加 `--all-mcp`，仅在需要时添加特定的服务器标志位（如 `--c7`）。

### 内存泄漏
- **对策**：在进行长时间的高强度开发后，建议完全退出并重启 Claude Code，以彻底终结 Node.js 僵尸进程。

---

## 紧急恢复流程

### 完全重置所有 MCP 服务器
如果您发现所有 MCP 服务器都失效了，请按照以下步骤操作：

1. **备份现有配置**：`cp -r ~/.claude ~/.claude.mcp.backup`。
2. **卸载已知服务器**：
   ```bash
   npm uninstall -g @context7/mcp-server @sequential/mcp-server @magic/ui-generator
   ```
3. **清理缓存**：`npm cache clean --force`。
4. **重新安装**：`python3 -m SuperClaude install --components mcp --force`。
5. **验证**：通过 `npm list -g` 确认安装状态。

---

## 故障排除验证清单
- [ ] `node --version` 显示 16.0.0+。
- [ ] `npm list -g | grep mcp` 能看到对应的包。
- [ ] 在 Claude Code 中发起特定请求（如查阅文档），能够得到正常响应且无错误提示。

---

**核心理念**：排除 MCP 故障时，请遵循以下优先级：**Node.js 环境 → npm 权限 → 个体服务器重装 → Claude Code 整体重启**。绝大多数问题都可以通过确保 Node.js 正确配置和重新安装服务器包来解决。
