# MCP 推荐实现

MCP 工具可以参考 `tools/mcp-tool.ts` 和 `tools/dynamic-tools.ts`。推荐保持“前端启用哪些 server，后端动态加载哪些 tools”的方向；单个 MCP 服务失败时返回 `{}`，不要让整个聊天不可用。

```ts
export async function getBuiltMcpTools() {
  try {
    const mcpClient = new MCPClient({
      id: 'build-mcp-server',
      servers: {
        electron: {
          url: new URL(`http://localhost:${MCP_PORT}/mcp`),
          connectTimeout: 1000 * 20,
        },
      },
    })
    return await mcpClient.listTools()
  }
  catch {
    return {}
  }
}
```

```ts
export function dynamicTools(config?: {
  withChain?: boolean
}): AgentConfig['tools'] {
  return async ({ requestContext }) => {
    const servers = requestContext.get<string, Record<string, CustomMcpServerConfig>>('__mcp_servers__')
    const customTools = await getCustomMcpTools(servers)
    const builtTools = await getBuiltMcpTools()

    return {
      internetWebpage,
      ...(config?.withChain === false ? {} : { thoughtChain }),
      ...builtTools,
      ...customTools,
    }
  }
}
```
