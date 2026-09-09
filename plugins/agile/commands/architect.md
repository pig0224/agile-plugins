---
description: 技术方案设计。调度 tech-architect subagent 基于 PRD/AC 与团队知识库产出 design.md（SDD：先设计后开发）；design 冻结时一并填写 implementation.md 任务分配表与联调约定
argument-hint: <需求编号，如 STO-001>
---

# /agile:architect — 技术方案设计

先阅读 skill `sdd-tdd-method`，然后按以下步骤执行。**分工例外声明**：设计产出委派 tech-architect subagent；design 冻结后的填表（步骤 4）为主会话直接执行的文件操作，无独立实施工作（SKILL 分工红线中「个别命令文件显式声明的例外」）。

## 前置校验

1. 读 `.agile/settings.json` 获取 `paths.processDocs`。
2. 需求编号：`$ARGUMENTS`（为空时列出 `process-docs/` 下现有编号目录让用户选择）。
3. 检查 `process-docs/<编号>/requirement.md` 已填充（完整形态含 AC，或轻量形态含一句话需求）。**未填充则停止**，提示先执行 `/agile:sync-req`（完整流程需先 `/agile:prd`；轻量通道直接提供一句话需求）。
   已填充时按 skill「人工修订感知流程」重读 requirement.md；design.md 已存在且 requirement 相对既有设计有人工修订 → 先处置（采纳并在修订记录登记 / 存疑返回人工确认），防止按过时需求续写设计。

## 执行步骤

1. 调用 **tech-architect** subagent（Task 工具委派），传入：
   - 需求编号、`process-docs/<编号>/requirement.md` 路径
   - 抽屉一路径（`paths` 中 `techSpecs`）、抽屉二路径（`bizTechDocs`）、抽屉四路径（`projects`）
   - **通道深度**：requirement.md 头部含 `本变更走轻量通道` 标记时为轻量——design.md 按 SOP 轻量填法输出**三五行方案简述**（STO 轻量 / OPS），不做完整设计
2. subagent 产出 `process-docs/<编号>/design.md`。
3. 审查产出（自己读一遍 design.md）：
   - 涉及项目是否都在 `projects/` 目录中存在（不在则列出并建议 `agile init project --template <模板>`）
   - 是否有 TBD 项
4. **design 冻结：填写 implementation.md 主文件**（主会话直接执行）：TBD 清零或用户确认接受后，把任务分配表填入 `implementation.md`——任务、归属、明细从 design.md「涉及模块」表导出（归属端按项目类型判定，无法判定时向用户确认），替换骨架占位行；「联调约定」按模板括注填接口对齐方式 / 环境 / 时间（契约本身不复制，以 design.md 为唯一依据）。轻量形态按 design 简述填实际任务（一两行）。**填表即冻结**：之后主文件只读，仅 `/agile:add-task` 可追加行。任务分配表已填（非骨架占位，如人工已填或重跑）时不重写——按 skill「人工修订感知流程」处置差异。TBD 存在且未获确认则跳过本步，汇报中说明「任务分配表未填写（design 未冻结）」。
5. 汇报：设计要点摘要、涉及仓库、接口数、TBD 列表、任务分配表填写结果（未填则说明原因）。

## 完成后建议

提示下一步：`/agile:gen-test <编号>`（测试案例先行）。
