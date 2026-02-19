# 附录 E：术语表

> **生成模型**：Claude Opus 4.6 (anthropic/claude-opus-4-6)
> **Token 消耗**：输入 ~395k tokens，输出 ~4k tokens（本节）

---

本书出现的全部专业术语中英文对照与解释。

## A

| 术语 | 英文 | 解释 |
|------|------|------|
| A2UI | Agent-to-UI | Agent 直接向用户推送 HTML 界面并操纵 Web 视图的机制 |
| ACP | Agent Communication Protocol | 标准化的 Agent 间通信协议，类似 LSP |
| Agent | Agent | 具有自主决策能力的 AI 软件实体，能感知环境、使用工具、完成任务 |
| ANSI 转义码 | ANSI Escape Codes | 用于控制终端文字颜色、光标位置等的特殊字符序列 |

## B

| 术语 | 英文 | 解释 |
|------|------|------|
| BM25 | Best Matching 25 | 经典的全文检索评分算法，基于词频和文档长度 |
| Bonjour | Bonjour (mDNS) | Apple 的零配置网络服务发现协议 |

## C

| 术语 | 英文 | 解释 |
|------|------|------|
| Canvas | Canvas | OpenClaw 的可视化工作区，支持 Agent 推送 HTML 内容 |
| CDP | Chrome DevTools Protocol | Chrome 浏览器的调试协议，允许程序化控制浏览器 |
| CLI | Command Line Interface | 命令行界面 |
| 控制平面 | Control Plane | 负责决策和协调的系统组件，区别于承载数据的"数据平面" |
| Cron | Cron | Unix 定时任务调度系统，使用 cron 表达式定义执行时间 |
| 上下文窗口 | Context Window | LLM 单次调用能处理的最大 token 数量 |

## D

| 术语 | 英文 | 解释 |
|------|------|------|
| DM | Direct Message | 直接消息（一对一私信） |
| Docker | Docker | 容器化平台，用于隔离和打包应用 |
| 防抖 | Debounce | 在指定时间内只执行最后一次操作，避免频繁触发 |

## E

| 术语 | 英文 | 解释 |
|------|------|------|
| 嵌入向量 | Embedding | 将文本转换为高维数值向量的技术，用于语义搜索 |
| 事件流 | Event Stream | 服务器主动推送事件给客户端的机制 |

## F

| 术语 | 英文 | 解释 |
|------|------|------|
| Funnel | Tailscale Funnel | Tailscale 的公网入口功能，将服务暴露到互联网 |
| 回退链 | Fallback Chain | 主要选项失败时依次尝试的备选项列表 |

## G

| 术语 | 英文 | 解释 |
|------|------|------|
| Gateway | Gateway | OpenClaw 的中央控制服务器，协调所有通道和 Agent |
| 守护进程 | Daemon | 在后台持续运行的系统服务进程 |

## H

| 术语 | 英文 | 解释 |
|------|------|------|
| 心跳 | Heartbeat | 定期发送的存活信号，用于检测连接是否断开 |
| 钩子 | Hook | 在特定事件发生时自动触发的回调函数 |

## I

| 术语 | 英文 | 解释 |
|------|------|------|
| 幂等键 | Idempotency Key | 用于防止重复操作的唯一标识符 |
| 身份 | Identity | Agent 的人格定义，影响其行为风格和回复方式 |

## J-K

| 术语 | 英文 | 解释 |
|------|------|------|
| JSON5 | JSON5 | JSON 的超集，支持注释、尾逗号、单引号等 |
| JSON Schema | JSON Schema | 用于描述和验证 JSON 数据结构的标准 |

## L

| 术语 | 英文 | 解释 |
|------|------|------|
| LLM | Large Language Model | 大语言模型，如 Claude、GPT |
| Lit | Lit | Google 开发的轻量级 Web Components 框架 |
| Linux Capabilities | Linux Capabilities | 将 root 权限拆分为细粒度权限的机制 |
| LSP | Language Server Protocol | 语言服务器协议，标准化编辑器与语言工具的通信 |

## M

| 术语 | 英文 | 解释 |
|------|------|------|
| mDNS | Multicast DNS | 多播 DNS，用于局域网内的服务发现 |
| MCP | Model Context Protocol | Anthropic 提出的模型上下文协议 |
| Monorepo | Monorepo | 单一代码仓库管理多个项目的工程实践 |
| MVP | Minimum Viable Product | 最小可行产品 |

## N

| 术语 | 英文 | 解释 |
|------|------|------|
| NDJSON | Newline-Delimited JSON | 每行一个 JSON 对象的文本格式 |
| 节点 | Node | OpenClaw 中代表一个设备（手机、电脑等）的远程能力代理 |

## O-P

| 术语 | 英文 | 解释 |
|------|------|------|
| OAuth | OAuth 2.0 | 开放授权协议，允许第三方应用获取用户资源 |
| 提示词注入 | Prompt Injection | 恶意用户通过精心构造的输入操纵 AI Agent 行为的攻击方式 |
| Provider | Provider | LLM 服务提供商（如 Anthropic、OpenAI、Google） |
| PTY | Pseudo-Terminal | 伪终端，模拟真实终端行为的设备 |

## R

| 术语 | 英文 | 解释 |
|------|------|------|
| RPC | Remote Procedure Call | 远程过程调用，允许程序调用另一个进程/机器上的函数 |
| 运行时 | Runtime | 程序执行时的环境和上下文 |

## S

| 术语 | 英文 | 解释 |
|------|------|------|
| 沙箱 | Sandbox | 隔离的执行环境，限制程序的权限和资源访问 |
| Session Key | Session Key | 唯一标识一个会话的字符串，编码了 Agent ID 和会话标识 |
| SOUL | SOUL | OpenClaw 中 Agent 的系统提示和行为定义 |
| 子代理 | Sub-agent | 由主 Agent 动态派生的临时 Agent，用于执行特定任务 |
| SSE | Server-Sent Events | 服务器向客户端单向推送事件的 HTTP 机制 |

## T

| 术语 | 英文 | 解释 |
|------|------|------|
| Tailscale | Tailscale | 基于 WireGuard 的零配置 VPN 服务 |
| Token | Token | LLM 处理文本的最小单位，通常 1 个 token 约 4 个英文字符 |
| TUI | Terminal User Interface | 终端用户界面，在终端中构建类似 GUI 的交互界面 |
| TypeBox | TypeBox | TypeScript 类型系统与 JSON Schema 的桥接库 |

## W

| 术语 | 英文 | 解释 |
|------|------|------|
| WebChat | WebChat | 基于 Web 浏览器的聊天界面 |
| Webhook | Webhook | 服务器主动向客户端 URL 发送 HTTP 请求的机制 |
| WebSocket | WebSocket | 全双工 TCP 通信协议，支持服务器和客户端双向实时通信 |
| 工具调用 | Tool Use / Function Calling | LLM 通过结构化输出请求外部系统执行操作的机制 |

## Z

| 术语 | 英文 | 解释 |
|------|------|------|
| Zod | Zod | TypeScript 优先的运行时类型校验库 |
| 零配置 | Zero-Configuration | 无需手动配置即可自动发现和连接的网络协议 |
