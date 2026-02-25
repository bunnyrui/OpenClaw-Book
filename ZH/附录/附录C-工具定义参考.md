# 附录 C：工具定义参考

> **生成模型**：Claude Opus 4.6 (anthropic/claude-opus-4-6)
> **Token 消耗**：输入 ~385k tokens，输出 ~3k tokens（本节）

---

OpenClaw 内置的所有工具定义。每个工具通过 JSON Schema 定义输入参数。

## 文件系统工具

| 工具名 | 说明 | 关键参数 |
|--------|------|---------|
| `read` | 读取文件内容 | `filePath, offset?, limit?` |
| `write` | 写入文件 | `filePath, content` |
| `edit` | 编辑文件（查找替换） | `filePath, oldString, newString, replaceAll?` |
| `apply_patch` | 应用 unified diff 补丁 | `filePath, patch` |

## Shell 工具

| 工具名 | 说明 | 关键参数 |
|--------|------|---------|
| `exec` | 执行 Shell 命令 | `command, workdir?, timeout?` |
| `process` | 进程管理（列出/终止） | `action, pid?` |

## 浏览器工具

| 工具名 | 说明 | 关键参数 |
|--------|------|---------|
| `browser` | 浏览器自动化 | `action(navigate/click/type/screenshot/...), url?, selector?, text?` |

## 会话工具

| 工具名 | 说明 | 关键参数 |
|--------|------|---------|
| `sessions_spawn` | 派生子 Agent | `task, agentId?, model?, label?, cleanup?` |
| `sessions_list` | 列出会话 | `limit?` |
| `sessions_history` | 查看会话历史 | `sessionKey, limit?` |
| `sessions_send` | 向会话发送消息 | `sessionKey, message` |
| `session_status` | 查看会话状态 | `sessionKey?` |

## 记忆工具

| 工具名 | 说明 | 关键参数 |
|--------|------|---------|
| `memory_search` | 搜索记忆 | `query, limit?` |
| `memory_save` | 保存记忆 | `key, content` |

## 多媒体工具

| 工具名 | 说明 | 关键参数 |
|--------|------|---------|
| `image` | 图像处理 | `action, path?, url?` |

## Canvas 工具

| 工具名 | 说明 | 关键参数 |
|--------|------|---------|
| `canvas` | A2UI 操作 | `action(push/reset/eval/snapshot), html?, js?` |

## 通道工具

| 工具名 | 说明 | 关键参数 |
|--------|------|---------|
| `message` | 发送消息到通道 | `channel, to, message, threadId?` |
| `telegram` | Telegram 特定操作 | `action, chatId?, ...` |
| `discord` | Discord 特定操作 | `action, channelId?, ...` |

## 系统工具

| 工具名 | 说明 | 关键参数 |
|--------|------|---------|
| `cron` | 定时任务管理 | `action(create/list/delete), expression?, prompt?` |
| `nodes` | 节点设备控制 | `command, nodeId?, ...` |

## 沙箱默认策略

```
允许: exec, process, read, write, edit, apply_patch, image,
      sessions_list, sessions_history, sessions_send,
      sessions_spawn, session_status

禁止: browser, canvas, nodes, cron, gateway,
      telegram, discord, slack, whatsapp, signal, ...
```

> 详细的 JSON Schema 定义可在 OpenClaw 源码的 `src/agents/tools/` 和 `src/gateway/protocol/schema/` 目录中找到。
