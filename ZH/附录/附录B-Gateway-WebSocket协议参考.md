# 附录 B：Gateway WebSocket 协议参考

> **生成模型**：Claude Opus 4.6 (anthropic/claude-opus-4-6)
> **Token 消耗**：输入 ~380k tokens，输出 ~4k tokens（本节）

---

## 帧格式

所有 WebSocket 消息使用 JSON 格式，包含三种帧类型：

```typescript
// 请求帧（客户端 → Gateway）
{ "type": "req", "id": "req-001", "method": "chat.send", "params": { ... } }

// 响应帧（Gateway → 客户端）
{ "type": "res", "id": "req-001", "result": { ... } }
// 或错误响应
{ "type": "res", "id": "req-001", "error": "error message" }

// 事件帧（Gateway → 客户端，推送）
{ "type": "event", "event": "chat", "payload": { ... }, "seq": 42 }
```

## 请求方法速查

### 聊天

| 方法 | 参数 | 说明 |
|------|------|------|
| `chat.send` | `sessionKey, message, channel?, deliver?, thinking?, idempotencyKey?` | 发送消息给 Agent |
| `chat.abort` | `sessionKey` | 中止当前 Agent 运行 |

### 会话管理

| 方法 | 参数 | 说明 |
|------|------|------|
| `sessions.list` | `limit?, agentId?` | 列出会话 |
| `sessions.patch` | `key, model?, thinking?, label?` | 修改会话属性 |
| `sessions.reset` | `key` | 重置会话 |
| `sessions.delete` | `key, deleteTranscript?` | 删除会话 |
| `sessions.resolve` | `key? \| label?` | 按键或标签查找会话 |
| `sessions.history` | `key, limit?` | 获取会话历史 |

### Agent

| 方法 | 参数 | 说明 |
|------|------|------|
| `agent` | `sessionKey, message, deliver?, lane?, spawnedBy?` | 直接调用 Agent |
| `agent.wait` | `runId, timeoutMs` | 等待 Agent 运行完成 |
| `agents.list` | — | 列出所有 Agent |

### 配置

| 方法 | 参数 | 说明 |
|------|------|------|
| `config.get` | `key?` | 获取配置 |
| `config.set` | `key, value` | 设置配置 |
| `config.reload` | — | 重载配置文件 |

### 健康与状态

| 方法 | 参数 | 说明 |
|------|------|------|
| `health` | — | 健康检查 |
| `status` | — | 系统状态 |
| `presence` | — | 在线状态 |

### 其他

| 方法 | 参数 | 说明 |
|------|------|------|
| `models.list` | — | 可用模型列表 |
| `skills.list` | — | 已安装技能列表 |
| `events.subscribe` | `stream, filter?` | 订阅事件流 |

## 事件类型

| 事件名 | 触发条件 | Payload 关键字段 |
|--------|---------|-----------------|
| `chat` | 聊天状态变更 | `sessionKey, state(delta/final/aborted/error), message` |
| `agent` | Agent 活动 | `sessionKey, stream(tool/lifecycle), data` |
| `presence` | 在线状态变更 | `host, version, mode` |
| `config` | 配置变更 | `key, value` |
| `session` | 会话变更 | `key, action(created/updated/deleted)` |

## 连接握手

客户端连接后发送 `hello` 请求：

```json
{
  "type": "req",
  "id": "hello-1",
  "method": "hello",
  "params": {
    "protocolVersion": 1,
    "clientName": "webchat",
    "clientVersion": "1.0.0",
    "token": "auth-token",
    "password": "auth-password",
    "capabilities": ["chat", "events"]
  }
}
```

Gateway 响应：

```json
{
  "type": "res",
  "id": "hello-1",
  "result": {
    "ok": true,
    "protocolVersion": 1,
    "serverVersion": "2026.2.1",
    "features": ["chat", "sessions", "agents", "tools"]
  }
}
```
