---
name: sdd-tdd-method
description: agile 工作区的 SDD/TDD 研发方法论与文档规范。凡执行 agile:prd / agile:architect / agile:backend / agile:frontend / agile:gen-test / agile:run-test 等 agile 系列命令时必须先阅读本 skill。涵盖：一个根五个抽屉的目录约定、需求编号任务目录、SDD 先设计后开发、TDD Red-Green-Refactor 循环、过程产物五文档的填写规范。
---

# agile SDD/TDD 方法论

## 1. 工作区结构（一个根、五个抽屉）

工作区根 = 存在 `.agile/workspace.yaml` 的目录。**一切路径先读 `.agile/workspace.yaml` 的 `paths` 段获得**，默认约定：

| 抽屉 | 默认路径 | 内容 | 角色 |
|---|---|---|---|
| 一 | `tech-specs/` | 公司级技术规范（技术栈、SQL、安全硬规范） | 全员遵守 |
| 二 | `biz-tech-docs/` | 团队技术设计知识库（架构、状态机、技术方案、工程规范） | 架构师 |
| 三 | `biz-product-docs/` | 产品设计知识库（PRD、产品规范、UI 规范、交互规范） | 产品经理 |
| 四 | `projects/` | 项目代码（workspace 单仓内普通目录） | 开发 |
| 五 | `process-docs/` | 过程产物（按需求编号归档，workspace 根仓库内） | 全员 |

CLI 与 MCP：工作区操作（sync/status/doctor 等）通过 Bash 执行 `agile <command>`，或调用捆绑 MCP 工具 `mcp__plugins_agile_agile__*`（task 目录创建只有 MCP 工具 `agile_task_create`，无 CLI 命令）。**不要手工造 git submodule 命令，交给 agile CLI。**

## 2. 需求编号任务目录与通道判定（STO / BUG / OPS）

`process-docs/<编号>/` 标准任务目录（STO-xxx 业务需求 / BUG-xxx 缺陷修复 / OPS-xxx 技术变更），由 MCP 工具 `agile_task_create` 生成（task 能力不暴露为 CLI 命令，插件命令统一经 MCP 调用）。五文档 + 两份角色卫星文件，共 7 个 .md：

- `requirement.md` — 需求说明与验收标准（AC）。产品/需求侧填充。
- `design.md` — 技术设计。**SDD 核心：开发前必须先完成**。参考抽屉一/二规范。
- `implementation.md` — 实施记录主文件：**任务分配表**（design 冻结时填写，之后只读）+ 联调约定。
- `implementation-be.md` — **后端专属**实施记录：任务清单、TDD 循环记录、变更清单。前端禁写。
- `implementation-fe.md` — **前端专属**实施记录：任务清单、测试记录、变更清单。后端禁写。
- `review.md` — 评审记录。
- `release.md` — 发布记录与回滚方案。

> 文件级隔离：前后端并行开发（同一需求分支）时各写各的角色文件，git 合并零冲突。测试案例文档 gen-test.md 同理分「后端用例」「前端用例」两节。

当前需求编号贯穿始终：所有命令产出都写入对应 `process-docs/<编号>/`。

### 通道判定（所有命令先判再动）

**两维正交**：性质（编号前缀）定拍板人，深度（完整 / 轻量）定填写量。

| 形态 | 判定 | 典型场景 | 目录创建 |
|---|---|---|---|
| 完整 | 抽屉三 `requirements/<编号>/` 有 PRD 产物 | 需产品定稿 PRD/AC 的需求 | `/agile:sync-req <编号>` |
| STO 轻量 | 产品一句话确认分配 STO 编号，无 PRD | mini feat、文案/样式调整 | `/agile:sync-req <编号> <一句话需求>` |
| BUG | 缺陷，无需拍板（回归正确） | 行为与预期不符 | `/agile:fix-bug`（无编号顺延 BUG-xxx） |
| OPS | 技术变更，运维拍板 | 重构、依赖升级、CI 微调 | `/agile:sync-req <编号> <改动说明>` |

**轻量机读标记**：`requirement.md` 头部含 `> 本变更走轻量通道` 即轻量形态，各命令按此自适应——architect 输出三五行方案简述（design.md）；review 一行验收确认（报告人确认修复生效（BUG）/ 提需求人确认（STO 轻量）/ 负责人自查（OPS））；release 涉及部署才记一行。**不变**：TDD 红线不豁免（bug 修复必须复现测试 Red→Green）；worktree、main 禁直推、PR、CI 门禁照走。详细规范见团队 SOP「轻量通道」节。

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
4. 当前任务 `design.md`（本次的具体决策）

**有效性过滤（硬规则）**：仅使用状态为「有效」的技术文档/知识条目作为依据——frontmatter `状态` 为 `已废弃` 或 `已被替代` 的条目**不得引用**（`已被替代` 的顺其正文链接取新文档）；无状态字段的存量文档视为有效，发现内容可疑时向用户确认。

**技术栈选择性引用**：tech / team 知识库按「通用领域 + 技术栈领域」划分（`frameworks/<栈>/`）——引用时只取与当前项目技术栈匹配的领域 + 通用领域，其他技术栈领域的文档不作为本项目依据。

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
| /agile:add-task | 补充遗漏任务 |
| /agile:fix-bug | 根因诊断修复 |
| /agile:feedback | 问题反馈报告 |
| /agile:knowledge | 知识库建设与沉淀 |
