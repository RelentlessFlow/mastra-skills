# Hono 服务端推荐实现

服务端可以用 Hono + Mastra 集成：创建 Hono app，初始化 Mastra，按需挂自定义路由，再用 `MastraServer` 接入标准 Mastra API。

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
    port: Number(process.env.PORT ?? 4111),
  })
}
```

```ts
app.post('/api/chat-ui', async ({ req }) => {
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

注意：不要假设 `@mastra/react` 一定走自定义路由。优先确认所用版本的 `@mastra/react` / `@mastra/client-js` 行为；自定义路由更适合作为业务自有 AI SDK UI-message stream 端点参考。
