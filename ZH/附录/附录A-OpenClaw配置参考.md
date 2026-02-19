# 附录 A：OpenClaw 配置参考

> **生成模型**：Claude Opus 4.6 (anthropic/claude-opus-4-6)
> **Token 消耗**：输入 ~375k tokens，输出 ~4k tokens（本节）

---

OpenClaw 的配置文件位于 `~/.openclaw/openclaw.json`，使用 JSON5 格式（支持注释和尾逗号）。

## 完整配置示例

```jsonc
{
  // ── Gateway 服务器 ──────────────────────────────────────
  "gateway": {
    "port": 3000,
    "host": "127.0.0.1",
    "mode": "local",         // "local" | "remote"
    "auth": {
      "token": "your-secret-token",
      "password": "your-password"
    },
    "tls": {
      "autoGenerate": false,
      "certPath": "/path/to/cert.pem",
      "keyPath": "/path/to/key.pem"
    },
    "trustedProxies": ["127.0.0.1", "::1"],
    "tailscale": {
      "mode": "off"          // "off" | "serve" | "funnel"
    },
    "discovery": {
      "mdns": "minimal"      // "off" | "minimal" | "full"
    }
  },

  // ── 会话管理 ────────────────────────────────────────────
  "session": {
    "scope": "per-sender",   // "per-sender" | "global"
    "mainKey": "main",
    "store": "~/.openclaw/sessions"
  },

  // ── Agent 配置 ──────────────────────────────────────────
  "agents": {
    "defaults": {
      "workspace": "~/openclaw-workspace",
      "sandbox": {
        "mode": "off"        // "off" | "auto" | "always"
      }
    },
    "list": [
      {
        "id": "main",
        "name": "主助手",
        "default": true,
        "model": "claude-opus-4-6",
        "skills": ["coding-agent", "github"],
        "identity": "你是一个有帮助的 AI 助手。",
        "subagents": {
          "allowAgents": ["*"],
          "model": "claude-sonnet-4-20250514"
        }
      }
    ]
  },

  // ── 模型配置 ────────────────────────────────────────────
  "models": {
    "default": "claude-opus-4-6",
    "fallbacks": ["claude-sonnet-4-20250514"],
    "providers": {
      "anthropic": {
        "apiKey": "${ANTHROPIC_API_KEY}",
        "models": [
          {
            "id": "claude-opus-4-6",
            "cost": { "input": 15, "output": 75, "cacheRead": 1.5, "cacheWrite": 18.75 }
          }
        ]
      },
      "openai": {
        "apiKey": "${OPENAI_API_KEY}"
      }
    }
  },

  // ── 通道配置 ────────────────────────────────────────────
  "channels": {
    "telegram": {
      "botToken": "${TELEGRAM_BOT_TOKEN}",
      "allowedUsers": [123456789],
      "dmPolicy": "pairing"
    },
    "discord": {
      "botToken": "${DISCORD_BOT_TOKEN}",
      "activation": "mention"
    },
    "whatsapp": {
      "phoneNumber": "+1234567890"
    }
  },

  // ── 工具配置 ────────────────────────────────────────────
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["exec", "read", "write", "edit", "image"],
        "deny": ["browser", "canvas", "cron"]
      }
    }
  },

  // ── 记忆系统 ────────────────────────────────────────────
  "memory": {
    "embeddings": {
      "provider": "openai",
      "model": "text-embedding-3-small"
    }
  },

  // ── 钩子系统 ────────────────────────────────────────────
  "hooks": {
    "plugins": []
  }
}
```

## 配置键速查表

| 路径 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `gateway.port` | number | `3000` | WebSocket 服务端口 |
| `gateway.host` | string | `"127.0.0.1"` | 监听地址 |
| `gateway.mode` | string | `"local"` | 运行模式 |
| `gateway.auth.token` | string | — | 认证令牌 |
| `gateway.auth.password` | string | — | 认证密码 |
| `session.scope` | string | `"per-sender"` | 会话范围 |
| `session.mainKey` | string | `"main"` | 主会话键 |
| `agents.list[]` | array | `[]` | Agent 列表 |
| `agents.list[].id` | string | — | Agent 唯一标识 |
| `agents.list[].default` | boolean | `false` | 是否为默认 Agent |
| `agents.list[].model` | string/object | — | 模型配置 |
| `agents.list[].skills` | string[] | — | 启用的技能列表 |
| `agents.list[].workspace` | string | — | 工作空间路径 |
| `agents.list[].sandbox.mode` | string | `"off"` | 沙箱模式 |
| `agents.list[].subagents.allowAgents` | string[] | — | 允许派生的 Agent ID |
| `models.default` | string | `"claude-opus-4-6"` | 默认模型 |
| `models.fallbacks` | string[] | — | 模型回退链 |
| `models.providers` | object | — | 提供商配置 |
| `channels.telegram.botToken` | string | — | Telegram Bot Token |
| `channels.telegram.dmPolicy` | string | `"pairing"` | DM 安全策略 |
| `channels.discord.botToken` | string | — | Discord Bot Token |
| `channels.discord.activation` | string | `"mention"` | 群聊激活方式 |
| `tools.sandbox.tools.allow` | string[] | 见 31.3 节 | 沙箱工具白名单 |
| `tools.sandbox.tools.deny` | string[] | 见 31.3 节 | 沙箱工具黑名单 |
| `memory.embeddings.provider` | string | — | 嵌入向量提供商 |

> 配置文件支持 `${ENV_VAR}` 语法的环境变量替换（详见第 23.5 节）。
