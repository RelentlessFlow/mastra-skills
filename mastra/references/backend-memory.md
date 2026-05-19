# 记忆系统推荐实现

当前 `dynamicMemory()` 使用 `@mastra/memory`。推荐按业务决定是否启用 memory、如何生成标题、如何设计 `resourceId`。

```ts
import { Memory } from '@mastra/memory'

export function dynamicMemory(): DynamicArgument<MastraMemory> {
  return () => new Memory({
    options: {
      lastMessages: 40,
      generateTitle: {
        model: dynamicModel({ modelId: LlmModel.QWEN_35_FLASH }),
        instructions: `
        - 请根据用户开启对话的第一条消息生成一个简短的标题
        - 标题必须使用与用户消息相同的语言
        - 标题长度严格控制在 80 个字符以内
        - 严禁使用引号或冒号
        `.trim(),
      },
    },
  })
}
```
