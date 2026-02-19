# 附录 D：源码导航图

> **生成模型**：Claude Opus 4.6 (anthropic/claude-opus-4-6)
> **Token 消耗**：输入 ~390k tokens，输出 ~4k tokens（本节）

---

## 关键模块依赖关系图

```
                          ┌──────────────────┐
                          │    src/entry.ts   │
                          │    (进程入口)      │
                          └────────┬─────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
            ┌──────────┐  ┌──────────────┐  ┌─────────┐
            │ CLI 命令  │  │   Gateway    │  │   TUI   │
            │ src/cli/  │  │ src/gateway/ │  │ src/tui/│
            └──────────┘  └──────┬───────┘  └────┬────┘
                                 │               │
                    ┌────────────┼────────┐      │
                    ▼            ▼        ▼      │
            ┌───────────┐ ┌──────────┐ ┌─────┐  │
            │  会话管理  │ │ 通道系统  │ │ HTTP│  │
            │src/sessions│ │src/channels│ │    │  │
            └──────┬────┘ └─────┬────┘ └─────┘  │
                   │            │               │
                   ▼            ▼               │
            ┌──────────────────────────────┐    │
            │       Agent 运行时            │◄───┘
            │     src/agents/              │
            │  ┌─────────┐ ┌───────────┐  │
            │  │ pi-agent │ │ 工具系统   │  │
            │  └────┬────┘ └─────┬─────┘  │
            │       │            │         │
            │  ┌────▼────────────▼──────┐ │
            │  │   LLM Provider 适配器   │ │
            │  │   src/providers/       │ │
            │  └────────────────────────┘ │
            └──────────────────────────────┘
                   │            │
          ┌────────┤            ├────────┐
          ▼        ▼            ▼        ▼
    ┌──────────┐ ┌──────┐ ┌──────────┐ ┌──────────┐
    │ 记忆系统  │ │ 技能  │ │ 沙箱系统  │ │ 配置系统  │
    │src/memory│ │src/  │ │src/agents│ │src/config│
    │          │ │agents│ │/sandbox/ │ │          │
    └──────────┘ │/skills│ └──────────┘ └──────────┘
                 └──────┘
```

## 数据流序列图：一条消息的完整路径

```
用户                Telegram        Gateway         Agent        LLM API        工具
 │                    │               │               │              │            │
 │  发送消息           │               │               │              │            │
 │──────────────────►│               │               │              │            │
 │                    │  webhook      │               │              │            │
 │                    │──────────────►│               │              │            │
 │                    │               │  会话路由       │              │            │
 │                    │               │──────────────►│              │            │
 │                    │               │               │  API 调用     │            │
 │                    │               │               │─────────────►│            │
 │                    │               │               │              │            │
 │                    │               │               │  返回(含工具调用)           │
 │                    │               │               │◄─────────────│            │
 │                    │               │               │              │            │
 │                    │               │               │  执行工具     │            │
 │                    │               │               │─────────────────────────►│
 │                    │               │               │              │            │
 │                    │               │               │  工具结果     │            │
 │                    │               │               │◄─────────────────────────│
 │                    │               │               │              │            │
 │                    │               │               │  再次 API 调用 │            │
 │                    │               │               │─────────────►│            │
 │                    │               │               │              │            │
 │                    │               │               │  最终回复     │            │
 │                    │               │               │◄─────────────│            │
 │                    │               │               │              │            │
 │                    │               │  回复投递       │              │            │
 │                    │               │◄──────────────│              │            │
 │                    │  sendMessage  │               │              │            │
 │                    │◄──────────────│               │              │            │
 │  收到回复           │               │               │              │            │
 │◄──────────────────│               │               │              │            │
```

## 源码目录速查

| 目录 | 章节 | 核心文件 |
|------|------|---------|
| `src/gateway/` | 3-4 章 | `server.ts`, `server-startup.ts`, `protocol/` |
| `src/sessions/` | 5-6 章 | `session-runtime.ts`, `session-store.ts` |
| `src/agents/` | 7-9 章 | `pi-agent.ts`, `context-manager.ts`, `tool-runner.ts` |
| `src/providers/` | 10 章 | `anthropic.ts`, `openai.ts`, `gemini.ts` |
| `src/channels/` | 11-13 章 | `telegram/`, `discord/`, `whatsapp/` |
| `src/agents/tools/` | 14-15 章 | `tool-definitions.ts`, `bash-tools.exec.ts` |
| `src/browser/` | 16 章 | `pw-session.ts`, `cdp.ts`, `chrome.ts` |
| `src/canvas-host/` | 17 章 | `server.ts`, `a2ui.ts` |
| `src/cron/` | 18 章 | `service.ts`, `schedule.ts` |
| `src/gateway/node-*` | 19 章 | `node-registry.ts`, `node-command-policy.ts` |
| `src/memory/` | 20 章 | `manager.ts`, `embeddings.ts`, `hybrid.ts` |
| `src/agents/skills*` | 21 章 | `skills.ts`, `skills-install.ts` |
| `src/hooks/` | 22 章 | `loader.ts`, `hooks.ts` |
| `src/config/` | 23 章 | `config.ts`, `schema.ts`, `zod-schema.ts` |
| `src/security/` | 24 章 | `audit.ts`, `audit-fs.ts` |
| `src/cli/` | 25 章 | `program.ts`, `run-main.ts` |
| `src/media/` | 26 章 | `fetch.ts`, `parse.ts`, `image-ops.ts` |
| `ui/` | 27 章 | `src/app.ts` |
| `apps/` | 28 章 | `macos/`, `ios/`, `android/` |
| `src/daemon/` | 29 章 | `daemon.ts`, `launchd.ts` |
| `src/agents/sandbox/` | 31 章 | `constants.ts`, `tool-policy.ts`, `config.ts` |
| `src/acp/` | 32 章 | `server.ts`, `translator.ts`, `client.ts` |
| `src/tui/` | 33 章 | `tui.ts`, `gateway-chat.ts` |
