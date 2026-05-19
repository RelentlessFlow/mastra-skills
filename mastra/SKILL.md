---
name: mastra
description: Mastra 参考实现与开发偏好。处理 Mastra agents、models、MCP、memory、workspace、Hono server，或前端 chat/thread 代码时使用；前端尽量优先用 @mastra/client-js、@mastra/react 和 @ant-design/x，必要时再 HTTP 调接口。
---

# Mastra

## 1. 简介、适用范围

本 skill 用于实现或审查 Mastra 相关功能。这里沉淀的是一套通用的后端与前端实现偏好，不是固定业务模板；业务可以自由替换 agent、调整工具、增删 memory 或 workspace 能力。

前端部分只给推荐方向：尽量复用 `@mastra/client-js` 和 `@mastra/react`；聊天 UI 尽量使用 `@ant-design/x` / `@ant-design/x-markdown` 的组件；SDK 不覆盖、类型不顺或项目有自定义路由时，再用 HTTP 薄封装。

代码示例放在 references 中，需要实现时再读取：

- Agent 示例：[references/backend-agent.md](references/backend-agent.md)
- 模型示例：[references/backend-model.md](references/backend-model.md)
- MCP 示例：[references/backend-mcp.md](references/backend-mcp.md)
- Request Context 示例：[references/backend-request-context.md](references/backend-request-context.md)
- 记忆系统示例：[references/backend-memory.md](references/backend-memory.md)
- 工作区与 Skills 示例：[references/backend-workspace-skills.md](references/backend-workspace-skills.md)
- 提示词示例：[references/backend-prompt.md](references/backend-prompt.md)
- Hono 服务端示例：[references/backend-hono.md](references/backend-hono.md)
- 前端示例：[references/frontend.md](references/frontend.md)

## 2.1 Agent 推荐实现

Agent 可以按业务拆成轻量聊天、任务执行、检索问答、数据分析等类型。轻量聊天 agent 通常少工具、小 `maxSteps`；任务型 agent 可以接入 workspace、完整 tools 和更大的 `maxSteps`。

需要代码时读取 [references/backend-agent.md](references/backend-agent.md)。

## 2.2 模型推荐实现

模型推荐保持“动态模型 + OpenAI-compatible 适配”的思路。模型选择可以从 `requestContext.__model__` 读取；认证信息优先走服务端环境变量、配置中心或模型配置，不建议放进 Mastra requestContext。

需要代码时读取 [references/backend-model.md](references/backend-model.md)。

## 2.3 MCP 推荐实现

MCP 推荐保持“请求上下文声明启用哪些 server，后端动态加载哪些 tools”的方向；单个 MCP 服务失败时返回 `{}`，不要让整个聊天不可用。

需要代码时读取 [references/backend-mcp.md](references/backend-mcp.md)。

## 2.4 Request Context 推荐实现

前端可以通过 Mastra `requestContext` 传运行时动态值。推荐只传业务上下文、模型选择、提示词、工作区、MCP 配置等信息；认证信息走服务端环境变量、配置中心或模型配置。

推荐字段：

- `__model__`：本次请求选择的模型。
- `__system_prompt__`：用户自定义系统提示词。
- `__project_prompt__`：业务或项目提示词。
- `__workspace__`：工作区绝对路径。
- `__mcp_servers__`：启用的 MCP servers。

需要代码时读取 [references/backend-request-context.md](references/backend-request-context.md) 和 [references/frontend.md](references/frontend.md) 的「@mastra/react 示例」。

## 2.5 记忆系统推荐实现

推荐按业务决定是否启用 memory、如何生成标题、如何设计 `resourceId`。前端读取 memory 状态、thread 列表和 thread messages 时优先用 `@mastra/client-js`。

需要代码时读取 [references/backend-memory.md](references/backend-memory.md) 和 [references/frontend.md](references/frontend.md) 的「@mastra/client-js 示例」。

## 2.6 其他功能（工作区、Skills 等）

工作区可通过 `requestContext.__workspace__` 动态创建文件系统，并按业务启用 workspace tools 和 skills。

Mastra 官方也提供 Skills，可作为外部参考来源：

- 文档地址：https://mastra.ai/docs/build-with-ai/skills
- 安装示例：`pnpm dlx skills add mastra-ai/skills`
- 更新示例：`pnpm dlx skills update mastra`

需要 workspace 代码时读取 [references/backend-workspace-skills.md](references/backend-workspace-skills.md)。

## 2.7 提示词推荐实现

提示词推荐放在后端 agent 侧维护，前端只通过 `requestContext` 传用户自定义系统提示词、业务提示词等动态补充。不要在前端复制完整 agent prompt。

需要代码时读取 [references/backend-prompt.md](references/backend-prompt.md)。

## 2.8 服务端集成（基于 Hono）

服务端可用 Hono + Mastra 集成：创建 Hono app，初始化 Mastra，按需挂自定义路由，再用 `MastraServer` 接入标准 Mastra API。

注意：不要假设 `@mastra/react` 一定走自定义路由。优先确认所用版本的 `@mastra/react` / `@mastra/client-js` 行为；自定义路由更适合作为业务自有 AI SDK UI-message stream 端点参考。

需要代码时读取 [references/backend-hono.md](references/backend-hono.md)。

## 3.1 前端 SDK 概述

前端优先考虑这两个包：

- `@mastra/client-js`：标准 Mastra resources，例如 agents、memory、threads、messages、tools。
- `@mastra/react`：React 场景下的 client hook、chat hook、message state 和 stream 发送。
- `@ant-design/x`：聊天 UI 组件，例如 `XProvider`、`Conversations`、`Bubble.List`、`Actions`、`Think`、`ThoughtChain`、`Sources`。
- `@ant-design/x-markdown`：Markdown、代码块、LaTeX 等消息内容渲染。

共享客户端建议集中放在应用自己的 Mastra SDK/client 模块中。需要代码时读取 [references/frontend.md](references/frontend.md) 的「前端 SDK 概述」和「Ant Design X 组件示例」。

## 3.2 @mastra/client-js 示例

常见读取操作包括 `listAgents()`、`getMemoryStatus()`、`listMemoryThreads()`、`listThreadMessages()`。SDK 返回结构和 UI 类型不匹配时，优先在 hook 层做薄适配；实在不顺再 HTTP 调接口。

需要代码时读取 [references/frontend.md](references/frontend.md) 的「@mastra/client-js 示例」。

## 3.3 @mastra/react 示例

发送消息可以先用 `useChat`，具体 UI 状态按业务组织。切换 thread 时注意同步 `initialMessages` / `setMessages`，run 过程中让 `useChat` 托管实时 message 列表。

需要代码时读取 [references/frontend.md](references/frontend.md) 的「@mastra/react 示例」。

## 3.4 API 调用示例

只有 SDK 不合适、业务有自定义路由，或临时验证接口时再使用 HTTP。建议把 HTTP 调用做成窄封装，不要散落在组件里。

需要代码时读取 [references/frontend.md](references/frontend.md) 的「API 调用示例」。

## 代码风格

- 注释、导入顺序、React 写法遵循所在项目的编码规范。
- 后端 agent instructions 涉及用户文件操作时，要求使用绝对路径。
- 资料和 skill 正文优先使用中文；包名、API 名、代码片段保留原文。
