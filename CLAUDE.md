# CLAUDE.md — agile-plugins 仓库导航

本仓库是 **Claude Code 插件市场（marketplace）**：`.claude-plugin/marketplace.json` 声明全部插件，主插件在 `plugins/agile/`。**新增插件无需改动 agile-cli**（CLI 从 workspace.yaml 的 `plugin.marketplace` 指向本仓库）。

## 常用命令

```bash
claude plugin validate .                       # 校验市场清单与插件清单
claude plugin marketplace add .                # 本地安装（开发热加载）
claude plugin install agile
```

## 结构

```
.claude-plugin/marketplace.json    # 市场清单：plugins[] → source（相对路径）
plugins/agile/
  .claude-plugin/plugin.json       # 插件清单（name: agile）
  commands/                        # 12 个 /agile:xxx 斜杠命令
  agents/                          # 7 个角色 subagent
  skills/sdd-tdd-method/           # 共享方法论（命令按需引用）
  .mcp.json                        # 捆绑 agile mcp MCP Server
docs/design.md                     # 设计文档
```

## 关键约定

- 新增插件：新建 `plugins/<name>/`（含 `.claude-plugin/plugin.json`）+ 在 marketplace.json `plugins[]` 追加条目
- 插件名（plugin.json `name`）= marketplace 条目名 = `plugins/` 目录名，三者一致；市场名固定 `fcc`
- 插件文案全部中文；agent 的 `description` 用第三人称描述"何时使用"（Task 委派触发依据）
- 命令只做「前置校验 → Task 委派 agent → 复核汇报」，不写实现细节
- 两条 SDD/TDD 红线不得削弱：无 design.md 不开发；无失败测试不写实现
- 推送即发版（无版本号，无 npm）
