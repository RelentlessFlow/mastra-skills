# Request Context 推荐实现

前端可以通过 Mastra `requestContext` 传运行时动态值。当前推荐只传业务上下文、模型选择、提示词、工作区、MCP 配置等信息；认证信息走项目统一请求层或模型配置。

推荐字段：

- `__model__`：当前选择的模型。
- `__system_prompt__`：用户自定义系统提示词。
- `__project_prompt__`：项目提示词。
- `__workspace__`：工作区绝对路径。
- `__mcp_servers__`：启用的 MCP servers。

```ts
export function readRuntimeContext(requestContext: RequestContext) {
  return {
    model: requestContext.get<string, LlmUsed>('__model__'),
    systemPrompt: requestContext.get<string, string>('__system_prompt__') || '',
    projectPrompt: requestContext.get<string, string>('__project_prompt__') || '',
    workspace: requestContext.get<string, string | undefined>('__workspace__'),
    mcpServers: requestContext.get<string, Record<string, CustomMcpServerConfig>>('__mcp_servers__'),
  }
}
```
