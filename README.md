# 《深入 OpenClaw》—— 目录与纲要

> **生成模型**：Claude Opus 4.6 (anthropic/claude-opus-4-6)
> **Token 消耗**：输入 ~120,000 tokens，输出 ~8,000 tokens
> **生成日期**：2026-02-17

---

## 前言

- 本书的写作动机与目标读者
- OpenClaw 项目概况：一个运行在自己设备上的个人 AI 助手
- 本书的组织结构与阅读建议
- 源码版本说明（基于 `v2026.2.x` 分支）
- 预备知识：TypeScript 基础、Node.js 运行时概念、WebSocket 协议、基本的操作系统知识

---

## 第一部分：全局概览

### 第 1 章 认识 OpenClaw

- 1.1 什么是 OpenClaw：个人 AI 助手的设计哲学
  - 1.1.1 "本地优先"（Local-first）理念与传统 SaaS 的对比
  - 1.1.2 单用户设计（Single-user）：为什么不做多租户
  - 1.1.3 多通道统一收件箱：WhatsApp / Telegram / Slack / Discord / Signal / iMessage / WebChat 等
- 1.2 OpenClaw 的核心架构总览
  - 1.2.1 架构图解：消息通道 → Gateway 控制平面 → Pi Agent 运行时 → 工具执行
  - 1.2.2 关键子系统一览：Gateway、Agent、Channel、Tools、Skills、Memory、Canvas
  - 1.2.3 数据流全景：一条消息从用户发出到 AI 回复的完整路径
- 1.3 技术栈全景
  - 1.3.1 TypeScript + Node.js 22+：语言与运行时选型
  - 1.3.2 pnpm Monorepo 工程结构（`pnpm-workspace.yaml`）
  - 1.3.3 构建工具链：tsdown / oxlint / oxfmt / vitest
  - 1.3.4 原生应用：Swift（macOS/iOS）、Kotlin（Android）
- 1.4 项目目录结构详解
  - 1.4.1 `src/` — 核心源码（69 个子目录）
  - 1.4.2 `apps/` — 原生客户端应用（macOS / iOS / Android）
  - 1.4.3 `ui/` — Web 控制台 UI（Lit + Vite）
  - 1.4.4 `extensions/` — 可选通道扩展（31 个扩展）
  - 1.4.5 `packages/` — 内部共享包
  - 1.4.6 `skills/` — 内置技能（52 个技能）
  - 1.4.7 `docs/` — 官方文档源码

### 第 2 章 快速上手与开发环境

- 2.1 环境准备
  - 2.1.1 Node.js 22+、pnpm 安装
  - 2.1.2 从源码克隆与构建（`pnpm install && pnpm build`）
  - 2.1.3 UI 构建（`pnpm ui:build`）
- 2.2 引导向导（Onboarding Wizard）
  - 2.2.1 `openclaw onboard` 命令的源码入口（`src/wizard/onboarding.ts`）
  - 2.2.2 向导流程：Gateway 配置 → 模型选择 → 通道连接 → 技能安装
  - 2.2.3 交互式提示的实现：Clack Prompts 库
- 2.3 启动 Gateway 守护进程
  - 2.3.1 前台启动 vs 守护进程模式（`src/daemon/`）
  - 2.3.2 launchd / systemd 服务安装的源码分析
- 2.4 开发循环
  - 2.4.1 `pnpm gateway:watch` — 文件监听与热重载
  - 2.4.2 测试体系：vitest 单元测试 / e2e 测试 / live 测试 / Docker 测试
  - 2.4.3 代码质量：oxlint 类型感知 Lint + oxfmt 格式化

---

## 第二部分：Gateway 控制平面

### 第 3 章 Gateway 服务器架构

- 3.1 Gateway 的角色与设计目标
  - 3.1.1 "控制平面"（Control Plane）的含义
    > **衍生解释**：控制平面是网络工程中的概念，指负责决策和协调的组件，区别于承载实际数据传输的"数据平面"。在 OpenClaw 中，Gateway 就是控制平面——它不直接承载 AI 对话，而是协调所有通道、客户端和 Agent 之间的通信。
  - 3.1.2 单 Gateway 实例约束：为什么每台主机只运行一个 Gateway
  - 3.1.3 Gateway 的职责边界：通道管理、会话管理、工具路由、事件分发
- 3.2 服务器启动流程源码分析
  - 3.2.1 入口文件 `src/entry.ts`：进程标题设置、实验性警告抑制、CLI 分发
  - 3.2.2 Gateway CLI 启动（`src/cli/gateway-cli.ts`）
  - 3.2.3 服务器启动序列（`src/gateway/server-startup.ts`）
  - 3.2.4 服务器实现类（`src/gateway/server.ts` 与 `src/gateway/server.impl.ts`）
- 3.3 WebSocket 服务器实现
  - 3.3.1 WebSocket 传输层：`ws` 库的使用与配置
  - 3.3.2 WebSocket 运行时管理（`src/gateway/server-ws-runtime.ts`）
  - 3.3.3 连接生命周期：connect 握手 → 认证 → 事件订阅 → 断开
  - 3.3.4 帧协议格式：`{type:"req", id, method, params}` / `{type:"res"}` / `{type:"event"}`
- 3.4 HTTP 层
  - 3.4.1 HTTP 服务器搭建（`src/gateway/server-http.ts`）
  - 3.4.2 OpenAI 兼容 HTTP 接口（`src/gateway/openai-http.ts`）
  - 3.4.3 Open Responses API（`src/gateway/openresponses-http.ts`）
  - 3.4.4 工具调用 HTTP 接口（`src/gateway/tools-invoke-http.ts`）

### 第 4 章 Gateway 协议与类型系统

- 4.1 协议设计哲学
  - 4.1.1 为什么选择 WebSocket 而非 REST
  - 4.1.2 请求-响应 + 服务器推送事件的混合模式
  - 4.1.3 幂等键（Idempotency Key）机制防止重复操作
- 4.2 TypeBox 类型模式（Schema）
  > **衍生解释**：TypeBox 是一个用 TypeScript 定义 JSON Schema 的库。它通过 TypeScript 的类型系统在编译期生成 JSON Schema，实现了"一次定义，类型安全 + 运行时验证"的双重保障。
  - 4.2.1 协议 Schema 定义（`src/gateway/protocol/schema/`）
  - 4.2.2 从 TypeBox 生成 JSON Schema（`scripts/protocol-gen.ts`）
  - 4.2.3 从 JSON Schema 生成 Swift 模型（`scripts/protocol-gen-swift.ts`）
  - 4.2.4 协议一致性校验（`pnpm protocol:check`）
- 4.3 核心方法与事件
  - 4.3.1 请求方法一览（`src/gateway/server-methods-list.ts`）
  - 4.3.2 `connect` — 握手与认证
  - 4.3.3 `agent` / `agent.wait` — 发起 AI 对话
  - 4.3.4 `send` — 发送消息到通道
  - 4.3.5 `sessions.*` — 会话管理方法族
  - 4.3.6 事件类型：`agent`、`chat`、`presence`、`health`、`heartbeat`、`cron`
- 4.4 认证与授权
  - 4.4.1 Gateway Token 认证（`src/gateway/auth.ts`）
  - 4.4.2 设备配对（Device Pairing）机制（`src/gateway/device-auth.ts`）
  - 4.4.3 本地连接自动批准 vs 远程连接挑战签名
  - 4.4.4 Origin 检查防止跨站 WebSocket 劫持（`src/gateway/origin-check.ts`）

### 第 5 章 会话管理

- 5.1 会话模型设计
  - 5.1.1 会话键（Session Key）的结构：`agent:<agentId>:<mainKey>`
  - 5.1.2 会话范围（DM Scope）：`main` / `per-peer` / `per-channel-peer` / `per-account-channel-peer`
  - 5.1.3 会话持久化：JSON Store + JSONL 转录文件
- 5.2 会话路由源码分析
  - 5.2.1 路由绑定（`src/routing/bindings.ts`）
  - 5.2.2 路由解析（`src/routing/resolve-route.ts`）
  - 5.2.3 会话键生成（`src/routing/session-key.ts`）
  - 5.2.4 直接消息 vs 群组消息 vs Cron 会话 vs Webhook 会话的键映射规则
- 5.3 会话生命周期
  - 5.3.1 创建与初始化：首条消息触发会话创建
  - 5.3.2 重置策略：每日重置（Daily Reset）、空闲重置（Idle Reset）
  - 5.3.3 按类型/通道覆盖重置策略（`resetByType` / `resetByChannel`）
  - 5.3.4 会话修补（Session Patch）：运行时修改会话属性（`src/gateway/sessions-patch.ts`）
- 5.4 会话裁剪（Session Pruning）
  - 5.4.1 工具结果裁剪：在 LLM 调用前修剪旧工具结果
  - 5.4.2 上下文窗口守卫（`src/agents/context-window-guard.ts`）
- 5.5 会话间通信（Agent-to-Agent）
  - 5.5.1 `sessions_list` / `sessions_history` / `sessions_send` 工具
  - 5.5.2 跨会话消息传递与回复乒乓（Reply Ping-Pong）

### 第 6 章 通道路由与消息分发

- 6.1 通道注册表（Channel Registry）
  - 6.1.1 通道注册机制（`src/channels/registry.ts`）
  - 6.1.2 通道能力声明（`src/config/channel-capabilities.ts`）
  - 6.1.3 通道 Dock 层（`src/channels/dock.ts`）
- 6.2 入站消息处理流水线
  - 6.2.1 消息接收与规范化（`src/channels/chat-type.ts`）
  - 6.2.2 发送者身份识别（`src/channels/sender-identity.ts`）
  - 6.2.3 白名单匹配（`src/channels/allowlists/`）
  - 6.2.4 @提及门控（Mention Gating）（`src/channels/mention-gating.ts`）
  - 6.2.5 命令门控（Command Gating）（`src/channels/command-gating.ts`）
- 6.3 出站消息处理
  - 6.3.1 消息分块（Chunking）算法与通道限制
  - 6.3.2 确认反应（ACK Reactions）（`src/channels/ack-reactions.ts`）
  - 6.3.3 打字指示器（Typing Indicators）（`src/channels/typing.ts`）
- 6.4 多代理路由
  - 6.4.1 多代理配置：`agents.list[]` 与绑定
  - 6.4.2 基于通道/账号/对等方的路由规则
  - 6.4.3 子代理注册表（`src/agents/subagent-registry.ts`）

---

## 第三部分：AI Agent 运行时

### 第 7 章 Pi Agent 运行时核心

- 7.1 Pi Agent 是什么
  > **衍生解释**：Pi Agent 是 OpenClaw 的 AI 推理运行时，源自 Mario Zechner 开发的 `pi-agent-core` 库。它实现了 LLM 的 RPC 调用模式，负责管理对话上下文、工具调用和响应流式传输。可以类比为一个"AI 对话引擎"。
  - 7.1.1 `@mariozechner/pi-agent-core` / `pi-ai` / `pi-coding-agent` 依赖分析
  - 7.1.2 RPC 模式 vs 嵌入模式
  - 7.1.3 Agent 入口文件层次：`src/agents/pi-embedded.ts` → `pi-embedded-runner.ts`
- 7.2 Agent Loop（代理循环）端到端解析
  - 7.2.1 循环入口：Gateway 的 `agent` RPC 方法
  - 7.2.2 步骤 1：参数校验与会话解析
  - 7.2.3 步骤 2：`agentCommand` — 模型解析、技能快照加载
  - 7.2.4 步骤 3：`runEmbeddedPiAgent` — 队列序列化、auth profile 解析、Pi 会话构建
  - 7.2.5 步骤 4：`subscribeEmbeddedPiSession` — 事件桥接（tool → assistant → lifecycle）
  - 7.2.6 步骤 5：结果汇总、用量统计、会话持久化
- 7.3 队列与并发控制
  - 7.3.1 每会话车道（Session Lane）序列化
  - 7.3.2 全局车道（Global Lane）限流
  - 7.3.3 队列模式：collect / steer / followup
  - 7.3.4 车道实现（`src/gateway/server-lanes.ts`）
- 7.4 超时与中止机制
  - 7.4.1 Agent 运行超时（默认 600 秒）
  - 7.4.2 `agent.wait` 等待超时（默认 30 秒）
  - 7.4.3 AbortSignal 取消链
  - 7.4.4 聊天中止（`src/gateway/chat-abort.ts`）

### 第 8 章 模型提供者与故障转移

- 8.1 模型选择机制
  - 8.1.1 主模型 → 回退模型 → 提供者内部 auth failover 的三级策略
  - 8.1.2 模型选择源码（`src/agents/model-selection.ts`）
  - 8.1.3 模型回退（`src/agents/model-fallback.ts`）
  - 8.1.4 模型兼容性层（`src/agents/model-compat.ts`）
- 8.2 Auth Profile 与凭据管理
  - 8.2.1 Auth Profile 机制（`src/agents/auth-profiles.ts`）
  - 8.2.2 OAuth 认证流程：Anthropic Claude Pro/Max、OpenAI ChatGPT/Codex
  - 8.2.3 API Key 认证
  - 8.2.4 Profile 轮换与冷却（Cooldown）
  - 8.2.5 GitHub Copilot Token 认证（`src/providers/github-copilot-token.ts`）
- 8.3 模型目录与配置
  - 8.3.1 模型目录（`src/agents/model-catalog.ts`）
  - 8.3.2 模型扫描：OpenRouter 免费模型探测（`src/agents/model-scan.ts`）
  - 8.3.3 模型配置文件（`models.json`）与提供者配置（`src/agents/models-config.ts`）
  - 8.3.4 合成模型与特殊提供者（Venice、Chutes、Z.AI 等）
- 8.4 故障转移错误处理
  - 8.4.1 错误分类（`src/agents/failover-error.ts`）
  - 8.4.2 认证错误、计费错误、上下文溢出错误的检测与处理
  - 8.4.3 故障转移日志与用户可见的错误消息

### 第 9 章 系统提示词与上下文组装

- 9.1 系统提示词的构建
  - 9.1.1 基础提示词模板（`src/agents/system-prompt.ts`）
  - 9.1.2 提示词参数（`src/agents/system-prompt-params.ts`）
  - 9.1.3 提示词报告（`src/agents/system-prompt-report.ts`）
- 9.2 工作区与上下文文件注入
  - 9.2.1 工作区根目录解析（`src/agents/workspace.ts`）
  - 9.2.2 引导文件（Bootstrap Files）：`AGENTS.md` / `SOUL.md` / `TOOLS.md`（`src/agents/bootstrap-files.ts`）
  - 9.2.3 引导钩子（Bootstrap Hooks）（`src/agents/bootstrap-hooks.ts`）
  - 9.2.4 工作区模板（`src/agents/workspace-templates.ts`）
- 9.3 身份系统
  - 9.3.1 AI 助手身份（`src/agents/identity.ts`）
  - 9.3.2 身份文件（`src/agents/identity-file.ts`）
  - 9.3.3 身份头像（`src/agents/identity-avatar.ts`）
  - 9.3.4 通道前缀身份
- 9.4 上下文压缩（Compaction）
  - 9.4.1 自动压缩触发机制
  - 9.4.2 压缩流程（`src/agents/compaction.ts`）
  - 9.4.3 压缩前的记忆刷新（Memory Flush）
  - 9.4.4 压缩重试与缓冲重置

### 第 10 章 流式传输与分块回复

- 10.1 流式传输架构
  - 10.1.1 Pi Agent Core 事件流 → OpenClaw 事件桥接
  - 10.1.2 流事件类型：`lifecycle` / `assistant` / `tool`
  - 10.1.3 原始流处理（`src/agents/pi-embedded-subscribe.raw-stream.ts`）
- 10.2 块流（Block Streaming）
  - 10.2.1 EmbeddedBlockChunker 算法（`src/agents/pi-embedded-block-chunker.ts`）
  - 10.2.2 低水位线 / 高水位线分块策略
  - 10.2.3 断点偏好：段落 → 换行 → 句子 → 空白 → 硬断
  - 10.2.4 代码围栏（Fenced Block）安全拆分：关闭 + 重开
- 10.3 合并（Coalescing）与人性化节奏
  - 10.3.1 连续块合并策略：空闲间隙、最小/最大字符数
  - 10.3.2 人性化延迟（Human Delay）：`natural` / `custom` 模式
- 10.4 Telegram 草稿流
  - 10.4.1 `sendMessageDraft` 的实现
  - 10.4.2 partial 模式 vs block 模式
- 10.5 回复塑形与抑制
  - 10.5.1 `NO_REPLY` 静默令牌过滤
  - 10.5.2 消息工具去重
  - 10.5.3 工具摘要内联

---

## 第四部分：多通道消息系统

### 第 11 章 通道适配器抽象层

- 11.1 通道适配器设计模式
  > **衍生解释**：适配器模式（Adapter Pattern）是 GoF 设计模式之一，用于将不兼容的接口转换为统一的接口。OpenClaw 的通道适配器将每个消息平台的独特 API 转换为统一的内部消息格式，使 Gateway 可以不关心底层平台的具体实现。
  - 11.1.1 通道核心接口分析
  - 11.1.2 通道配置类型系统（`src/config/types.channels.ts`）
  - 11.1.3 通道与 Gateway 的连接桥
- 11.2 入站消息规范化
  - 11.2.1 消息格式统一：文本、图片、音频、视频、文件
  - 11.2.2 媒体附件处理（`src/gateway/chat-attachments.ts`）
  - 11.2.3 消息清洗（`src/gateway/chat-sanitize.ts`）
- 11.3 出站消息适配
  - 11.3.1 Markdown 格式化与通道特异性（`src/markdown/`）
  - 11.3.2 回复前缀（`src/channels/reply-prefix.ts`）
  - 11.3.3 会话标签（`src/channels/conversation-label.ts`）

### 第 12 章 核心通道实现深入

- 12.1 WhatsApp 通道（Baileys）
  - 12.1.1 Baileys 库简介：基于 Web WhatsApp 协议的非官方实现
  - 12.1.2 WhatsApp Web 登录与 QR 码配对（`src/web/login-qr.ts`）
  - 12.1.3 入站消息监听（`src/web/inbound.ts`）
  - 12.1.4 出站消息发送（`src/web/outbound.ts`）
  - 12.1.5 自动回复系统（`src/web/auto-reply.ts` 与 `auto-reply.impl.ts`）
  - 12.1.6 会话重连（`src/web/reconnect.ts`）
  - 12.1.7 媒体处理（`src/web/media.ts`）
- 12.2 Telegram 通道（grammY）
  - 12.2.1 grammY 框架与 Bot API 对接
  - 12.2.2 Telegram Bot 配置与 Webhook 模式
  - 12.2.3 草稿流特有实现
  - 12.2.4 自定义命令（`src/config/telegram-custom-commands.ts`）
- 12.3 Discord 通道
  - 12.3.1 discord.js / Carbon 库的使用
  - 12.3.2 Guild 管理、DM 策略
  - 12.3.3 原生 Slash 命令与文本命令
  - 12.3.4 每消息行数限制（`maxLinesPerMessage`）
- 12.4 Slack 通道（Bolt）
  - 12.4.1 Slack Bolt SDK 对接
  - 12.4.2 App Token + Bot Token 双 Token 架构
  - 12.4.3 线程（Thread）会话管理
- 12.5 其他核心通道
  - 12.5.1 Signal（signal-cli）
  - 12.5.2 BlueBubbles（iMessage 推荐集成）
  - 12.5.3 iMessage Legacy（macOS 原生 imsg）
  - 12.5.4 WebChat（Gateway 内置 Web 聊天）

### 第 13 章 通道扩展（Extension）机制

- 13.1 扩展架构设计
  - 13.1.1 扩展与核心通道的区别
  - 13.1.2 扩展加载机制（`src/gateway/server-plugins.ts`）
  - 13.1.3 扩展目录结构分析（以 `extensions/msteams/` 为例）
- 13.2 扩展 API 表面
  - 13.2.1 `src/extensionAPI.ts` — 扩展可访问的 API 集
  - 13.2.2 插件 SDK（`src/plugin-sdk/`）
  - 13.2.3 插件钩子（Plugin Hooks）接口
- 13.3 代表性扩展实现解析
  - 13.3.1 Microsoft Teams 扩展（Bot Framework 对接）
  - 13.3.2 Matrix 扩展（matrix-sdk-crypto-nodejs）
  - 13.3.3 Twitch 扩展
  - 13.3.4 Nostr 扩展
  - 13.3.5 Google Chat 扩展
  - 13.3.6 Zalo / Zalo Personal 扩展
  - 13.3.7 飞书（Feishu/Lark）扩展
  - 13.3.8 LINE 扩展
- 13.4 开发自定义扩展
  - 13.4.1 扩展脚手架搭建
  - 13.4.2 实现通道适配器接口
  - 13.4.3 注册到插件系统
  - 13.4.4 测试与调试

---

## 第五部分：工具系统与自动化

### 第 14 章 工具系统架构

- 14.1 工具的定义与分类
  - 14.1.1 工具类型：bash、browser、canvas、cron、sessions、nodes、channel
  - 14.1.2 工具 Schema 定义（`src/agents/pi-tools.schema.ts`）
  - 14.1.3 工具定义适配器（`src/agents/pi-tool-definition-adapter.ts`）
- 14.2 工具注册与策略
  - 14.2.1 工具注册流程（`src/agents/pi-tools.ts`）
  - 14.2.2 工具策略（Tool Policy）：允许列表 / 拒绝列表（`src/agents/tool-policy.ts`）
  - 14.2.3 工具前置拦截（Before Tool Call）（`src/agents/pi-tools.before-tool-call.ts`）
  - 14.2.4 工具显示名映射（`src/agents/tool-display.json`）
- 14.3 工具执行与结果处理
  - 14.3.1 工具调用 ID 生成（`src/agents/tool-call-id.ts`）
  - 14.3.2 工具结果守卫（`src/agents/session-tool-result-guard.ts`）
  - 14.3.3 工具图像处理（`src/agents/tool-images.ts`）
  - 14.3.4 工具摘要生成（`src/agents/tool-summaries.ts`）
- 14.4 执行审批机制
  - 14.4.1 执行审批管理器（`src/gateway/exec-approval-manager.ts`）
  - 14.4.2 用户审批工作流

### 第 15 章 Bash 工具与进程管理

- 15.1 Bash 执行引擎
  - 15.1.1 `src/agents/bash-tools.exec.ts` — 命令执行核心
  - 15.1.2 PTY 模拟终端（`src/agents/bash-tools.exec.pty.ts`）
  - 15.1.3 PTY 回退机制（`src/agents/bash-tools.exec.pty-fallback.ts`）
  - 15.1.4 PATH 环境与安全 bin 列表（`src/agents/pi-tools.safe-bins.ts`）
- 15.2 进程管理
  - 15.2.1 进程工具（`src/agents/bash-tools.process.ts`）
  - 15.2.2 后台进程注册表（`src/agents/bash-process-registry.ts`）
  - 15.2.3 进程 send-keys 操作
- 15.3 Shell 工具
  - 15.3.1 Shell 工具集（`src/agents/shell-utils.ts`）
  - 15.3.2 工作区运行（`src/agents/workspace-run.ts`）

### 第 16 章 浏览器控制

- 16.1 浏览器架构概述
  - 16.1.1 Playwright + CDP（Chrome DevTools Protocol）的混合方案
    > **衍生解释**：CDP 是 Chrome 浏览器提供的调试协议，允许外部程序通过 WebSocket 控制浏览器的行为——包括导航页面、执行 JavaScript、截取屏幕截图等。Playwright 是微软开发的浏览器自动化库，在 CDP 之上提供了更高层的 API。
  - 16.1.2 OpenClaw 管理的 Chrome/Chromium 实例（`src/browser/chrome.ts`）
  - 16.1.3 浏览器配置文件（Profiles）管理（`src/browser/profiles.ts`）
- 16.2 CDP 层实现
  - 16.2.1 CDP 连接管理（`src/browser/cdp.ts`）
  - 16.2.2 CDP 辅助函数（`src/browser/cdp.helpers.ts`）
  - 16.2.3 Target ID 管理与标签页操作（`src/browser/target-id.ts`）
- 16.3 Playwright 层实现
  - 16.3.1 Playwright 会话管理（`src/browser/pw-session.ts`）
  - 16.3.2 AI 辅助模块（`src/browser/pw-ai.ts`）
  - 16.3.3 角色快照（Role Snapshot）（`src/browser/pw-role-snapshot.ts`）
  - 16.3.4 Playwright 工具核心（`src/browser/pw-tools-core.ts`）：
    - 交互操作（`interactions.ts`）
    - 截图（`screenshots-element-selector`）
    - 活动追踪（`activity.ts`）
    - 状态管理（`state.ts`）
    - 下载处理（`downloads.ts`）
    - 存储管理（`storage.ts`）
- 16.4 浏览器服务器
  - 16.4.1 浏览器 HTTP 服务器（`src/browser/server.ts`）
  - 16.4.2 服务器上下文与标签页管理（`src/browser/server-context.ts`）
  - 16.4.3 客户端动作层（`src/browser/client-actions.ts`）
  - 16.4.4 浏览器扩展中继（`src/browser/extension-relay.ts`）
  - 16.4.5 Bridge Server（`src/browser/bridge-server.ts`）

### 第 17 章 Canvas 与 A2UI

- 17.1 Canvas 概念
  - 17.1.1 什么是 Canvas：Agent 驱动的可视化工作区
  - 17.1.2 A2UI（Agent-to-UI）：AI Agent 直接操纵用户界面
    > **衍生解释**：A2UI 是 OpenClaw 提出的概念，指 AI Agent 可以直接向用户推送 HTML 内容并操纵一个 Web 视图。它类似于"AI 主导的前端渲染"——Agent 不仅能回复文字，还能展示交互式界面。
- 17.2 Canvas Host 实现
  - 17.2.1 Canvas 服务器（`src/canvas-host/server.ts`）
  - 17.2.2 A2UI 核心（`src/canvas-host/a2ui.ts` 与 `a2ui/`）
  - 17.2.3 A2UI 打包（`scripts/bundle-a2ui.sh`）
- 17.3 Canvas 工具
  - 17.3.1 push / reset / eval / snapshot 操作
  - 17.3.2 Canvas 在 macOS / iOS / Android 上的表面

### 第 18 章 Cron 调度与自动化

- 18.1 Cron 系统设计
  - 18.1.1 Cron 服务（`src/cron/service.ts`）
  - 18.1.2 调度引擎（`src/cron/schedule.ts`）与 Croner 库
  - 18.1.3 Cron 表达式解析（`src/cron/parse.ts`）
  - 18.1.4 Cron 规范化（`src/cron/normalize.ts`）
- 18.2 Cron 作业执行
  - 18.2.1 隔离 Agent 执行（`src/cron/isolated-agent.ts`）
  - 18.2.2 投递计划（Delivery Plan）（`src/cron/delivery.ts`）
  - 18.2.3 运行日志（`src/cron/run-log.ts`）
  - 18.2.4 Cron 存储与迁移（`src/cron/store.ts`）
- 18.3 Webhook 与 Gmail Pub/Sub
  - 18.3.1 Webhook 触发器
  - 18.3.2 Gmail Pub/Sub 邮件钩子（`src/hooks/gmail.ts`）
  - 18.3.3 钩子系统总览（`src/hooks/hooks.ts`）

### 第 19 章 节点系统（Nodes）

- 19.1 节点概念
  - 19.1.1 什么是节点：设备能力的远程暴露
  - 19.1.2 节点角色：macOS / iOS / Android / 无头
  - 19.1.3 节点命令：`canvas.*` / `camera.*` / `screen.record` / `location.get` / `system.run` / `system.notify`
- 19.2 节点注册与发现
  - 19.2.1 节点注册表（`src/gateway/node-registry.ts`）
  - 19.2.2 节点事件系统（`src/gateway/server-node-events.ts`）
  - 19.2.3 节点订阅（`src/gateway/server-node-subscriptions.ts`）
  - 19.2.4 节点命令策略（`src/gateway/node-command-policy.ts`）
- 19.3 节点 Host 实现
  - 19.3.1 `src/node-host/runner.ts` — 节点命令执行器
  - 19.3.2 `src/node-host/config.ts` — 节点配置

---

## 第六部分：记忆、技能与生态系统

### 第 20 章 记忆系统

- 20.1 记忆模型设计
  - 20.1.1 "纯 Markdown 即记忆"的设计哲学
  - 20.1.2 记忆文件布局：`MEMORY.md`（长期）+ `memory/YYYY-MM-DD.md`（日志）
  - 20.1.3 记忆管理器（`src/memory/manager.ts`）
- 20.2 向量记忆搜索
  - 20.2.1 嵌入向量（Embeddings）引擎
    > **衍生解释**：嵌入向量是将文本转换为高维向量的技术。语义相似的文本在向量空间中距离相近，使得可以通过计算向量相似度来实现"模糊搜索"——即使搜索词和文档用词不同，只要语义相关就能匹配。
  - 20.2.2 多嵌入提供者支持：OpenAI / Gemini / Voyage / Local（node-llama-cpp）（`src/memory/embeddings.ts`）
  - 20.2.3 SQLite 存储（`src/memory/sqlite.ts`）
  - 20.2.4 sqlite-vec 向量加速（`src/memory/sqlite-vec.ts`）
- 20.3 混合搜索（BM25 + 向量）
  - 20.3.1 BM25 全文搜索原理
  - 20.3.2 混合检索实现（`src/memory/hybrid.ts`）
  - 20.3.3 分数融合策略：加权线性组合
- 20.4 高级记忆功能
  - 20.4.1 批量嵌入（Batch Indexing）（`src/memory/batch-openai.ts` / `batch-gemini.ts`）
  - 20.4.2 嵌入缓存机制
  - 20.4.3 会话记忆搜索（实验性）
  - 20.4.4 QMD 后端（BM25 + 向量 + 重排序）
  - 20.4.5 搜索管理器（`src/memory/search-manager.ts`）

### 第 21 章 技能系统

- 21.1 技能平台设计
  - 21.1.1 什么是技能：可热插拔的 Agent 能力模块
  - 21.1.2 技能类型：Bundled（内置）/ Managed（托管）/ Workspace（工作区）
  - 21.1.3 技能加载与快照（`src/agents/skills.ts`）
- 21.2 技能结构
  - 21.2.1 `SKILL.md` 文件规范
  - 21.2.2 内置技能目录分析（52 个技能）
  - 21.2.3 代表性技能解读：`coding-agent`、`github`、`discord`、`canvas`、`weather` 等
- 21.3 技能安装与管理
  - 21.3.1 技能安装流程（`src/agents/skills-install.ts`）
  - 21.3.2 技能状态管理（`src/agents/skills-status.ts`）
  - 21.3.3 技能 CLI 命令（`src/cli/skills-cli.ts`）
- 21.4 ClawHub 技能注册中心
  - 21.4.1 ClawHub 的设计与功能
  - 21.4.2 技能自动搜索与安装

### 第 22 章 钩子系统

- 22.1 内部钩子（Gateway Hooks）
  - 22.1.1 钩子加载器（`src/hooks/loader.ts`）
  - 22.1.2 钩子安装（`src/hooks/install.ts`）
  - 22.1.3 内置钩子（`src/hooks/bundled/`）
  - 22.1.4 `agent:bootstrap` 钩子
  - 22.1.5 命令钩子：`/new`、`/reset`、`/stop`
- 22.2 插件钩子（Plugin Hooks）
  - 22.2.1 钩子映射（`src/gateway/hooks-mapping.ts`）
  - 22.2.2 Agent 生命周期钩子：`before_agent_start` / `agent_end`
  - 22.2.3 工具生命周期钩子：`before_tool_call` / `after_tool_call` / `tool_result_persist`
  - 22.2.4 消息钩子：`message_received` / `message_sending` / `message_sent`
  - 22.2.5 网关钩子：`gateway_start` / `gateway_stop`

---

## 第七部分：安全、配置与基础设施

### 第 23 章 配置系统

- 23.1 配置加载与解析
  - 23.1.1 配置文件位置：`~/.openclaw/openclaw.json`（JSON5 格式）
  - 23.1.2 配置 Schema（`src/config/schema.ts`）
  - 23.1.3 Zod Schema 校验（`src/config/zod-schema.ts`）
  - 23.1.4 配置 IO（`src/config/io.ts`）
  - 23.1.5 配置路径解析（`src/config/config-paths.ts`）
- 23.2 配置类型系统详解
  - 23.2.1 核心类型概览（`src/config/types.ts`）
  - 23.2.2 Agent 配置（`types.agents.ts` / `types.agent-defaults.ts`）
  - 23.2.3 通道配置（`types.channels.ts` / `types.discord.ts` / `types.telegram.ts` 等）
  - 23.2.4 安全配置（`types.sandbox.ts` / `types.auth.ts`）
  - 23.2.5 模型配置（`types.models.ts`）
  - 23.2.6 工具/技能/钩子配置（`types.tools.ts` / `types.skills.ts` / `types.hooks.ts`）
- 23.3 配置热重载
  - 23.3.1 配置重载机制（`src/gateway/config-reload.ts`）
  - 23.3.2 运行时覆盖（`src/config/runtime-overrides.ts`）
  - 23.3.3 服务器重载处理器（`src/gateway/server-reload-handlers.ts`）
- 23.4 遗留配置迁移
  - 23.4.1 迁移框架（`src/config/legacy.ts`）
  - 23.4.2 迁移规则（`src/config/legacy.rules.ts`）
  - 23.4.3 分阶段迁移（`legacy.migrations.part-1/2/3.ts`）
- 23.5 环境变量
  - 23.5.1 环境变量替换（`src/config/env-substitution.ts`）
  - 23.5.2 环境变量列表（`src/config/env-vars.ts`）

### 第 24 章 安全模型

- 24.1 安全设计原则
  - 24.1.1 入站 DM 作为不受信任输入
  - 24.1.2 默认安全 vs 显式放开
  - 24.1.3 提示词注入防御
- 24.2 DM 配对系统
  - 24.2.1 配对流程：配对码生成 → 用户审批 → 白名单持久化
  - 24.2.2 配对 CLI（`src/cli/pairing-cli.ts`）
  - 24.2.3 DM 策略：`pairing` / `open`
- 24.3 沙箱机制
  - 24.3.1 沙箱设计：非主会话的 Docker 隔离（`src/agents/sandbox.ts`）
  - 24.3.2 沙箱配置解析（`src/agents/sandbox/`）
  - 24.3.3 沙箱路径管理（`src/agents/sandbox-paths.ts`）
  - 24.3.4 Docker 沙箱镜像（`Dockerfile.sandbox` / `Dockerfile.sandbox-browser`）
  - 24.3.5 工具白名单 / 黑名单
- 24.4 安全审计
  - 24.4.1 审计系统（`src/security/audit.ts`）
  - 24.4.2 文件系统审计（`src/security/audit-fs.ts`）
  - 24.4.3 技能安全扫描（`src/security/skill-scanner.ts`）
  - 24.4.4 外部内容安全（`src/security/external-content.ts`）
  - 24.4.5 `openclaw security audit` 命令
- 24.5 SOUL 安全
  - 24.5.1 恶意 SOUL 检测（`src/hooks/soul-evil.ts`）

### 第 25 章 CLI 命令行工具

- 25.1 CLI 架构
  - 25.1.1 Commander.js 命令框架（`src/cli/program.ts`）
  - 25.1.2 命令分发（`src/cli/run-main.ts`）
  - 25.1.3 命令格式化（`src/cli/command-format.ts`）
- 25.2 核心命令解析
  - 25.2.1 `gateway` — 启动/管理 Gateway 服务器
  - 25.2.2 `agent` — 发送消息给 AI Agent
  - 25.2.3 `send` — 通过通道发送消息
  - 25.2.4 `channels` — 通道管理（登录/配置）
  - 25.2.5 `models` — 模型管理（列表/设置/扫描）
  - 25.2.6 `sessions` — 会话管理
  - 25.2.7 `skills` — 技能管理
  - 25.2.8 `browser` — 浏览器控制
  - 25.2.9 `cron` — 定时任务管理
  - 25.2.10 `nodes` — 节点管理
  - 25.2.11 `doctor` — 健康检查与诊断
  - 25.2.12 `security` — 安全审计
  - 25.2.13 `onboard` — 引导向导
  - 25.2.14 `update` — 版本更新
- 25.3 聊天命令（Slash Commands）
  - 25.3.1 命令处理（`src/commands/`）
  - 25.3.2 `/status` / `/new` / `/reset` / `/compact` / `/think` / `/verbose` / `/model`

### 第 26 章 基础设施

- 26.1 日志系统
  - 26.1.1 日志框架：tslog（`src/logger.ts`）
  - 26.1.2 日志分级与输出（`src/logging.ts` / `src/logging/`）
  - 26.1.3 WebSocket 日志（`src/gateway/ws-log.ts`）
- 26.2 媒体管道
  - 26.2.1 媒体获取（`src/media/fetch.ts`）
  - 26.2.2 媒体解析（`src/media/parse.ts`）
  - 26.2.3 媒体存储（`src/media/store.ts`）
  - 26.2.4 图像处理：Sharp 库（`src/media/image-ops.ts`）
  - 26.2.5 音频处理（`src/media/audio.ts`）
  - 26.2.6 MIME 类型检测（`src/media/mime.ts`）
- 26.3 链接理解与媒体理解
  - 26.3.1 链接理解（`src/link-understanding/`）
  - 26.3.2 媒体理解（`src/media-understanding/`）
- 26.4 TTS（文本转语音）
  - 26.4.1 TTS 引擎（`src/tts/`）
  - 26.4.2 ElevenLabs / Edge TTS 集成
- 26.5 投票系统
  - 26.5.1 投票（Polls）实现（`src/polls.ts`）

---

## 第八部分：客户端应用与 Web UI

### 第 27 章 Web 控制台 UI

- 27.1 UI 技术选型
  - 27.1.1 Lit Web Components 框架
    > **衍生解释**：Lit 是 Google 开发的轻量级 Web Components 库。Web Components 是浏览器原生支持的组件化标准，包括 Custom Elements、Shadow DOM 和 HTML Templates。Lit 在此基础上提供了声明式模板和响应式属性。
  - 27.1.2 Vite 构建工具
  - 27.1.3 Legacy Decorators 约束
- 27.2 控制台 UI 架构
  - 27.2.1 Gateway 内嵌 UI 服务（`src/gateway/control-ui.ts`）
  - 27.2.2 UI 源码结构（`ui/src/`）
  - 27.2.3 Gateway WebSocket 客户端
- 27.3 WebChat 实现
  - 27.3.1 WebChat 作为 Gateway WS 客户端
  - 27.3.2 聊天历史加载与消息发送

### 第 28 章 原生客户端应用

- 28.1 macOS 应用（OpenClaw.app）
  - 28.1.1 Swift + SwiftUI 架构
  - 28.1.2 菜单栏控制（Menu Bar）
  - 28.1.3 Voice Wake + Push-to-Talk
  - 28.1.4 Talk Mode 覆盖层
  - 28.1.5 远程 Gateway 控制
  - 28.1.6 macOS 权限与 TCC
- 28.2 iOS 节点应用
  - 28.2.1 Bonjour 配对
  - 28.2.2 Canvas 表面
  - 28.2.3 Voice Wake / Talk Mode
  - 28.2.4 摄像头与屏幕录制
- 28.3 Android 节点应用
  - 28.3.1 Kotlin 架构
  - 28.3.2 Canvas / 摄像头 / 屏幕录制
  - 28.3.3 可选 SMS 支持
- 28.4 共享组件
  - 28.4.1 `apps/shared/OpenClawKit/` — 跨平台共享代码

---

## 第九部分：部署与运维

### 第 29 章 部署方案

- 29.1 本地部署
  - 29.1.1 npm 全局安装
  - 29.1.2 守护进程管理（launchd / systemd）
  - 29.1.3 `openclaw doctor` 健康检查
- 29.2 Docker 部署
  - 29.2.1 Dockerfile 分析：多阶段构建、非 root 用户
  - 29.2.2 `docker-compose.yml` 解析
  - 29.2.3 Docker 沙箱（`Dockerfile.sandbox`）
  - 29.2.4 环境变量配置
- 29.3 远程访问
  - 29.3.1 Tailscale Serve / Funnel 配置（`src/gateway/server-tailscale.ts`）
  - 29.3.2 SSH 隧道
  - 29.3.3 服务发现（`src/gateway/server-discovery.ts`）
  - 29.3.4 Bonjour/mDNS（`@homebridge/ciao`）
- 29.4 Nix 声明式部署
- 29.5 VPS 部署（`docs/vps.md`）

### 第 30 章 监控与故障排查

- 30.1 健康检查
  - 30.1.1 Gateway 健康端点
  - 30.1.2 Gateway 探针（`src/gateway/probe.ts`）
  - 30.1.3 心跳机制
- 30.2 在线追踪（Presence）
  - 30.2.1 在线状态管理
  - 30.2.2 打字指示器
- 30.3 用量追踪
  - 30.3.1 Token 用量统计（`src/agents/usage.ts`）
  - 30.3.2 用量展示选项：off / tokens / full
- 30.4 故障排查
  - 30.4.1 `openclaw doctor` 诊断流程
  - 30.4.2 常见问题：通道断连、模型错误、沙箱故障
  - 30.4.3 日志分析指南

---

## 第十部分：高级主题与实践

### 第 31 章 多代理架构

- 31.1 多代理设计
  - 31.1.1 Agent 范围（`src/agents/agent-scope.ts`）
  - 31.1.2 Agent 路径（`src/agents/agent-paths.ts`）
  - 31.1.3 Agent 默认配置（`src/agents/defaults.ts`）
- 31.2 子代理（Sub-agent）
  - 31.2.1 子代理注册表（`src/agents/subagent-registry.ts`）
  - 31.2.2 子代理生成（`sessions_spawn` 工具）
  - 31.2.3 子代理公告（`src/agents/subagent-announce.ts`）
  - 31.2.4 子代理公告队列（`src/agents/subagent-announce-queue.ts`）
- 31.3 多代理沙箱工具
  - 31.3.1 跨代理工具隔离
  - 31.3.2 沙箱代理配置

### 第 32 章 ACP（Agent Communication Protocol）

- 32.1 ACP 协议概述
  > **衍生解释**：ACP 是一种 Agent 间通信协议标准，允许不同 AI Agent 之间进行结构化通信。它定义了 Agent 的发现、认证和消息交换格式。
  - 32.1.1 `@agentclientprotocol/sdk` 依赖
  - 32.1.2 `src/acp/` — OpenClaw 的 ACP 实现

### 第 33 章 TUI（终端用户界面）

- 33.1 TUI 架构
  - 33.1.1 `@mariozechner/pi-tui` 集成
  - 33.1.2 `src/tui/` — TUI 适配层
  - 33.1.3 终端渲染（`src/terminal/`）

### 第 34 章 实战项目：构建你自己的 AI 助手

- 34.1 项目规划
  - 34.1.1 需求分析：多通道 AI 助手的最小可行产品
  - 34.1.2 架构设计参考
  - 34.1.3 技术选型建议
- 34.2 核心功能实现
  - 34.2.1 搭建 WebSocket Gateway 骨架
  - 34.2.2 实现消息路由与会话管理
  - 34.2.3 对接 LLM API（OpenAI / Anthropic）
  - 34.2.4 实现基础工具系统（bash 执行）
- 34.3 通道集成
  - 34.3.1 实现 Telegram Bot 通道适配器
  - 34.3.2 实现 Discord Bot 通道适配器
  - 34.3.3 实现 WebChat 通道
- 34.4 高级功能
  - 34.4.1 添加记忆系统（向量搜索）
  - 34.4.2 添加技能系统
  - 34.4.3 添加 Cron 调度
- 34.5 部署上线
  - 34.5.1 Docker 容器化
  - 34.5.2 守护进程配置
  - 34.5.3 安全加固

---

## 附录

### 附录 A：OpenClaw 配置参考

- 完整配置 JSON5 示例
- 所有配置键速查表

### 附录 B：Gateway WebSocket 协议参考

- 所有请求方法与参数
- 所有事件类型与负载

### 附录 C：工具定义参考

- 所有内置工具的 Schema 与用途

### 附录 D：源码导航图

- 关键模块依赖关系图
- 数据流序列图（message → gateway → agent → tool → response → channel）

### 附录 E：术语表

- 本书出现的全部专业术语中英文对照与解释

---

## 关于估算

| 部分 | 章节数 | 预估字数 |
|------|--------|----------|
| 第一部分：全局概览 | 2 章 | ~8,000 字 |
| 第二部分：Gateway 控制平面 | 4 章 | ~16,000 字 |
| 第三部分：AI Agent 运行时 | 4 章 | ~16,000 字 |
| 第四部分：多通道消息系统 | 3 章 | ~12,000 字 |
| 第五部分：工具系统与自动化 | 6 章 | ~18,000 字 |
| 第六部分：记忆、技能与生态 | 3 章 | ~10,000 字 |
| 第七部分：安全、配置与基础设施 | 4 章 | ~12,000 字 |
| 第八部分：客户端应用与 Web UI | 2 章 | ~6,000 字 |
| 第九部分：部署与运维 | 2 章 | ~6,000 字 |
| 第十部分：高级主题与实践 | 4 章 | ~12,000 字 |
| 附录 | 5 项 | ~4,000 字 |
| **合计** | **34 章 + 5 附录** | **~120,000 字** |
