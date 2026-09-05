# agile-plugins 设计

> Claude Code 插件市场：15 个斜杠命令 + 7 个角色 subagent + 1 个方法论 skill。本仓库独立分发（git），**新增插件无需升级 [agile-cli](https://github.com/pig0224/agile-cli)**。

## 1. 市场仓库结构

```
agile-plugins/                       # 插件市场仓库
├── .claude-plugin/marketplace.json  # 市场清单（name: fcc）
│                                    #   plugins[]: { name, source: "./plugins/<name>" }
└── plugins/
    └── agile/                       # SDD/TDD 主插件
        ├── .claude-plugin/plugin.json
        ├── commands/                # 15 个命令（安装后 /agile:xxx）
        ├── agents/                  # 7 个角色 subagent
        ├── skills/sdd-tdd-method/   # 共享方法论
        └── .mcp.json                # 捆绑 agile mcp MCP Server
```

**职责分离**：命令 = 人机入口（前置校验 + 委派 + 复核汇报）；agent = 具体执行（产出文档/代码）；skill = 共享知识（所有命令开头要求先读）。命令体内不写实现细节，保证角色 prompt 集中且可独立演化。

## 2. 安装链路（CLI 零知识）

```
agile plugin install [name] [--marketplace <url>]
  → 读 workspace.yaml plugin.marketplace（默认官方 git 地址，可指向团队私有市场）
  → claude plugin marketplace add <git 地址>
  → claude plugin install <name>@fcc
  → 记录 .agile/plugin.yaml（source = 市场地址）
```

新增插件：市场仓库加 `plugins/<name>/`（含 `.claude-plugin/plugin.json`）+ 登记 marketplace.json 的 `plugins[]`。用户侧 `agile plugin install <name>` 即完成，CLI 不发版。

约定：插件名（plugin.json `name`）= marketplace 条目名 = `plugins/` 下目录名，三者一致；市场名固定 `fcc`。

## 3. 流程主线（SDD）

```
需求 → /agile:prd → /agile:sync-req → /agile:architect → /agile:gen-test(Stage1)
     → /agile:backend | /agile:frontend (TDD) → /agile:run-test(Stage2)
     → review.md / release.md 归档
```

两条红线，写进 skill 与各 agent：
1. **SDD**：没有 `design.md` 不进入开发（architect/backend/frontend 命令都有前置校验）
2. **TDD**：没有失败测试不写实现（Red → Green → Refactor，循环记录登记进 implementation.md）

## 4. 命令 ↔ 角色映射

| 命令 | 委派 agent | 产物（落盘位置） |
|---|---|---|
| /agile:prd | product-manager | PRD/AC/功能树/菜单树 → `<抽屉三>/requirements/<编号>/` |
| /agile:sync-req | -（自身执行） | 复制到 `process-docs/<编号>/`，requirement.md 并入 AC |
| /agile:architect | tech-architect | `process-docs/<编号>/design.md` |
| /agile:gen-test | test-engineer | `process-docs/<编号>/gen-test.md`（案例表 + 自动化映射） |
| /agile:backend | backend-dev | worktree 内 TDD 实现 + implementation.md 更新 |
| /agile:frontend | frontend-dev | 接口层→组件层→页面层 分层实现 + 浏览器验证 |
| /agile:ui | ui-designer | 组件库 / `<抽屉三>/prototypes/<编号>/` / CHANGELOG |
| /agile:run-test | test-engineer | `process-docs/<编号>/run-test.md`（验收报告） |
| /agile:review | -（主会话直接执行，**分工例外**） | `process-docs/<编号>/review.md`（验收矩阵 + 门禁判定，不代验收） |
| /agile:release | -（主会话直接执行，**分工例外**） | `process-docs/<编号>/release.md`（前置检查 + 回滚方案 + 发布记录） |
| /agile:fix-bug | bug-hunter | 最小修复 + 复现测试 + 文档登记（无编号则 BUG-xxx） |
| /agile:add-task | -（只追加） | implementation.md 任务清单追加 |
| /agile:feedback | -（收集会话） | `process-docs/<编号>/feedback-<日期>.md` |
| /agile:knowledge | -（主会话直接执行，**分工例外**） | build：知识库骨架 + 提纲 + README 导航；capture：会话/过程产物提炼的长期结论文档 + README 导航 |
| /agile:help | -（静态） | 命令总览 + 流程图 + workspace 状态 |

## 5. 与 CLI/MCP 的协作

命令体指示模型通过以下方式调用 CLI 能力（不手工造 git 命令）：
- Bash 执行 `agile status / worktree create / doctor / template list`
- 捆绑 MCP 工具（`agile mcp`）：`agile_status`、`agile_sync`（默认 dryRun）、`agile_template_list`、`agile_task_create`（task 能力仅 MCP 暴露，/agile:sync-req 等命令经它创建任务目录）、`agile_doctor` 等

抽屉路径不硬编码：所有命令/agent 先读 `.agile/workspace.yaml` 的 `paths` 段。

## 6. 规范引用优先级（写入 agent prompt）

1. `tech-specs/`（抽屉一，公司硬规范，冲突时最高优先级）
2. `biz-tech-docs/`（抽屉二，团队设计，保持一致、禁止重复造轮子）
3. `biz-product-docs/`（抽屉三，产品/UI 规范，前端必须对齐）
4. 当前任务 `design.md`（本次具体决策）

## 7. 命令 frontmatter 约定

- `description`：中文、动词开头，是模型自动触发的依据，须准确描述功能
- `argument-hint`：提示参数形态（如 `<需求编号> [仓库路径]`）
- `disable-model-invocation: true`：仅人工触发的命令（help/feedback）
- 委派类命令不设 `allowed-tools`（委派的 agent 自带 tools 白名单）

## 8. 校验

CI 中 `claude plugin validate .` 校验市场清单与插件清单合法性。
