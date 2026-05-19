# MCP 推荐实现

推荐保持“请求上下文声明启用哪些 server，后端动态加载哪些 tools”的方向；单个 MCP 服务失败时返回 `{}`，不要让整个聊天不可用。

```ts
export async function getDefaultMcpTools() {
  try {
    const mcpClient = new MCPClient({
      id: 'default-mcp-client',
      servers: {
        workspace: {
          url: new URL(process.env.WORKSPACE_MCP_URL!),
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
    const servers = requestContext.get<string, Record<string, McpServerConfig>>('__mcp_servers__')
    const customTools = await getCustomMcpTools(servers)
    const defaultTools = await getDefaultMcpTools()

    return {
      internetWebpage,
      ...(config?.withChain === false ? {} : { thoughtChain }),
      ...defaultTools,
      ...customTools,
    }
  }
}
```
