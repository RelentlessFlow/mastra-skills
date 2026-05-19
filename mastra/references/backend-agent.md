# Agent 推荐实现

当前代码里的 `assistant` 和 `specialist` 只是业务示例，不是固定 agent 模型。轻量聊天 agent 可以少工具、小 `maxSteps`；任务型 agent 可以接入 workspace、完整 tools 和更大的 `maxSteps`。

```ts
import { Agent } from '@mastra/core/agent'
import { dynamicInstruction } from '../instruction'
import { dynamicModel } from '../llm'
import { dynamicMemory } from '../memory'
import { dynamicTools } from '../tools'

const instructions = `
# Role
按业务填写 agent 角色、能力边界和输出风格。
`

export function initExampleAgent() {
  return new Agent({
    id: 'example',
    name: '示例模式',
    description: '按业务填写',
    instructions: dynamicInstruction(instructions),
    model: dynamicModel(),
    memory: dynamicMemory(),
    tools: dynamicTools(),
    defaultOptions: { maxSteps: 20 },
  })
}
```

```ts
export function initMastra() {
  const example = initExampleAgent()
  return new Mastra({
    agents: { example },
    storage,
  })
}
```
