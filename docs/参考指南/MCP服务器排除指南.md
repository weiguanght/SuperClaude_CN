# MCP 服务器故障排除与使用指南

本指南涵盖了 SuperClaude 集成的 Model Context Protocol (MCP) 服务器的安装、配置及运行诊断建议。

## 什么是 MCP 服务器？
MCP 服务器为 Claude Code 提供增强能力，如文档查询 (Context7)、UI 生成 (Magic) 及系统化逻辑推导 (Sequential)。

## 环境要求
- **Node.js 16.0.0 或更高版本**。
- **npm** 包管理器。
- 稳定的网络连接（部分服务器如 Tavily 需要）。

## 常见问题与对策

### 1. Node.js 版本不匹配
- **症状**：提示需要 Node.js 16+。
- **对策**：推荐使用 `nvm` 安装最新稳定版 Node。

### 2. npm 权限问题
- **症状**：出现 `EACCES` 错误。
- **对策**：尝试使用 `pipx` 类似的 `npx` 机制，或通过 `chown` 修复全局 Node 模块目录权限。

### 3. 服务器连接失败 (如 Context7)
- **症状**：显示服务器无法连接。
- **对策**：尝试**完全重启** Claude Code 会话；或运行 `SuperClaude install --components mcp --force` 强制重装。

### 4. 浏览器自动化失败 (Playwright)
- **症状**：提示浏览器未安装。
- **对策**：运行 `npx playwright install` 以补全必要的浏览器二进制文件。

### 5. API Key 缺失
- **症状**：Magic 或 Tavily 等服务器无法返回结果。
- **对策**：确保在环境变量中正确设置了 `MORPH_API_KEY`, `TAVILY_API_KEY` 等秘钥。

## 性能优化建议
- **按需开启**：不要始终使用 `--all-mcp`。在处理简单任务时使用 `--no-mcp` 以节省资源并提升响应速度。
- **定期重启**：在进行大规模代码库分析后，重启会话有助于释放 MCP 服务器可能占用的内存。

---

*大部分 MCP 问题都可以通过确保 Node.js 环境正确及重新安装相应的服务器组件来解决。*
