---
name: sdd-tdd-method
description: agile 工作区的 SDD/TDD 研发方法论与文档规范。凡执行 agile:prd / agile:architect / agile:backend / agile:frontend / agile:gen-test / agile:run-test 等 agile 系列命令时必须先阅读本 skill。涵盖：一个根五个抽屉的目录约定、需求编号任务目录、SDD 先设计后开发、TDD Red-Green-Refactor 循环、过程产物五文档 + 两角色卫星文件（任务目录初始 8 个 .md，完整档案 9 个）的填写规范。
---

# agile SDD/TDD 方法论

## 1. 工作区结构（一个根、五个抽屉）

工作区根 = 存在 `.agile/settings.json` 的目录。**一切路径先读 `.agile/settings.json` 的 `paths` 段获得**，默认约定：

| 抽屉 | 默认路径 | 内容 | 角色 |
|---|---|---|---|
| 一 | `tech-specs/` | 公司级技术规范（技术栈、SQL、安全硬规范） | 全员遵守 |
| 二 | `biz-tech-docs/` | 团队技术设计知识库（架构、状态机、技术方案、工程规范） | 架构师 |
| 三 | `biz-product-docs/` | 产品设计知识库（PRD、产品规范、UI 规范、交互规范） | 产品经理 |
| 四 | `projects/` | 项目代码（workspace 单仓内普通目录；单例与组合模板成员项目全部平铺） | 开发 |
| 五 | `process-docs/` | 过程产物（按需求编号归档，workspace 根仓库内） | 全员 |

CLI 直调：工作区操作（sync / config / worktree / template / plugin 等）通过 Bash 执行 `agile <command>`（CLI 是插件的硬依赖，未安装时先提示用户 `npm i -g fcc-agile-cli`）。**submodule 已废弃（2.0）：tech-specs 与 biz-tech-docs 默认都是 workspace 内普通目录（随仓库提交获得版本管理；登记为外部仓库后独立成库、被 .gitignore 忽略、由 agile sync 管理）——commit/push 一律由人工处理，AI 不代做。**

## 2. 需求编号任务目录与通道判定（STO / BUG / OPS）

`process-docs/<编号>/` 标准任务目录（STO-xxx 业务需求 / BUG-xxx 缺陷修复 / OPS-xxx 技术变更），**由创建它的插件命令（/agile:sync-req、/agile:fix-bug；后者无编号时由其委派的 bug-hunter subagent 创建）按附录 A 模板直接创建**（幂等：已存在的文件不覆盖）。五文档 + 两份角色卫星文件 + gen-test.md 骨架（模板见附录 A），初始创建共 8 个 .md：

- `requirement.md` — 需求说明与验收标准（AC）。产品/需求侧填充。
- `design.md` — 技术设计。**SDD 核心：开发前必须先完成**。参考抽屉一/二规范。
- `implementation.md` — 实施记录主文件：**任务分配表**（由 /agile:architect 在 design 冻结时填写，之后**只读，仅允许按 design 冻结结论追加一行**（add-task））+ 联调约定。
- `implementation-be.md` — **后端专属**实施记录：任务清单、TDD 循环记录、变更清单。前端禁写。
- `implementation-fe.md` — **前端专属**实施记录：任务清单、测试记录、变更清单。后端禁写。
- `review.md` — 评审记录。
- `release.md` — 发布记录与回滚方案。

> 文件级隔离：前后端并行开发（同一需求分支）时各写各的角色文件，git 合并零冲突。测试案例文档 gen-test.md 同理分「后端用例」「前端用例」两节。gen-test.md 骨架随任务目录创建（由 /agile:gen-test 填充，模板见附录 A），run-test.md 由 /agile:run-test 阶段产出（不在附录 A 初始模板之列）——任务目录完整档案共 9 个 .md。

当前需求编号贯穿始终：所有命令产出都写入对应 `process-docs/<编号>/`。

### 通道判定（所有命令先判再动）

**两维正交**：性质（编号前缀）定审批人，深度（完整 / 轻量）定填写量。

| 形态 | 判定 | 典型场景 | 目录创建 |
|---|---|---|---|
| 完整 | 抽屉三 `requirements/<编号>/` 有 PRD 产物 | 需产品定稿 PRD/AC 的需求 | `/agile:sync-req <编号>` |
| STO 轻量 | 产品一句话确认分配 STO 编号，无 PRD | mini feat、文案/样式调整 | `/agile:sync-req <编号> <一句话需求>` |
| BUG | 缺陷，无需审批（回归正确） | 行为与预期不符 | `/agile:fix-bug`（无编号顺延 BUG-xxx） |
| OPS | 技术变更，运维审批 | 重构、依赖升级、CI 微调 | `/agile:sync-req <编号> <改动说明>` |

**轻量机读标记**：`requirement.md` 头部含 `> 本变更走轻量通道` 即轻量形态，各命令按此自适应——architect 输出三五行方案简述（design.md）；review 一行验收确认（报告人确认修复生效（BUG）/ 提需求人确认（STO 轻量）/ 负责人自查（OPS））；run-test 不产出完整 Stage 2 报告，只在 run-test.md 记一行回归/验证结论；release 涉及部署才记一行。**不变**：TDD 红线不豁免（bug 修复必须复现测试 Red→Green）；worktree、main 禁直推、PR、CI 门禁照走。**升级出口**：过程中发现影响面超出预期（涉及接口契约 / 数据模型 / 业务行为明显变化）→ 停止轻量流程，提示用户按团队 SOP「轻量通道」页「编号变更与升级出口」节**人工处理**换号与文档补全——AI 不自行执行编号变更、目录改名或分支操作。详细规范见团队 SOP「轻量通道」节。

## 3. SDD（Spec-Driven Design）流程主线

```
需求输入
  → /agile:prd          （产品经理 subagent：PRD/AC/功能树/菜单树 → 抽屉三）
  → /agile:sync-req     （需求产物同步到 process-docs/STO-xxx，创建标准目录）
  → /agile:architect    （架构师 subagent：design.md，先设计后开发）
  → /agile:gen-test     （测试工程师 Stage 1：基于 AC 产出测试案例文档）
  → /agile:backend / /agile:frontend   （TDD 开发实现）
  → /agile:run-test     （测试工程师 Stage 2：执行测试 + 验收报告）
  → review.md / release.md 归档闭环
```

**硬规则：**
1. 没有 `design.md` 不得进入开发阶段（SDD 红线）。**轻量通道豁免**：STO 轻量 / BUG-xxx / OPS-xxx 编号（判定见 §2 通道判定）下，design.md 可由「根因分析」（BUG）或三五行方案简述（STO 轻量 / OPS）替代；TDD 红线（规则 2）**不豁免**——bug 修复必须先有复现测试。
2. 没有失败测试不得写实现代码（TDD 红线；脚手架/接口签名除外）。
3. 所有产物先落盘到 process-docs，再写代码；代码变更与文档同步更新。
4. **提交红线（add 归人工）**：绝对不执行 `git add`——哪些变更进入提交由人工审阅决定；每个 TDD 循环完成后，把建议的 commit message（`STO-xxx(red|green|refactor): <内容>`）登记到本角色文件（implementation-be.md / implementation-fe.md）。人工 add 完成后，可汇总执行 `git commit`，但 commit 前必须 `git status` 检查：若仍有本次变更相关的未暂存文件，提醒人工补充 add（不得自行 add），确认无遗漏后才提交。`git push` 一律人工；**决不允许发版**（创建/推送 tag、触发 Release workflow 等一切发版动作只能由人工处理）。只读 git 命令（status/log/diff/blame）不受限制。
5. **分工红线（不得反转）**：命令（主会话）负责前置校验、Task 委派与复核汇报，实施一律委派对应角色 subagent；**禁止以「subagent 不可靠」「更快」「上下文更全」等理由改由主会话直接实施、subagent 验收**（个别命令文件显式声明的例外除外）。subagent 拿不到主会话上下文——委派时必须显式传入任务编号、约束与验收要求，不得依赖「它应该知道」；subagent 产出必须经主会话按各命令的复核清单复核后才向用户汇报。
6. **人工修订感知（会话记忆 = 文件）**：任务目录文档是唯一持久记忆——一律重读文件，禁止凭会话内印象行事。开工前按下方「人工修订感知流程」检测人工修改：合理改动采纳并在该文档「修订记录」节登记；与 AC/设计/红线冲突或疑似误改时停下给意见，**返回人工确认后才能继续**。
7. **壳层边界（会话层配置只认 workspace 根）**：Claude Code 会话从 workspace 根（或 worktree 根）启动，只读取**启动目录**的 `.claude/` 与 `.mcp.json`——`projects/<项目>/` 内**不得创建**这两者（写了不生效）；权限白名单 / hooks 归 workspace 根 `.claude/`（AI 出建议清单，人工配置）；项目工程配置（lint / build / test / 依赖）一律项目内自包含，不提升到根。根上守白名单：仅 `.agile/`、根 `CLAUDE.md`、`.gitignore` / `.gitattributes`、`.github/`、`PRD模板.md`、process-docs/、知识库抽屉、projects/ 与团队自选的壳层文件（钩子壳 `.githooks/` 或 `.husky/`、命令分发壳 `Taskfile.yml` / `lefthook.yml` / `Makefile`、配套 `package.json`、`CODEOWNERS`）——根上不落栈专属工具配置，用什么钩子/分发工具由团队自决，AI 不预置。栈联调壳（`go.work`、`pnpm-workspace.yaml`、本地 compose 等）确有联调需要时经确认才建，且不进模板（见 /agile:add-template、/agile:share-template）。

### 人工修订感知流程（硬规则 6 的执行口径）

人工可能在中途或跨会话直接修改已产出文档（PRD/AC、design.md、gen-test.md 等）。各命令**开工前**执行两层检测，把人工修改纳入会话记忆：

1. **git 层**：对本次涉及的文档路径执行 `git status --short -- <路径>` 与 `git diff -- <路径>`——已跟踪文件的未提交改动即上次 commit 后的人工/外部修改（最常见场景）。
2. **语义层**：重读上游文档与下游派生产物对照（AC vs gen-test.md 用例映射、requirement.md/design.md vs 任务清单、design.md 接口 vs 已实现代码）；无 git 基线可用时（改动未被 commit 过）由本层兜底。

**处置**：

- **可采纳**（合理、与 AC/设计/红线兼容）→ 纳入本次执行依据，在该文档「修订记录」节登记一行（`| 日期 | 来源 | 摘要 | 采纳结论 |`，来源标人工/AI、结论标已采纳/待确认），并向用户汇报已采纳项；
- **存疑**（与 AC/设计/红线冲突、疑似误改、逻辑不自洽）→ 停下输出「发现的问题 + 意见」，**返回人工确认**；确认前不调整、不登记；
- **登记纪律**：修订记录只追加不重排；登记前与节内既有行查重，同一改动不重复登记。

附录 A 的 requirement.md / design.md / gen-test.md 模板自带「修订记录」节；其余文档按需沿用同格式（不强制）。

## 4. TDD 循环（Red → Green → Refactor）

每个开发任务在本角色文件（后端 implementation-be.md / 前端 implementation-fe.md）登记，然后：

1. **Red**：根据 design.md 与测试案例写一个失败的测试（运行证明它失败）。
2. **Green**：写最小实现让测试通过（不许过度设计）。
3. **Refactor**：消除重复、改善命名与结构，测试保持绿色。

循环记录写入角色文件的「TDD 循环记录」表格。多任务按依赖顺序逐个循环。

## 5. 测试脚本与产物归属

| 层 | 内容 | 位置 | 是否入 git |
|---|---|---|---|
| 固化 e2e 脚本 | 关键路径回归脚本（长期资产，随页面同 PR 演进，供 /agile:run-test 与 stage 冒烟复用；**默认不进 PR CI 门禁**——e2e flaky 且慢，各项目可选跑关键路径冒烟子集） | 项目内 `e2e/`（Playwright，如 `e2e/*.spec.ts`） | ✅ 提交 |
| 临时验证/复现脚本 | 修 bug 复现脚本、一次性验证脚本 | `process-docs/<编号>/scripts/`（**严禁散落在 projects/ 下的项目内**） | ✅ 随需求分支提交 |
| 运行产物 | 截图、trace、HTML 报告、test-results | 框架默认输出目录 | ❌ 一律 .gitignore，不提交 |
| 报告证据 | run-test / 浏览器验证引用的关键截图（少量） | `process-docs/<编号>/assets/` | ✅ 提交 |

**测试工具约定**：e2e 主要测试工具 **Playwright**，辅助调试 **Chrome DevTools**；项目尚无可用测试工具时，给出「引入 Playwright（主）+ Chrome DevTools（辅助调试）」的建议，经负责人确认后引入。

## 6. 规范引用优先级

写任何代码/文档前：
1. `tech-specs/`（公司硬规范，冲突时优先级最高）
2. `biz-tech-docs/`（团队规范与既有设计，保持一致，禁止重复造轮子）
3. `biz-product-docs/`（产品/UI 规范，前端实现必须对齐）
4. 项目级约定：`projects/<name>/CLAUDE.md`（入口索引：技术栈 / 命令速查 / 硬规则）与其指向的 `docs/conventions.md`、`docs/architecture.md`（由模板经 `init project` 生成，随项目入库；单例与组合成员项目平铺，每个项目各一份）
5. 当前任务 `design.md`（本次的具体决策）

**有效性过滤（硬规则）**：仅使用状态为「有效」的技术文档/知识条目作为依据——frontmatter `状态` 为 `已废弃` 或 `已被替代` 的条目**不得引用**（`已被替代` 的顺其正文链接取新文档）；无状态字段的存量文档视为有效，发现内容可疑时向用户确认。

**技术栈选择性引用**：tech / team 知识库按「通用领域 + 技术栈领域」划分（`frameworks/<栈>/`）——引用时只取与当前项目技术栈匹配的领域 + 通用领域，其他技术栈领域的文档不作为本项目依据。

**团队库匹配人工确认（硬规则）**：项目级 `CLAUDE.md` 的团队规范段存在「⛔ 栈领域待人工确认」标记时（`init project` 生成即带），AI 必须先发起确认再使用团队栈领域——列出 `biz-tech-docs/frameworks/` 实际存在的目录，按项目技术栈给出建议匹配项，**以 AI 提问、人工回答的方式**确定引用哪个领域（或确认无匹配），确认后把该标记段改写为具体领域路径并附确认人与日期；**人工未确认前只引用通用领域**，不混入其他技术栈、不臆造领域名；上层无匹配领域时显式提示缺口（`/agile:knowledge capture` 沉淀或 tech-specs 提案/条款修正）。

## 7. 命令速查

| 命令 | 用途 |
|---|---|
| /agile:help | 全部命令总览 |
| /agile:prd | 生成 PRD/AC/功能树/菜单树 |
| /agile:sync-req | 需求产物 → process-docs |
| /agile:architect | 技术方案设计 |
| /agile:gen-test | Stage 1 测试案例 |
| /agile:backend | 后端 TDD 开发 |
| /agile:frontend | 前端分层开发 |
| /agile:ui | UI/组件库生命周期 |
| /agile:run-test | Stage 2 测试执行与验收 |
| /agile:review | 验收汇总与门禁判定 |
| /agile:release | 发布前置检查与记录 |
| /agile:init | AI 陪同初始化项目（问答定制 + 团队库确认 + 辅助能力配置） |
| /agile:add-template | AI 辅助建设新模板（骨架 + 登记 + 校验 + 冒烟） |
| /agile:share-template | 把 workspace 项目打包为模板（清理审计 + 占位符还原 + 登记 + 校验 + 冒烟） |
| /agile:add-task | 补充遗漏任务 |
| /agile:fix-bug | 根因诊断修复 |
| /agile:feedback | 问题反馈报告 |
| /agile:knowledge | 知识库建设与沉淀 |

## 附录 A：任务目录模板（初始 8 个 .md，创建规范）

创建 `process-docs/<编号>/` 时按以下模板逐一生成（`{{id}}` 替换为编号；**幂等**：目录与文件已存在则跳过，绝不覆盖既有内容）。轻量通道（STO 轻量 / BUG / OPS）只额外初始化 `requirement.md` 与 `gen-test.md` 的差异内容（见各命令），其余文件仍按模板创建骨架。requirement.md / design.md / gen-test.md 模板自带「修订记录」节（人工修改登记，执行口径见 §3「人工修订感知流程」）。

**requirement.md**：

```markdown
# {{id}} 需求说明

> 由 /agile:sync-req（完整）或 /agile:fix-bug（BUG）创建，agile:prd / agile:sync-req 会填充此文档。

## 背景

（需求来源、业务背景）

## 目标

（本需求要达成的目标）

## 验收标准（AC）

- [ ] AC1: ...
- [ ] AC2: ...

## 修订记录

> 人工修改本文件后补记一行；未补记的由 AI 人工修订感知流程登记。

| 日期 | 来源 | 摘要 | 采纳结论 |
|---|---|---|---|
```

**design.md**：

```markdown
# {{id}} 技术设计

> 由 agile:architect 填充（SDD：先设计后开发）。参考抽屉一/二规范。

## 方案概述

## 涉及模块

| 模块 | 仓库 | 改动类型 |
|---|---|---|
| | | |

## 接口设计

### 跨接口统一约定

> 本表 = 全部接口的公共契约，逐接口契约块不再重复这些内容，偏离必须列入「本次例外」。已有规范引用出处（优先级：抽屉一 > 抽屉二），不重复发明；未覆盖项由本次设计声明。每行必填——真不适用填「不适用」，不许留空。

| 约定项 | 本次采用 | 来源 |
|---|---|---|
| 响应包装 | | |
| 鉴权 | | |
| 分页 | | |
| 时间格式 | | |
| 幂等 | | |

### 本次例外

（偏离公共契约的接口与原因；无则写「无」）

### 接口清单

| 编号 | 方法 | 路径 | 用途 | AC 映射 |
|---|---|---|---|---|

### API-<编号> <接口名>（逐接口契约块，每接口一份）

- 路径参数 / 查询参数：字段、类型、必填、说明、示例
- 请求体：字段、类型、必填、说明、示例
- 响应体：字段、类型、说明、示例（分页结构按统一约定表）
- 错误码：错误码、语义、前端处理建议
- 示例：成功 / 失败 JSON 各一

（契约块只写本接口特有内容；字段粒度到名 / 类型 / 可空——前后端并行开发仅依赖本节）

## 状态机 / 数据模型

## 风险与取舍

## 修订记录

> 人工修改本文件后补记一行；未补记的由 AI 人工修订感知流程登记。

| 日期 | 来源 | 摘要 | 采纳结论 |
|---|---|---|---|
```

**implementation.md**：

```markdown
# {{id}} 实施记录（任务分配）

> 主文件：任务分配表由 /agile:architect 在 design 冻结时填写（填表即冻结），之后**只读**；执行状态在各角色文件的任务清单中体现。
> 分工红线：后端只写 [implementation-be.md](implementation-be.md)，前端只写 [implementation-fe.md](implementation-fe.md)。

## 任务分配

| # | 任务 | 归属 | 明细 |
|---|---|---|---|
| 1 | | be / fe | [BE-1](implementation-be.md) / [FE-1](implementation-fe.md) |

## 联调约定

（接口对齐方式、环境、时间；双方知会。接口契约以 design.md 为唯一依据——变更由负责人修订 design.md 并知会对端，双方重读对齐后再继续，不沿用旧契约）
```

**implementation-be.md**：

```markdown
# {{id}} 后端实施记录

> 本文件由**后端专属维护**（agile:backend / TDD：Red → Green → Refactor）；前端记录见 [implementation-fe.md](implementation-fe.md)。

## 任务清单

- [ ] BE-1:

## TDD 循环记录

| # | 测试（先写） | 实现后状态 |
|---|---|---|
| 1 | | |

## 变更清单
```

**implementation-fe.md**：

```markdown
# {{id}} 前端实施记录

> 本文件由**前端专属维护**（agile:frontend / 分层开发：接口层 → 组件层 → 页面层）；后端记录见 [implementation-be.md](implementation-be.md)。

## 任务清单

- [ ] FE-1:

## 测试记录

| # | 测试（先写） | 实现后状态 |
|---|---|---|
| 1 | | |

## 变更清单
```

**gen-test.md**：

```markdown
# {{id}} 测试案例

> 由 /agile:gen-test（test-engineer）按 requirement.md 的 AC 与 design.md 填充（Stage 1，先于实现）。开发期各自只在本方节内补充/勾选。

## 后端用例

| 编号 | 类型 | 描述 | 自动化映射 | 状态 |
|---|---|---|---|---|
| | 单测 | | | |

## 前端用例

| 编号 | 类型 | 描述 | 自动化映射 | 状态 |
|---|---|---|---|---|
| | 单测 / e2e | | | |

（e2e 用例归入「前端用例」节：类型标 `e2e`，自动化映射指向前端项目的 e2e 脚本（如 `e2e/*.spec.ts`），只覆盖关键路径。）

> **轻量通道豁免**：本变更走轻量通道（STO 轻量 / BUG / OPS）时不填用例表，本文件只保留一行 `> 本变更走轻量通道，此文档不适用`。

## 修订记录

> 人工修改本文件后补记一行；未补记的由 AI 人工修订感知流程登记。

| 日期 | 来源 | 摘要 | 采纳结论 |
|---|---|---|---|
```

**review.md**：

```markdown
# {{id}} 评审记录

> 由 /agile:review 汇总填写：只记录与格式化验收结论、判定门禁，不代替人工验收。

## 验收矩阵

| 验收项 | 验收人 | 环境 | 结论 |
|---|---|---|---|
| | | | |

## 未闭环清单

## 门禁结论

（✅ 可交付 PR / ⛔ 不可交付）
```

**release.md**：

```markdown
# {{id}} 发布记录

## 发布内容

## 涉及仓库与 commit

| 仓库 | 分支 | commit |
|---|---|---|
| | | |

## 回滚方案
```
