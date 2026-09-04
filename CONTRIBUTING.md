# 贡献指南（CONTRIBUTING）

感谢关注 agile-plugins（Claude Code 插件市场）！欢迎提交新插件、改进现有插件或文档。

## 环境搭建

```bash
git clone git@github.com:pig0224/agile-plugins.git
cd agile-plugins
claude plugin validate .            # 校验清单（需本机安装 Claude Code）
claude plugin marketplace add .     # 本地市场（开发热加载：改文件重启会话即生效）
```

## 新增一个插件

1. 新建 `plugins/<插件名>/`，内含 `.claude-plugin/plugin.json`（必填 `name`）及 commands/agents/skills
2. 在 `.claude-plugin/marketplace.json` 的 `plugins[]` 登记条目
3. **一致性铁律**：插件名（plugin.json `name`）= marketplace 条目名 = 目录名，三者一致

## 提交流程

1. 分支开发（`feat/<描述>` 或 `fix/<描述>`），推送后开 PR 指向 main
2. CI 自动执行 `claude plugin validate .`，全绿是合并前提
3. 维护者 review（CODEOWNERS 自动请求）后合并——**merge 即发版**，所有用户立即可用

## 插件内容约定

- 命令只做「前置校验 → Task 委派 agent → 复核汇报」，实现细节写在 agent 里
- 共享知识放 skill；产物全中文；不硬编码抽屉路径与 git 命令（经 workspace.yaml / CLI / MCP）
- 文件系统操作经 CLI/MCP（如任务目录用 MCP `agile_task_create`），不手搓命令
- 有写操作的 MCP 工具保持 dry-run 默认或显式确认参数

## 报告问题

使用 [issue 模板](https://github.com/pig0224/agile-plugins/issues/new/choose)。行为准则见 [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)。
