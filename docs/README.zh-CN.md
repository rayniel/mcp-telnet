# MCP-Telnet

[English](./README.md) | 简体中文

连接大型语言模型（LLM）与 Telnet 服务的桥接工具。

## MCP-Telnet 是什么？

MCP-Telnet 是一个 [MCP](https://modelcontextprotocol.io/) 服务器，可与基于 Telnet 的服务交互。它负责处理 Telnet 通信的复杂细节，便于将 LLM 接入以下服务和场景：

- 网络设备、旧式系统及其他基于文本的服务
- 旧式系统和网络设备管理
- 邮件服务器及其他文本交互界面
- 较早期的网络协议与系统
- 在可通过网络访问的设备上执行日常任务

请仅连接你拥有或获明确授权管理的服务。Telnet 本身不加密；如服务支持，请使用 TLS。不要向云端 LLM 提供密码或其他凭据。

## 主要功能

- 实现了大部分 Telnet 协议功能，并支持 MTTS 客户端标识
- 可靠连接、自动重连和状态持久化
- 可灵活调整命令发送间隔
- 按连接保存备注的记忆功能
- 支持筛选的完整会话日志
- 可通过 Telnet 访问的交互式 CLI（端口 9000）
- 自动发送保活数据包以维持连接
- 丰富的配置选项

## 安装与快速开始

使用 Claude Desktop 是最简单的方式。在 `claude-desktop-config.json` 中添加以下配置：

```json
{
  "mcpServers": {
    "telnet": {
      "command": "npx",
      "args": ["-y", "github:distantorigin/mcp-telnet"]
    }
  }
}
```

无需单独安装；Claude Desktop 会在需要时自动启动该服务。

### 基本用法

1. 请求 Claude 连接到你有权访问的 Telnet 服务：
   ```
   连接到我有权访问的 monitor.example.net:2323，读取欢迎信息并告诉我服务端提示了什么；如果需要登录，请暂停并让我自行输入凭据。
   ```

2. 用自然语言发送命令：
   ```
   读取服务端当前响应，总结设备状态和需要关注的告警。
   ```

3. 使用更高级的功能：
   ```
   请在此连接的记忆中保存：“维护窗口为每周日 02:00；执行变更前需确认。”
   ```

更多用法示例请参阅[提示词示例](./EXAMPLE_PROMPTS.zh-CN.md)。

## 工具概览

MCP-Telnet 提供多种用于管理 Telnet 连接的工具，以下是最常用的几种：

- **连接管理**：
  ```
  connect_telnet，参数：{ "host": "monitor.example.net", "port": 2323 }
  ```

- **发送命令并按模式等待响应**：
  ```
  send_command，参数：{ "command": "status", "waitFor": "Status:|>" }
  ```

- **发送命令并等待固定时长**：
  ```
  send_command，参数：{ "command": "show alerts", "waitAfter": 2 }
  ```

- **按顺序发送多条命令**：
  ```
  sequence_commands，参数：{
    "commands": [
      { "command": "show interfaces status", "waitFor": "Interface|Port|>" },
      { "command": "show alerts", "waitAfter": 2 }
    ]
  }
  ```

工具及参数的完整说明见[工具参考](./TOOLS.zh-CN.md)。

## 命令间隔与防止过快发送

网络设备、旧式系统及其他 Telnet 服务通常需要时间处理命令。合理设置间隔有助于避免触发服务器的频率限制，也能避免 LLM 过于频繁地向服务发送命令。

MCP-Telnet 支持以下等待方式：

1. **按模式等待（`waitFor`）**：使用正则表达式匹配服务端输出；匹配后立即返回，是响应最快的方式。
2. **命令后等待**：在特定命令后等待固定时长。
3. **命令序列**：为多条命令设置精确的等待时间或逐条指定匹配模式。
4. **默认间隔**：为所有命令设置统一的默认等待时间。
5. **手动等待**：在操作之间显式暂停。

合理设置间隔有助于避免服务器限流、因发送过快而断开连接，以及因命令过于频繁而影响服务性能。请遵守目标服务的使用规则。

## 连续模式

连续模式是一项用于在持续交互中保留上下文的功能。启用后，MCP-Telnet 会在每次服务端响应末尾添加提示，表明可以继续发送命令：

```
connection_status，参数：{ "toggleContinuousMode": true }
```

连续模式可以：

- 模拟与 Telnet 服务持续交互的体验
- 在多条命令之间保留上下文，让交互更自然
- 明确提示服务端何时完成响应
- 提示何时可以继续输入，让交互更直观

这并不是持久化的服务端会话；Telnet 响应仍然是分段返回的。连续模式让逐条发送命令的交互更接近直接使用 Telnet 的连续体验，适用于交互式终端操作和较长时间的故障排查。

可随时关闭连续模式：

```
connection_status，参数：{ "toggleContinuousMode": false }
```

## 配置

MCP-Telnet 默认将配置保存在用户主目录下：

- `~/.mcp-telnet/config.json`：连接配置和服务器设置
- `~/.mcp-telnet/llm-identity.json`：LLM 身份信息
- `~/.mcp-telnet/logs/`：会话日志目录

可用以下环境变量覆盖部分默认设置：

- `MCP_TELNET_DATA_DIR`：指定其他数据目录
- `CLI_PORT`：指定其他 CLI 端口（默认 `9000`）
- `NODE_ENV`：设置为 `production` 时启用生产环境设置

## 注意事项

- 登录流程使用精确文本匹配；服务端响应不符合预期时可能超时。
- 会话日志可能随使用时间增长；目前没有日志轮转功能。
- 输出中可能包含 ANSI 控制码。
- **不要将凭据以明文形式提供给云端 LLM。** 如需登录，可在 Claude 发起连接后，通过端口 9000 的 Telnet 服务器自行在带外输入凭据。
- Telnet 流量默认不加密。不要在不可信网络中通过未加密的 Telnet 发送敏感信息；优先使用 TLS 或更安全的管理协议。

## 贡献

MCP-Telnet 最初是一个原型项目，作者日常使用较少，因此作为个人副项目维护，主要由生成式 AI 辅助编写。维护节奏可能不固定；作者也可能因对生成式 AI 伦理问题的看法而随时下架项目。作者会尽力回应并处理问题和拉取请求，但不作保证。