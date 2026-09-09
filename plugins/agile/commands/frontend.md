---
description: 前端开发与调试编排。主会话按 frontend-dev 角色规范（roles/frontend-dev.md）完成分层开发（接口层/组件层/页面层）与浏览器测试。分工红线显式例外：执行循环类命令，主会话直接执行、不委派 subagent
argument-hint: <需求编号> [前端项目名]
---

# /agile:frontend — 前端分层开发编排

先阅读 skill `sdd-tdd-method` 与 `roles/frontend-dev.md`（前端角色规范），然后按以下步骤执行。**分工例外声明**：执行循环类任务（跑测试 → 看结果 → 迭代的代码实施）由主会话直接执行、不委派 subagent——执行过程全程可见、用户可随时插话纠偏（SKILL 分工红线中「个别命令文件显式声明的例外」）。

## 前置校验

1. 解析 `$ARGUMENTS`：第一段为需求编号，第二段（可选）为前端项目名（缺省时扫描 `projects/` 一层（`projects/*`，单例与组合成员项目全部平铺）中含 vue/react 特征的项目——按 package.json 依赖特征 Glob/读取识别）。
2. 校验 `process-docs/<编号>/design.md` 已填充（SDD 红线）。未填充则停止。
   已填充时按 skill「人工修订感知流程」对照既有任务清单与已实现代码，design.md 有未登记的人工修订 → 先处置（采纳登记 / 存疑返回人工确认），防按过时设计继续开发。
3. 读 `process-docs/<编号>/menu-tree.md`（或 feature-tree）确认页面范围；**文件不存在时（轻量通道 / BUG 无 PRD 产物）跳过本步**——页面范围以 design.md「涉及模块」与用户输入为准，向用户确认后再动手。
4. `git status` 确认工作区干净；dirty 则询问。
5. 读 `implementation.md` 任务分配表：仍是骨架占位（任务列为空）→ 停下向用户确认——先补跑 `/agile:architect <编号>`（幂等补填，已填不重写），或确认按 design.md「涉及模块」继续开发（表由负责人后补）。已填则对照核对任务归属。

## 开发环境准备

`agile worktree create feat/<编号>` 创建/进入开发环境（自动同步外部仓库）：负责人已推送远程分支时自动跟踪检出（多人各自拉取同一需求分支协作），否则新建分支。后续在 worktree 中工作。

## 执行步骤

1. 从 `process-docs/<编号>/implementation-fe.md` 读取/初始化前端任务清单（按「接口层→组件层→页面层」分层拆分；对照 implementation.md 任务分配表核对任务归属）。**只写 implementation-fe.md，禁止改 implementation-be.md 与主文件。**
2. 按 `roles/frontend-dev.md` 角色规范在主会话直接执行分层开发（接口层 → 组件层 → 页面层，每层 TDD），节奏与纪律：
   - **批次节奏**：每批 2~3 个任务（长任务单批单做）；批间向用户一行摘要（本批完成什么、测试状态）再继续
   - **增量落盘**：逐任务完成即更新 implementation-fe.md（勾选 + 测试记录 + 建议 commit message），再进下一任务——**禁止长时间只读不写**：任务清单即进度看板，会话上下文压缩后从盘上状态续作，不重做已完成任务
   - 执行中用户可随时插话调整方向；契约问题按角色规范「接口变更闭环」处理（停止 → 负责人修订 design.md → 重读对齐再继续）
3. 浏览器验证：按角色规范经 Playwright 脚本 / e2e 驱动（以脚本输出为准，不虚构浏览器结论），dev server 启动、关键路径对照 AC 走查；临时验证脚本归 `process-docs/<编号>/scripts/`。
4. 每批完成后在该 worktree 运行前端测试命令，确认绿色。

## 闭环条件

三层全部完成、组件测试与页面走查通过、implementation-fe.md 更新完整。

## 输出

汇报：分层完成度（带任务标题）、进度（implementation-fe.md x/y）、组件/页面清单、测试与浏览器验证结论、建议的提交清单（待人工 add 后汇总提交，diff 由人工在 add 时审阅）、遗留问题；建议下一步 `/agile:run-test <编号>`。
