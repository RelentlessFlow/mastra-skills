# 提示词推荐实现

提示词推荐放在后端 agent 侧维护，前端只通过 `requestContext` 传用户自定义系统提示词、项目提示词等动态补充。不要在前端复制完整 agent prompt。

```ts
import type { AgentConfig } from '@mastra/core/agent'

export function dynamicInstruction(
  instruction: string,
): AgentConfig['instructions'] {
  return ({ requestContext }) => {
    const systemPrompt = requestContext.get<string, string>('__system_prompt__') || ''
    const projectPrompt = requestContext.get<string, string>('__project_prompt__') || ''

    return `${instruction}

其他提示信息如下：
用户自定义系统提示词：${systemPrompt}
用户自定义项目提示词：${projectPrompt}`
  }
}
```

```ts
const instructions = `
# Role
你是一个按业务定义的编程助手。

## 原则
- 使用简体中文回复。
- 涉及用户文件时使用绝对路径。
- 修改代码后按项目方式验证。
`
```
