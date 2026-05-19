# 模型推荐实现

当前模型实现集中在 `llm/index.ts`，推荐保持“动态模型 + OpenAI-compatible 适配”的思路。模型选择从 `requestContext.__model__` 读取；认证信息优先走项目统一请求层或模型配置，不建议放进 Mastra requestContext。

```ts
import type { MastraModelConfig } from '@mastra/core/llm'
import type { DynamicArgument } from '@mastra/core/types'
import { createOpenAICompatible } from '@ai-sdk/openai-compatible'

export function createModel(model: LlmUsed = DEFAULT_LLM) {
  if (typeof model === 'string') {
    const provider = llmInfos[model].provider
    const openAICompatible = createOpenAICompatible({
      name: provider,
      baseURL: `${API_BASE}/llm/${provider}`,
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
  modelId?: LlmUsed
}): DynamicArgument<MastraModelConfig> {
  return ({ requestContext }) => {
    const model = requestContext.get<string, LlmUsed>('__model__')
    return createModel(config?.modelId ?? model)
  }
}
```

```ts
export function createEmbedding(model = EmbeddingModel.TEXT_EMBEDDING_V4) {
  const provider = embeddingInfos[model].provider
  const openAICompatible = createOpenAICompatible({
    name: provider,
    baseURL: `${API_BASE}/llm/${provider}`,
  })
  return openAICompatible.embeddingModel(model)
}
```
