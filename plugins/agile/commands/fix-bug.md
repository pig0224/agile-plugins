---
description: 快速修复 bug。自主诊断根因、设计最小修复并验证（含回归测试），开发/测试阶段任意问题均可使用。主会话按 bug-hunter 角色规范直接执行（分工例外，不委派 subagent）
argument-hint: <问题描述或需求编号+问题描述，如 STO-001 下单接口 500>
---

# /agile:fix-bug — 快速修复

团队 SOP **轻量通道的 BUG 形态**标准入口：缺陷修复（回归正确行为，无需产品审批）走此通道——worktree + PR 纪律不变，目录与文档按轻量填法（见 SOP「轻量通道」节；review.md 由 `/agile:review` 轻量形态生成一行验收确认，不填豁免声明）。

先阅读 skill `sdd-tdd-method` 与 `roles/bug-hunter.md`（诊断角色规范），然后按以下步骤执行。**分工例外声明**：诊断修复（跑复现 → 看结果 → 收窄的执行循环）由主会话直接执行、不委派 subagent——诊断过程全程可见，用户可随时补充线索（SKILL 分工红线中「个别命令文件显式声明的例外」）。

## 输入解析

- `$ARGUMENTS` 中若含需求编号（STO-xxx/BUG-xxx）则登记到该任务目录；不含则视为新缺陷——列出 `process-docs/` 现有编号顺延得到 `BUG-xxx`，按 skill `sdd-tdd-method` **附录 A 模板**直接创建 `process-docs/BUG-xxx/` 任务目录（初始 8 个 .md，幂等），并就地轻量初始化：`requirement.md` 头部标记 `> 本变更走轻量通道（BUG）` + 正文落缺陷描述与复现步骤；`gen-test.md` 填一行 `> 本变更走轻量通道，此文档不适用`。
- 其余文字为 bug 描述；为空则询问用户。

## 执行步骤

1. **环境检查**：`git status`（dirty 时停下询问用户，修复应基于干净基线）；外部资源未就位时先 `agile sync`。修复一律在需求分支的 worktree 内进行，无对应 worktree 时先 `agile worktree create feat/<编号>`（轻量通道 worktree 纪律不变）。
   登记到既有任务目录时，同时按 skill「人工修订感知流程」对该目录文档做人工修订感知（采纳登记 / 存疑返回人工确认）。
2. 按 `roles/bug-hunter.md` 角色规范在主会话直接执行「复现 → 定位 → 根因 → 最小修复 → 回归验证 → 登记」闭环（涉及仓库从 bug 描述推断或让用户指定），自检三件事：
   - 修复 diff 最小（只针对根因；顺手修表象需单独说明并获确认）
   - 复现测试确实从红转绿（TDD 式修 bug）
   - 该仓库全量测试无回归
3. 若根因涉及设计偏差，同步更新 design.md 并在汇报中标注。
4. **文档收尾（轻量通道）**：BUG-xxx 目录下 design.md 记录根因分析；run-test.md 记一行回归结论（全量测试通过）；review.md 不填——交付前由 `/agile:review` 轻量形态生成一行验收确认。

## 输出

汇报：根因（一句话）、证据链、修复概要、验证结果、登记位置；建议受影响面较大时运行 `/agile:run-test <编号>`。
