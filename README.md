# mastra-skills

Mastra 参考实现与开发偏好 skill。

## 内容

- `mastra/SKILL.md`：主说明，保持精简，只放适用范围、推荐方向和 reference 导航。
- `mastra/references/`：后端、前端示例代码，按颗粒度拆分，按需读取。

## 使用

安装到支持 Skills standard 的 coding agent：

```bash
pnpm dlx skills add <github-user>/mastra-skills
```

本地开发时也可以直接把 `mastra/` 目录复制到 `$CODEX_HOME/skills` 或项目 `.agents/skills` 下。
