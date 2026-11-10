# MCP-Telnet

[English](./README.md) | 简体中文

连接大型语言模型（LLM）与 Telnet 服务的桥接工具。

## MCP-Telnet 是什么？

MCP-Telnet 是一个 [MCP](https://modelcontextprotocol.io/) 服务器，可与基于 Telnet 的服务交互。它负责处理 Telnet 通信的底层细节，可用于：

- MUD（多人地牢游戏）等文字冒险游戏
- 经授权的旧式系统和网络设备管理
- 邮件服务器及其他基于文本的接口
- 历史网络协议与系统
- 对有权访问的网络设备执行日常任务

请仅连接你拥有或获明确授权管理的服务。Telnet 本身不加密；如服务支持，请使用 TLS。不要向云端 LLM 提供密码或其他凭据。

## 主要功能

- 较完整的 Telnet 协议实现，支持 MTTS 客户端标识
- 可靠连接、自动重连和状态持久化
- 可调节命令间隔
- 按连接保存备注的记忆功能
- 支持筛选的会话日志
- 可通过 Telnet（默认端口 9000）使用的交互式 CLI
- 自动发送保活数据包
- 多种配置选项

## 安装与快速开始

在 Claude Desktop 的 `claude-desktop-config.json` 中添加以下配置：

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

1. 请求连接到你有权访问的 Telnet 服务：
   ```
   连接到 mud.example.com 的 4000 端口，以访客身份帮我探索。
   ```

2. 用自然语言发送命令：
   ```
   查看当前房间，并总结可能值得留意的物品。
   ```

3. 使用记忆功能保存备注：
   ```
   请在此连接的记忆中保存：“魔法商店位于城镇广场以西。”
   ```

更多示例请参阅[提示词示例](./EXAMPLE_PROMPTS.zh-CN.md)。

## 工具概览

常用工具如下：

- **管理连接**
  ```
  connect_telnet，参数：{ "host": "mud.example.com", "port": 4000 }
  ```

- **发送命令并等待匹配内容**
  ```
  send_command，参数：{ "command": "look", "waitFor": "You can go" }
  ```

- **发送命令并等待固定时长**
  ```
  send_command，参数：{ "command": "look", "waitAfter": 2 }
  ```

- **按顺序发送多条命令**
  ```
  sequence_commands，参数：{
    "commands": [
      { "command": "north", "waitFor": "You can go" },
      { "command": "look", "waitAfter": 0 }
    ]
  }
  ```

工具及参数的完整说明见[工具参考](./TOOLS.zh-CN.md)。

## 命令间隔与防止过快发送

MUD 和其他 Telnet 服务通常需要时间处理命令。合理设置间隔也能避免触发服务器的频率限制，减少对其他用户的影响。

MCP-Telnet 支持以下等待方式：

1. **按模式等待（`waitFor`）**：使用正则表达式匹配服务端输出；匹配后立即返回。
2. **命令后等待**：在特定命令后等待固定时长。
3. **命令序列**：为每条命令设置等待时间或匹配模式。
4. **默认间隔**：为所有命令设置统一的默认等待时间。
5. **手动等待**：在操作之间显式暂停。

合理设置间隔有助于避免服务器限流、因发送过快而断开连接，以及对服务或其他用户造成不必要的负担。请遵守目标服务的使用规则。

## 连续模式

启用连续模式后，MCP-Telnet 会在每次服务端响应末尾添加提示，表明可以继续发送命令：

```
connection_status，参数：{ "toggleContinuousMode": true }
```

连续模式可以：

- 让交互更接近持续对话
- 在多条命令之间保留上下文
- 明确提示服务端何时完成响应、何时可以继续输入

这并不是持久化的服务端会话；Telnet 响应仍然是分段返回的。该模式适用于 MUD、互动小说和持续故障排查等场景。

可随时关闭：

```
connection_status，参数：{ "toggleContinuousMode": false }
```

## 配置

默认情况下，配置文件保存在用户主目录下：

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

MCP-Telnet 最初是一个原型项目，作者日常使用较少，因此作为个人副项目维护，维护节奏可能不固定；项目也可能因作者对生成式 AI 伦理问题的看法而随时下架。欢迎提交问题和拉取请求，但不保证一定能及时处理。
