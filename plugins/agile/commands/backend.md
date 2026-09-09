---
description: 后端开发与测试编排。主会话按 backend-dev 角色规范（roles/backend-dev.md）完成 TDD 开发（Red-Green-Refactor）与接口测试。分工红线显式例外：执行循环类命令，主会话直接执行、不委派 subagent
argument-hint: <需求编号> [项目名/模块名，如 STO-001 order-service]
---

# /agile:backend — 后端 TDD 开发编排

先阅读 skill `sdd-tdd-method` 与 `roles/backend-dev.md`（后端角色规范），然后按以下步骤执行。**分工例外声明**：执行循环类任务（跑测试 → 看结果 → 迭代的代码实施）由主会话直接执行、不委派 subagent——执行过程全程可见、用户可随时插话纠偏（SKILL 分工红线中「个别命令文件显式声明的例外」）。

## 前置校验

1. 解析 `$ARGUMENTS`：第一段为需求编号，第二段（可选）为项目名/模块名（`projects/` 下的目录）。
2. 校验 `process-docs/<编号>/design.md` 已填充（SDD 红线：**无设计不开发**）。未填充则停止，提示先 `/agile:architect <编号>`。
   已填充时按 skill「人工修订感知流程」对照既有任务清单与已实现代码——design.md 有未登记的人工修订 → 先处置（采纳并登记 / 存疑返回人工确认），防按过时设计继续开发。
3. 校验测试案例文档存在（`gen-test.md` 或 AC 内嵌案例）。缺失时警告但允许继续（TDD 红线仍在：先写失败测试）。
4. 检查涉及项目目录存在且工作区干净（`git status`）；dirty 则停下询问用户。
5. 读 `implementation.md` 任务分配表：仍是骨架占位（任务列为空）→ 停下向用户确认——先补跑 `/agile:architect <编号>`（幂等补填，已填不重写），或确认按 design.md「涉及模块」继续开发（表由负责人后补）。已填则对照核对任务归属。

## 开发环境准备

- 用 `agile worktree create feat/<编号>` 创建/进入开发环境（workspace 级 worktree，自动同步外部仓库）：负责人已推送远程分支时自动跟踪检出（多人各自拉取同一需求分支协作），否则新建分支。后续工作在 worktree 路径进行。
- 未指定项目时：从 design.md「涉及模块」表中选出后端相关项目，逐个处理。

## 执行步骤

1. 从 `process-docs/<编号>/implementation-be.md` 读取任务清单（无则从 design.md 的接口/模块清单初始化；对照 implementation.md 任务分配表核对任务归属）。**只写 implementation-be.md，禁止改 implementation-fe.md 与主文件。**
2. 按 `roles/backend-dev.md` 角色规范在主会话直接执行 TDD 循环（Red → Green → Refactor），节奏与纪律：
   - **批次节奏**：每批 2~3 个任务（长任务单批单做）；批间向用户一行摘要（本批完成什么、测试状态）再继续
   - **增量落盘**：逐任务「写失败测试并记录红 → 最小实现转绿 → 必要重构 → 立即更新 implementation-be.md（勾选 + 循环记录 + 建议 commit message）」，再进下一任务——**禁止长时间只读不写**：任务清单即进度看板，会话上下文压缩后从盘上状态续作，不重做已完成任务
   - 执行中用户可随时插话调整方向；契约问题按角色规范「接口变更闭环」处理（停止 → 负责人修订 design.md → 重读对齐再继续）
3. 每批完成后复核：在 worktree 目录运行该仓库标准测试命令确认绿色；发现偏差当批修复。

## 闭环条件

- 任务清单全部勾选、测试全绿、implementation-be.md 的 TDD 循环记录完整、建议的提交序列符合 `STO-xxx(red|green|refactor):` 规范（git add 归人工；人工 add 后 AI 可汇总 commit，commit 前检查无遗漏未暂存文件）。

## 输出

汇报：完成任务（带任务标题，如「BE-5 导出任务超时状态机（完成）」）、进度（implementation-be.md x/y）、测试结果、建议的提交清单（待人工 add 后汇总提交，diff 由人工在 add 时审阅）、遗留问题；建议下一步 `/agile:frontend`（如涉及）或 `/agile:run-test <编号>`。
