# MCP-Telnet 工具参考

[English](./TOOLS.md) | 简体中文

本文详细介绍 MCP-Telnet 提供的 Telnet 交互工具。

## 身份管理

### `set_llm_identity`

设置 LLM 的身份信息。该信息用于 Telnet 协商期间标识客户端。通常会自动设置；必要时也可以手动配置。

```text
set_llm_identity，参数：{
  "name": "Claude",
  "version": "3.7 Sonnet",
  "provider": "Anthropic",
  "capabilities": ["text"],
  "metadata": { "additional": "info" }
}
```

参数：

- `name`：LLM 名称（必需）
- `version`：LLM 版本（必需）
- `provider`：LLM 提供方或创建者（必需）
- `capabilities`：能力名称数组（可选）
- `metadata`：其他元数据对象（可选）

### `get_llm_identity`

获取当前身份信息。

```text
get_llm_identity
```

返回包含名称、版本、提供方等信息的当前身份。

## 连接管理

### `connect_telnet`

通过主机和端口，或已保存的连接名称，连接到 Telnet 服务器。仅连接你拥有或获明确授权访问的服务。

```text
connect_telnet，参数：{
  "host": "mud.example.com",
  "port": 4000,
  "saveName": "my-mud"
}
```

使用 SSL/TLS 加密连接：

```text
connect_telnet，参数：{
  "host": "chatmud.example.com",
  "port": 7443,
  "tls": true,
  "saveName": "ChatMUD-secure"
}
```

使用已保存的连接：

```text
connect_telnet，参数：{
  "name": "my-mud"
}
```

参数：

- `host`：主机名或 IP 地址（新建连接时必需）
- `port`：端口号；未指定时默认为 `23`
- `saveName`：保存此连接时使用的名称（可选）
- `name`：要使用的已保存连接名称
- `tls`：是否启用 SSL/TLS 加密（可选，默认为 `false`）

### `disconnect_telnet`

关闭当前 Telnet 连接。

```text
disconnect_telnet
```

### `connection_status`

获取或更新当前连接状态。

```text
connection_status，参数：{
  "toggleContinuousMode": true,
  "defaultDelay": 2
}
```

参数：

- `toggleContinuousMode`：启用或关闭连续模式（可选）
- `defaultDelay`：设置命令之间的默认等待时长，单位为秒（可选）

返回当前连接信息，包括主机、端口和连接状态。

### `list_saved_connections`

列出所有已保存的连接。

```text
list_saved_connections
```

返回已保存连接及其详细信息的数组。

## 命令管理

### `send_command`

向 Telnet 服务器发送命令并返回响应，支持两种等待模式。

**默认模式**：固定等待缓冲区响应（500 毫秒）。

```text
send_command，参数：{
  "command": "look",
  "timeout": 30000,
  "waitAfter": 3
}
```

**`waitFor` 模式**：轮询响应缓冲区，匹配到正则表达式后立即返回。与固定等待相比，预期响应较快出现时可以减少等待时间。

```text
send_command，参数：{
  "command": "look",
  "waitFor": "You can go"
}
```

更多 `waitFor` 模式示例：

```text
send_command，参数：{ "command": "MAP POI", "waitFor": "Enter your selection" }
send_command，参数：{ "command": "WALK Town Square", "waitFor": "You've arrived" }
send_command，参数：{ "command": "CREDITS", "waitFor": "credits" }
send_command，参数：{ "command": "SALVAGE", "waitFor": "begin scanning|No debris" }
```

参数：

- `command`：要发送的命令（必需）
- `timeout`：超时时间，单位为毫秒（普通模式默认 `30000`；`waitFor` 模式默认 `2000`）
- `waitAfter`：发送命令后等待的秒数（默认 `0`；设置 `waitFor` 后忽略）
- `waitFor`：要在响应缓冲区中等待匹配的正则表达式（可选，不区分大小写）。设置后，匹配成功即返回，不再等待固定时长。如果超时前未匹配到模式，仍会返回已收集的缓冲区内容，并提示未匹配到该模式。

### `get_buffer`

获取当前响应缓冲区，不发送命令。

```text
get_buffer
```

返回当前缓冲区内容。

### `wait`

等待指定秒数后再继续。

```text
wait，参数：{
  "seconds": 5
}
```

参数：

- `seconds`：等待秒数，范围为 `0` 到 `60`，默认值为 `1`

### `sequence_commands`

按顺序发送多条命令，并为每条命令设置固定等待时间或模式匹配等待条件。

**固定等待时间：**

```text
sequence_commands，参数：{
  "commands": [
    { "command": "north", "waitAfter": 2 },
    { "command": "look", "waitAfter": 1 },
    { "command": "get treasure", "waitAfter": 0 }
  ],
  "timeout": 60000
}
```

**按模式等待（可更快响应）：**

```text
sequence_commands，参数：{
  "commands": [
    { "command": "BUY 14", "waitFor": "Enter your selection" },
    { "command": "5", "waitFor": "Continue with purchase" },
    { "command": "yes", "waitFor": "credits" }
  ]
}
```

**混合模式：**

```text
sequence_commands，参数：{
  "commands": [
    { "command": "north", "waitFor": "You can go" },
    { "command": "look", "waitAfter": 1 },
    { "command": "get treasure", "waitFor": "You pick up" }
  ]
}
```

参数：

- `commands`：命令对象数组，每个对象包含：
  - `command`：要发送的命令（必需）
  - `waitAfter`：发送该命令后等待的秒数（默认 `1`；设置 `waitFor` 后忽略）
  - `waitFor`：继续发送下一条命令前要匹配的正则表达式（可选，优先于 `waitAfter`）
- `timeout`：未设置 `waitFor` 的命令的默认超时时间，单位为毫秒（默认 `30000`）。设置了 `waitFor` 的命令，超时时间取此值与 `2000` 毫秒中的较小值。

## 日志与记忆

### `get_session_logs`

按条件筛选并获取会话日志。

```text
get_session_logs，参数：{
  "connectionName": "my-mud",
  "startDate": "2023-01-01T00:00:00Z",
  "endDate": "2023-01-31T23:59:59Z",
  "sessionId": "a1b2c3d4-e5f6-g7h8-i9j0"
}
```

参数：

- `connectionName`：按连接名称筛选（可选）
- `startDate`：按开始时间筛选，使用 ISO 日期格式（可选）
- `endDate`：按结束时间筛选，使用 ISO 日期格式（可选）
- `sessionId`：按特定会话 ID 筛选（可选）

### `update_connection_memory`

保存或更新某个连接的记忆/备注。

```text
update_connection_memory，参数：{
  "connectionName": "my-mud",
  "memory": "市场广场在主门以北"
}
```

参数：

- `connectionName`：已保存连接的名称；省略时使用当前连接
- `memory`：要保存的文本（必需）

### `get_connection_memory`

获取某个连接已保存的记忆。

```text
get_connection_memory，参数：{
  "connectionName": "my-mud"
}
```

参数：

- `connectionName`：已保存连接的名称；省略时使用当前连接

## 进阶用法

### 组合使用工具

可以按顺序组合工具完成交互：

1. 连接到已获授权访问的服务器。
2. 发送初始命令。
3. 获取缓冲区，查看服务器是否正在等待输入。
4. 如需身份验证，由你自行在安全的带外通道输入凭据；不要将凭据发送给云端 LLM。
5. 等待服务器响应。
6. 继续发送后续命令。

### 错误处理

工具会在发生问题时返回错误信息。常见错误包括：

- 连接错误（服务器不可达）
- 身份验证失败
- 超时（服务器未及时响应）
- 命令失败

请检查工具返回的成功状态，并据此处理错误。

### MTTS 协议说明

连接协商期间，MCP-Telnet 使用 MTTS（Mud Terminal Type Standard，MUD 终端类型标准）发送以下信息：

```text
MTTS 1 MCP-Telnet/[VERSION] [CLIENT_NAME]/[CLIENT_VERSION] ([PROVIDER]) 13
```

标志值 `13` 表示：

- UTF-8 文本编码（`4`）
- ANSI 颜色支持（`1`）
- 256 色支持（`8`）

服务器可据此选择合适的格式，并识别客户端信息。
