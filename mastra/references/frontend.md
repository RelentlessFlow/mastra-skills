# Mastra 前端示例

## 前端 SDK 概述

```ts
import { MASTRA_PORT } from '@constants'
import { MastraClient } from '@mastra/client-js'

/** mastra 客户端实例 Hook */
export { useMastraClient } from '@mastra/react'

/** mastra 客户端实例 */
export const mastraClient = new MastraClient({
  baseUrl: `http://localhost:${MASTRA_PORT}`,
})
```

```tsx
<MastraReactProvider baseUrl={`http://localhost:${MASTRA_PORT}`}>
  <App />
</MastraReactProvider>
```

## Ant Design X 组件示例

推荐用 `@mastra/client-js` / `@mastra/react` 处理数据和流，用 `@ant-design/x` / `@ant-design/x-markdown` 处理聊天 UI。当前项目已有这些参考代码：

- `projects/desktop/src/provider/antd.tsx`：`XProvider` 主题配置。
- `projects/desktop/src/views/home/index/chat/history/task-list/index.tsx`：`Conversations` 历史会话列表。
- `projects/desktop/src/views/home/index/chat/thread/message/index.tsx`：`Bubble.List`、`Actions`、`Think`、`ThoughtChain`、`Sources`。
- `projects/desktop/src/components/z-markdown.tsx`：`XMarkdown`、`CodeHighlighter`、LaTeX 插件。
- `projects/desktop/src/views/home/index/chat/thread/sender/index.tsx`：输入框目前是业务自定义 Tiptap，不强制改成 Ant Design X。

### XProvider 主题

```tsx
import { XProvider } from '@ant-design/x'

/**
 * Ant Design X 主题配置，继承 Ant Design 主题
 */
export const XConfigProvider: FC<{ children: React.ReactNode }> = ({ children }) => {
  return (
    <XProvider
      theme={{
        components: {
          Conversations: {
            paddingSM: getCssVarSize('--spacing-3xs'),
            controlHeightLG: 28,
          },
        },
      }}
    >
      {children}
    </XProvider>
  )
}
```

### Conversations 历史列表

```tsx
import type { ConversationsProps } from '@ant-design/x'
import type { GetProp } from 'antd'
import { Conversations } from '@ant-design/x'

/** 会话列表 */
const items: GetProp<ConversationsProps, 'items'> = (threads?.threads ?? []).map(v => ({
  key: v.id,
  label: (
    <div className={'flex gap-3xs'}>
      <span className={'truncate flex-1'}>{v.title || 'New Thread'}</span>
      <ZIconButton
        className={'group-hover:visible invisible'}
        color={'negative'}
        icon={IconTrash}
        size={'xs'}
        onClick={(e) => {
          e?.stopPropagation()
          deleteThread(v.id)
        }}
      />
    </div>
  ),
  className: 'group',
  group: getGroupLabel(v.updatedAt),
}))

return (
  <Conversations
    groupable
    activeKey={threadId}
    className={'w-60 max-h-100'}
    items={items}
    onActiveChange={(value) => {
      selectThread(value)
      onSelect()
    }}
  />
)
```

### Bubble.List 消息列表

```tsx
import type { BubbleItemType, BubbleListProps } from '@ant-design/x'
import { Actions, Bubble, Think } from '@ant-design/x'

/** 角色描述 */
const role: BubbleListProps['role'] = {
  assistant: {
    variant: 'borderless',
    loadingRender: () => (
      <div className={'flex flex-center gap-xs'}>
        <Spin size={'small'} />回复中...
      </div>
    ),
    classNames: {
      root: 'px-4xs py-xs',
      footer: 'mt-xs',
    },
    footer: content => <BubbleActions content={content} />,
  },
  user: {
    placement: 'end',
    avatar: () => <Avatar icon={<ZIcon icon={IconUser} />} size={32} />,
    classNames: {
      root: 'px-4xs',
      footer: 'mt-xs',
    },
    footer: content => <BubbleActions content={content} />,
  },
}

/** 列表信息 */
const items: BubbleListProps['items'] = messages.map(({ id, role, parts, metadata }) => {
  const content: BubbleItemType['content'] = parts.map((part, index) => {
    if (part.type === 'reasoning') {
      return (
        <Think key={`${id}-${index}`} defaultExpanded={false} title={'思考中'}>
          {part.text}
        </Think>
      )
    }

    if (part.type === 'text') {
      return (
        <span key={`${id}-${index}`} data-content={part.text}>
          {part.text}
        </span>
      )
    }

    return null
  })

  return {
    key: id,
    role,
    loading: metadata?.mode === 'stream' && !parts?.length,
    status: metadata?.mode === 'stream' ? 'updating' : 'local',
    content,
  }
})

return (
  <Bubble.List
    ref={listRef}
    items={items}
    role={role}
  />
)
```

### Bubble Actions

```tsx
export const BubbleActions: FC<{ content: ReactNode }> = ({ content }) => {
  const extractContent = (): string => {
    if (!Array.isArray(content))
      return ''

    return content.reduce((res, v) => {
      const element = v as ReactElement<any>
      const cur = element?.props?.['data-content']
      return !cur
        ? res
        : res ? `${res}\n\n${cur}` : cur
    }, '')
  }

  return (
    <Actions
      items={[{
        key: 'copy',
        icon: <ZIcon icon={IconCopy} />,
        label: '复制内容',
      }]}
      onClick={() => {
        navigator.clipboard?.writeText(extractContent())
        message.success('已复制到剪贴板')
      }}
    />
  )
}
```

### XMarkdown 渲染

```tsx
import type { ComponentProps, XMarkdownProps } from '@ant-design/x-markdown'
import { CodeHighlighter } from '@ant-design/x'
import { XMarkdown } from '@ant-design/x-markdown'
import Latex from '@ant-design/x-markdown/plugins/Latex'
import '@ant-design/x-markdown/themes/dark.css'
import '@ant-design/x-markdown/themes/light.css'

const CustomCode: FC<ComponentProps> = ({ children, className }) => {
  const { derives: { light } } = useTheme()
  const lang = className?.match(/language-(\w+)/)?.[1] || ''

  return (
    typeof children === 'string' && (
      <CodeHighlighter
        classNames={{ code: '[&_pre]:m-0!' }}
        highlightProps={{ style: light ? oneLight : atomDark }}
        lang={lang}
      >
        {children}
      </CodeHighlighter>
    )
  )
}

export const ZMarkdown: FC<XMarkdownProps> = (props) => {
  const { derives: { light } } = useTheme()

  return (
    <XMarkdown
      className={`x-markdown-${light ? 'light' : 'dark'}`}
      components={{ code: CustomCode }}
      config={{ extensions: Latex() }}
      {...props}
    />
  )
}
```

## @mastra/client-js 示例

```ts
const agentsRecord = await mastraClient.listAgents()
const agents = Object.values(agentsRecord)

const { result: memoryEnabled } = await mastraClient.getMemoryStatus(agentId)

const threadResult = await mastraClient.listMemoryThreads({
  agentId,
  resourceId,
  perPage: 9999,
})
const threads = threadResult.threads

const { messages } = await mastraClient.listThreadMessages(threadId, { agentId })
```

```ts
export async function mastraGetAgents() {
  const agentsRecord = await mastraClient.listAgents()
  return Object.values(agentsRecord).map(agent => ({
    ...agent,
    icon: iconMap[agent.id],
  }))
}
```

## @mastra/react 示例

```ts
import type { RequestContext } from '@mastra/core/request-context'
import { useChat as useChatSdk } from '@mastra/react'

const {
  messages,
  sendMessage,
  cancelRun,
  isRunning,
  setMessages,
  ...sdkProps
} = useChatSdk({
  agentId,
  resourceId,
  initialMessages,
})

await sendMessage({
  message: input,
  mode: 'stream',
  threadId,
  onChunk: async (chunk) => {
    if (chunk.type === 'finish')
      getThreads()
  },
  signal: controller.signal,
  requestContext: {
    __model__: llmUsed,
    __system_prompt__: systemPrompt,
    __project_prompt__: projectPrompt,
    __workspace__: root,
    __mcp_servers__: getEnabledServers(),
  } as unknown as RequestContext,
})
```

```ts
function onCancel() {
  cancelRun()
  abortControllerRef.current?.abort()
  abortControllerRef.current = null
}
```

## API 调用示例

```ts
const $http = axios.create({
  baseURL: `http://localhost:${MASTRA_PORT}`,
})

export async function mastraGetAgentsByHttp() {
  const { data } = await $http.get<Record<string, Agent>>('/api/agents')
  return Object.values(data)
}
```

```ts
export async function mastraChatByHttp(params: {
  agentId: string
  messages: unknown[]
  threadId?: string
  resourceId?: string
  requestContext?: RequestContext
}) {
  return $http.post('/api_c/chat', params)
}
```
