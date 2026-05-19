# Hono 服务端推荐实现

当前服务端集成在 `setup/index.ts`，可作为 Hono + Mastra 的参考：创建 Hono app，初始化 Mastra，按需挂自定义路由，再用 `MastraServer` 接入标准 Mastra API。

```ts
export async function setupMastra() {
  const app = new Hono<{ Bindings: HonoBindings, Variables: HonoVariables }>()
  const mastra = initMastra()

  const server = new MastraServer({
    app,
    mastra,
    openapiPath: '/openapi.json',
  })
  await server.init()

  mastraServer = serve({
    fetch: app.fetch,
    port: MASTRA_PORT,
  })
}
```

```ts
app.post('/api_c/chat', async ({ req }) => {
  const body = await req.json()
  const { agentId, ...params } = body
  const stream = await handleChatStream({
    mastra,
    agentId,
    params,
  })

  return createUIMessageStreamResponse({ stream })
})
```

```ts
export function cleanMastra() {
  if (mastraServer) {
    mastraServer.close()
    mastraServer = null
  }
}
```

注意：不要假设 `@mastra/react` 一定走 `/api_c/chat`。当前安装的 `@mastra/react@0.2.21` 通过 `@mastra/client-js` 的 agent stream 标准接口发送；`/api_c/chat` 更适合作为项目自定义 AI SDK UI-message stream 端点参考。
