# 工作区与 Skills 推荐实现

工作区可以通过 `requestContext.__workspace__` 动态创建文件系统，并按业务启用 workspace tools 和 skills。

```ts
export function dynamicWorkspace(
  fileSystemOptions?: Partial<LocalFilesystemOptions>,
): AgentConfig['workspace'] {
  return ({ requestContext }) => {
    const basePath = requestContext.get<string, string | undefined>('__workspace__')
    const filesystem = basePath
      ? new LocalFilesystem({
          basePath,
          contained: false,
          ...fileSystemOptions,
        })
      : undefined

    return new Workspace({
      id: 'dynamic-workspace',
      name: 'Dynamic Workspace',
      filesystem,
      skills: ['./**/skills', '.chat/marketing-skills'],
      bm25: true,
      autoIndexPaths: ['.'],
      tools: {
        enabled: true,
        requireApproval: false,
        [WORKSPACE_TOOLS.FILESYSTEM.DELETE]: {
          requireApproval: true,
        },
      },
    })
  }
}
```

Mastra 官方也提供 Skills，可作为外部参考来源：

```bash
pnpm dlx skills add mastra-ai/skills
pnpm dlx skills update mastra
```

文档地址：https://mastra.ai/docs/build-with-ai/skills
