# agile-plugins（Claude Code 插件市场）

[![Validate](https://github.com/pig0224/agile-plugins/actions/workflows/validate.yml/badge.svg)](https://github.com/pig0224/agile-plugins/actions/workflows/validate.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)

本仓库是 **Claude Code 插件市场（marketplace）**：`.claude-plugin/marketplace.json` 声明本仓库提供的全部插件。**新增插件无需改动 [agile-cli](https://github.com/pig0224/agile-cli)** —— 在本仓库加目录 + 登记 marketplace.json 即可。以 git 仓库分发，推送即发版，无需 npm。

## 安装（用户视角）

```bash
# 方式一：在 agile workspace 内（推荐，marketplace 地址可配置）
agile plugin install agile

# 方式二：手动
claude plugin marketplace add <本仓库 git 地址>
claude plugin install agile
```

## 目录结构

```
├── .claude-plugin/marketplace.json   # 市场清单：name + source（相对路径）
└── plugins/
    └── agile/                        # SDD/TDD 主插件
        ├── .claude-plugin/plugin.json
        ├── commands/                 # 12 个 /agile:xxx 斜杠命令
        ├── agents/                   # 7 个角色 subagent
        ├── skills/sdd-tdd-method/    # 共享方法论
        └── .mcp.json                 # 捆绑 agile mcp MCP Server
```

## 新增一个插件

1. 新建 `plugins/<plugin-name>/`，内含 `.claude-plugin/plugin.json`（必填 `name`）及 commands/agents/skills 等
2. 在 `.claude-plugin/marketplace.json` 的 `plugins[]` 追加条目：

```json
{ "name": "<plugin-name>", "description": "…", "source": "./plugins/<plugin-name>" }
```

3. 提交推送后，用户侧 `agile plugin install <plugin-name>` 即可安装（CLI 从 workspace.yaml 的 `plugin.marketplace` 读取本市场地址）

## 约定

- 插件名（plugin.json `name`）= marketplace 条目名 = `plugins/` 下目录名，三者一致
- 插件文案全部中文；agent 的 `description` 用第三人称描述"何时使用"
- 市场名固定 `fcc`（`claude plugin install <name>@fcc`）

## 开发

```bash
claude plugin validate .             # 校验清单（CI 同款）
claude plugin marketplace add .      # 本地市场（命令文件改动后新会话即生效）
```

设计文档：[docs/design.md](./docs/design.md)；开发约定见 [CLAUDE.md](./CLAUDE.md)。

## License

[MIT](./LICENSE) © fcc contributors
