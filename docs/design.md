# agile-plugins 设计

> Claude Code 插件市场：18 个斜杠命令 + 7 个角色 subagent + 1 个方法论 skill。本仓库独立分发（git），**新增插件无需升级 [agile-cli](https://github.com/pig0224/agile-cli)**。

## 1. 市场仓库结构

```
agile-plugins/                       # 插件市场仓库
├── .claude-plugin/marketplace.json  # 市场清单（name: fcc）
│                                    #   plugins[]: { name, source: "./plugins/<name>" }
└── plugins/
    └── agile/                       # SDD/TDD 主插件
        ├── .claude-plugin/plugin.json
        ├── commands/                # 18 个命令（安装后 /agile:xxx）
        ├── agents/                  # 7 个角色 subagent
        └── skills/sdd-tdd-method/   # 共享方法论（附录 A = 任务目录模板）
```

**职责分离**：命令 = 人机入口（前置校验 + 委派 + 复核汇报）；agent = 具体执行（产出文档/代码）；skill = 共享知识（所有命令开头要求先读）。命令体内不写实现细节，保证角色 prompt 集中且可独立演化。

## 2. 安装链路（CLI 零知识）

```
agile plugin install [name]
  → 读 .agile/settings.json 的 plugins.marketplace（默认官方 git 地址，可指向团队私有市场）
  → claude plugin marketplace add <git 地址>
  → claude plugin install <name>@fcc
  → 记录到 .agile/settings.json 的 plugins.dependencies（{ marketplace }）
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
| /agile:sync-req | -（主会话直接执行，**分工例外**） | 复制到 `process-docs/<编号>/`，requirement.md 并入 AC |
| /agile:architect | tech-architect | `process-docs/<编号>/design.md` |
| /agile:gen-test | test-engineer | `process-docs/<编号>/gen-test.md`（案例表 + 自动化映射） |
| /agile:backend | backend-dev | worktree 内 TDD 实现 + implementation-be.md 更新 |
| /agile:frontend | frontend-dev | 接口层→组件层→页面层 分层实现 + 浏览器验证 |
| /agile:ui | ui-designer | 组件库 / `<抽屉三>/prototypes/<编号>/` / CHANGELOG |
| /agile:run-test | test-engineer | `process-docs/<编号>/run-test.md`（验收报告） |
| /agile:review | -（主会话直接执行，**分工例外**） | `process-docs/<编号>/review.md`（验收矩阵 + 门禁判定，不代验收） |
| /agile:release | -（主会话直接执行，**分工例外**） | `process-docs/<编号>/release.md`（前置检查 + 回滚方案 + 发布记录） |
| /agile:fix-bug | bug-hunter | 最小修复 + 复现测试 + 文档登记（无编号则 BUG-xxx） |
| /agile:add-task | -（主会话直接执行，**分工例外**：只追加） | `implementation.md` 任务分配表追加一行（主文件冻结后只读，仅允许此追加） |
| /agile:feedback | -（主会话直接执行，**分工例外**：收集会话） | `process-docs/<编号>/feedback-<日期>.md` |
| /agile:knowledge | -（主会话直接执行，**分工例外**） | build：知识库骨架 + 提纲 + 根导航登记；capture：会话/过程产物提炼的长期结论文档 + 导航双登记（根导航 + 模块内 README）；sync-template：组合模板根耦合资产按 `类型: tech\|product` frontmatter 同步进 biz-tech-docs / biz-product-docs（快照来自 `.agile/solutions/<组合>/`，`init project` 带出） |
| /agile:init | -（主会话直接执行，**分工例外**）·负责人 | AI 陪同建项目：`agile init project` 骨架生成 + 项目约定问答定制 + 团队库匹配确认 + 辅助开发能力配置（环境检测 / 框架 AI 能力包） |
| /agile:add-template | -（主会话直接执行，**分工例外**）·负责人/模板维护者 | AI 辅助建设模板：agile-templates 骨架 + registry.json 登记 + check / 冒烟验证（仅限模板仓根目录使用） |
| /agile:share-template | -（主会话直接执行，**分工例外**）·架构/模板维护者 | workspace 项目 → 模板：清理审计 + 占位符还原 + 打包落盘 + registry.json 登记 + check / 冒烟（仅限 workspace 内使用） |
| /agile:help | -（主会话直接执行，**分工例外**：静态信息） | 命令总览 + 流程图 + workspace 状态 |

## 5. 与 CLI 的协作

命令体指示模型通过 Bash 调用 CLI 能力（不手工造 git 命令）：
- Bash 执行 `agile sync / config / worktree create / template list / plugin ...`

任务目录（process-docs/<编号>/，初始 8 个 .md）由创建它的命令按 sdd-tdd-method SKILL 附录 A 模板直接创建（幂等，无 CLI/MCP 依赖）。

抽屉路径不硬编码：所有命令/agent 先读 `.agile/settings.json` 的 `paths` 段。

## 6. 规范引用优先级（写入 agent prompt）

1. `tech-specs/`（抽屉一，公司硬规范，冲突时最高优先级）
2. `biz-tech-docs/`（抽屉二，团队设计，保持一致、禁止重复造轮子）
3. `biz-product-docs/`（抽屉三，产品/UI 规范，前端必须对齐）
4. 项目级约定：`projects/<name>/CLAUDE.md`（入口索引：技术栈 / 命令速查 / 硬规则）与其指向的 `docs/conventions.md`、`docs/architecture.md`（由模板经 `init project` 生成，随项目入库；每个项目各一份）
5. 当前任务 `design.md`（本次具体决策）

三条硬规则（与 sdd-tdd-method SKILL §6 一致）：**有效性过滤**——frontmatter `状态` 为「已废弃 / 已被替代」的知识条目不得引用；**技术栈选择性引用**——tech / team 库只取与当前项目技术栈匹配的领域 + 通用领域；**团队库匹配人工确认**——项目 CLAUDE.md 带「⛔ 栈领域待人工确认」标记时，先 AI 提问、人工回答确定 `frameworks/` 领域匹配，未确认前只引用通用领域。

## 7. 命令 frontmatter 约定

- `description`：中文，以产物/职能命名、简洁达意即可，是模型自动触发的依据，须准确描述功能
- `argument-hint`：提示参数形态（如 `<需求编号> [仓库路径]`）
- `disable-model-invocation: true`：仅人工触发的命令（help/feedback/review/release）
- 委派类命令不设 `allowed-tools`（委派的 agent 自带 tools 白名单）

## 8. 校验

CI 中 `claude plugin validate .` 校验市场清单与插件清单合法性。
