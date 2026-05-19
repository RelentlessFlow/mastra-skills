# 模型推荐实现

推荐保持“动态模型 + OpenAI-compatible 适配”的思路。模型选择可以从 `requestContext.__model__` 读取；认证信息优先走服务端环境变量、配置中心或模型配置，不建议放进 Mastra requestContext。

```ts
import type { MastraModelConfig } from '@mastra/core/llm'
import type { DynamicArgument } from '@mastra/core/types'
import { createOpenAICompatible } from '@ai-sdk/openai-compatible'

export type RuntimeModel =
  | string
  | {
    modelId: string
    baseURL: string
    apiKey?: string
  }

const DEFAULT_MODEL = 'gpt-4.1-mini'

export function createModel(model: RuntimeModel = DEFAULT_MODEL) {
  if (typeof model === 'string') {
    const openAICompatible = createOpenAICompatible({
      name: 'default',
      baseURL: process.env.OPENAI_COMPATIBLE_BASE_URL!,
      apiKey: process.env.OPENAI_API_KEY,
      includeUsage: true,
    })
    return openAICompatible(model)
  }

  const openAICompatible = createOpenAICompatible({
    name: 'custom',
    baseURL: model.baseURL,
    apiKey: model.apiKey,
    includeUsage: true,
  })
  return openAICompatible(model.modelId)
}

export function dynamicModel(config?: {
  modelId?: RuntimeModel
}): DynamicArgument<MastraModelConfig> {
  return ({ requestContext }) => {
    const model = requestContext.get<string, RuntimeModel>('__model__')
    return createModel(config?.modelId ?? model)
  }
}
```

```ts
export function createEmbedding(model = 'text-embedding-3-small') {
  const openAICompatible = createOpenAICompatible({
    name: 'embedding',
    baseURL: process.env.OPENAI_COMPATIBLE_BASE_URL!,
    apiKey: process.env.OPENAI_API_KEY,
  })
  return openAICompatible.embeddingModel(model)
}
```
